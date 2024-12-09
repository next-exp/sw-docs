:orphan:

============================
Satellite killer explanation
============================

The `functions in question <https://github.com/next-exp/IC/blob/master/invisible_cities/reco/deconv_functions.py#L26>`_ 
(introduced in `this PR <https://github.com/next-exp/IC/pull/890>`_) generates
a satellite mask that is used to remove satellite energy deposits produced throughout the deconvolution. 

Lets step through the logic!

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


.. highlight:: python 
.. code-block:: python

    footprint = ndi.generate_binary_structure(im_mask.ndim, 2)

This creates an n-dimensional array of ``True`` values that are
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

Which uses the footprint and the above mask to label the different 'deposits' as shown below:


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


This mask is then returned, and applied such that all true elements in the original array are zero.


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

