:orphan:

============================
Satellite killer explanation
============================



The `functions in question <https://github.com/next-exp/IC/blob/master/invisible_cities/reco/deconv_functions.py#L26>`_:


.. highlight:: python 
.. code-block:: python

    def collect_component_sizes(im_mask : np.ndarray[bool]) -> (np.ndarray[int], np.ndarray[int]):
        '''
        A function that returns the sizes of different clusters of 1s and 0s within the data
        for removal of satellites.

        This function uses the scipy ndimage library to identify different 'clusters' of 1s within the slice, and 
        checks if these clusters are below the size considered for satellite deposits (`satellite_max_size`).

        Parameters
        ----------
        im_mask : 2D boolean array describing the regions of energy in deconvolution slice

        Returns
        -------
        labels          : 2D array equivalent to im_mask with each region labelled 0, 1, 2, etc
        component_sizes : Array of the length of each component size
        '''
        # label deposits within the array
        # hardcoded to include diagonals in the grouping stage (2)
        # count the bins of each labelled deposit
        footprint       = ndi.generate_binary_structure(im_mask.ndim, 2)
        labels, _       = ndi.label(im_mask, footprint)
        component_sizes = np.bincount(labels.ravel())
        return labels, component_sizes


    def generate_satellite_mask(im_deconv          : np.ndarray[float], 
                                satellite_max_size : int, 
                                e_cut              : float, 
                                cut_type           : Optional[CutType]=CutType.abs) -> np.ndarray:
        '''
        An adaptation to the scikit-image (v0.24.0) function [1], identifies 
        satellite energy depositions within deconvolution image by size
        and proximity to other depositions.

        The function takes a deconvolution z-slice, applies a mask based on the `e_cut` and `cut_type` to only
        allow 0s and 1s for values passing the energy cut. 
        It then uses the scipy ndimage library to identify different 'clusters' of 1s within the slice, and 
        checks if these clusters are below the size considered for satellite deposits (`satellite_max_size`).
        These satellite deposits are highlighted in a new mask, that is returned and used to remove them.

        Returns the mask required to remove satellites as done in `richardson_lucy()`
        
        Parameters
        ----------
        im_deconv          : Deconvoluted 2D array
        satellite_max_size : Maximum size of satellite deposit, above which they are considered 'real'.
        e_cut              : Cut over the deconvolution output, arbitrary units or percentage
        cut_type           : Cut mode to the deconvolution output (`abs` or `rel`) using e_cut
                            `abs`: cut on the absolute value of the hits.
                            `rel`: cut on the relative value (to the max) of the hits.

        Returns
        -------
        array : Boolean mask of all labelled satellite deposits, True where satellites should be removed.

        References
        ----------
        .. [1] https://github.com/scikit-image/scikit-image/blob/main/skimage/morphology/misc.py#L59-L151
        '''
        if cut_type is CutType.rel:
            im_deconv = im_deconv / im_deconv.max()

        # separate different regions below and above e_cut
        # then label regions (components) appropriately and determine their sizes.
        labels, component_sizes = collect_component_sizes(im_deconv >= e_cut)

        # check if no satellites within deposit return False array
        # (mask that removes no satellites).
        if len(component_sizes) <= 2:
            return np.full(im_deconv.shape, False)

        # Find regions smaller than `satellite_max_size` and mask them, 
        # ignoring the first region (background). Read gist for full explanation.
        too_small      = component_sizes < satellite_max_size
        too_small[0]   = False 
        too_small_mask = too_small[labels]
        return too_small_mask

This mask is then used to remove satellites from 2D deconvolution arrays. Lets step through the logic!

**Stepping through the satellite killing process**

We start in ``generate_satellite_mask()``:


.. highlight:: python 
.. code-block:: python

    if cut_type is CutType.rel:
            im_deconv = im_deconv / im_deconv.max()

This is a simple check for the ``cut_type``, if it is absolute (``abs``) no changes are needed, the relative (``rel``) changes are self explanatory.


.. highlight:: python 
.. code-block:: python

    labels, component_sizes = collect_component_sizes(im_deconv >= e_cut)


As shown above, the function then uses ``collect_component_sizes()`` to collect an array of all the labelled 
regions, and the sizes of each of those regions. A boolean cut is applied in the argument (``im_deconv >= e_cut``) 
to ensure the array exists with only 0s and 1s, with 0s being energies below the `e_cut` and 1s being above.

So the array should look something like this (as an example):


.. highlight:: python 
.. code-block:: python

    >>> print(im_mask)
    array([[1., 1., 0., 0., 1.],
           [1., 1., 0., 0., 0.],
           [0., 0., 0., 0., 0.],
           [0., 0., 1., 1., 1.],
           [0., 0., 1., 1., 1.]])

The function ``collect_component_sizes()`` completes the following steps:

* ``footprint = ndi.generate_binary_structure(im_mask.ndim, 2)``
* This creates an n-dimensional array of ``True`` values that are
  used to map the connectivity of our 1s in the above array, like such:


.. highlight:: python 
.. code-block:: python

    >>> print(footprint)
    array([[ True,  True,  True],
           [ True,  True,  True],
           [ True,  True,  True]])

Since we're always working in the 2D case, we could hard code this, 
but its preferable to be generalised as such for futureproofing purposes (3D beersheba).
The next line is:


.. highlight:: python 
.. code-block:: python

    labels, _ = ndi.label(im_mask, footprint)

Which uses the footprint and the above mask to label the different 'deposits' as shown below


>>> print(labels)
array([[1, 1, 0, 0, 2],
       [1, 1, 0, 0, 0],
       [0, 0, 0, 0, 0],
       [0, 0, 3, 3, 3],
       [0, 0, 3, 3, 3]], dtype=int32)

Next:

.. highlight:: python 
.. code-block:: python

    component_sizes = np.bincount(labels.ravel())

This counts the occurence of each type within the array `labels`


>>> print(component_sizes)
array([14,  4,  1,  6])

14 zeros, 4 ones, 1 twos, 6 threes.

The ``labels`` and ``component_sizes`` are then returned, which is followed by an if statement:


.. highlight:: python 
.. code-block:: python

    if len(component_sizes) <= 2:
    	# Return a fully False array, so that no objects get removed
    	return np.full(im_deconv.shape, False)


If there are only 0s and 1s, there are no satellites! So you can pass back a completely False array.

.. highlight:: python 
.. code-block:: python

    too_small = component_sizes < satellite_max_size


This creates an equivalent array of trues and falses, so lets say ``satellite_max_size = 3``:

>>> print(too_small)
array([False, False, True, False])

This has flagged the 2nd element (corresponding to the 2s above) as a satellite.

We want the first element (0s) to always be false, so we set that:
``too_small[0] = False``

You can then map this true/false map back onto the array to create a mask in which 
only elements you want to remove from the initial relay are True.

>>> too_small_mask = too_small[label]
>>> print(too_small_mask)
array([[False, False, False, False, True],
       [False, False, False, False, False],
       [False, False, False, False, False],
       [False, False, False, False, False],
       [False, False, False, False, False]])


This mask is then returned, and applied such that all true elements in the original array are zero


>>> print(im_deconv)
array([[1., 1., 0., 0., 1.],
       [1., 1., 0., 0., 0.],
       [0., 0., 0., 0., 0.],
       [0., 0., 1., 1., 1.],
       [0., 0., 1., 1., 1.]])
>>> im_deconv[too_small_mask] = 0
>>> print(im_deconv)
array([[1., 1., 0., 0., 0.], #<--- satellite gone!
       [1., 1., 0., 0., 0.],
       [0., 0., 0., 0., 0.],
       [0., 0., 1., 1., 1.],
       [0., 0., 1., 1., 1.]])

