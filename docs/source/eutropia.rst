Eutropia
========

*From ancient Greek, ευτροπία: docile, of good manners, versatile, variable.*

The Point Spread Function (**PSF**) is a function that describes the pattern of light produced by a point-like source. In the context of NEXT, the **PSF** describes the average SiPM responses produced when a single electron crosses the EL gap. In practice, this function is replaced by a discretize mapping of the light response as a function of the relative transverse coordinates (:math:`x_r`, :math:`y_r`).

Since it is not practical to measure the **PSF** with single electrons, :math:`^{83m}Kr` events are used. These produce highly localized energy depositions (the closest we can achieve to a true point-like source) with fixed energy.

This city generates **PSF**\ s from **Hits** produced with a specific configuration of :doc:`penthesilea` [#]_.
The **PSF** is obtained by mapping the SiPM light collection relative to the barycenter (center of gravity of SiPM response) for pointlike events collapsed along the drift coordinate. The distribution obtained is averaged over a large number of events to reduce uncertainties. The **PSF** can be obtained for different sections of the active volume independently.

The **PSF** is later used by :doc:`beersheba` to reverse the effects of electron diffusion and light emission and obtain the original charge deposition.

.. _Eutropia input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/RECO/Events``

.. _Eutropia output:

Output
------

 * ``/PSF/PSFs``: Table containing all the **PSF**\ s generated. Columns: xr, yr, zr, x, y, z, factor, nevt.

The columns xr, yr, and zr indicate the central value of the **PSF** bin.
The columns x, y, and z indicate the central value of the overall sector in which the **PSF** is generated.
The columns factor and nevt are the **PSF** value and the number of events used to generate that number.

.. image:: images/eutropia/output_example.png
  :width: 500

.. _Eutropia config:

Config
------

Besides the :ref:`Common arguments to every city`, *Eutropia* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``xrange | yrange``
     - ``Sequence[float]``
     - Ranges of the relative coordinates used to measure the **PSF**.

   * - ``zbins``
     - ``Sequence[float]``
     - Binning in the longitudinal coordinate. A set of **PSF**\ s will be produced for each bin.

   * - ``xsectors | ysectors``
     - ``Sequence[float]``
     - Binning in x | y. A **PSF** will be generated for each (x, y) bin.

   * - ``bin_size_xy``
     - ``float``
     - Bin size for the **PSF**.

.. _Eutropia workflow:

Workflow
--------

Eutropia performs a number of data transformations in order to obtain the **PSF**\ s. These operations can be grouped in three main tasks, performed in the following order:

 * :ref:`Prepare data slices <Prepare data slices>`
 * :ref:`Compute the PSFs <Compute the PSFs>`
 * :ref:`Combine PSFs <Combine PSFs>`


.. _Prepare data slices:

Prepare data slices
:::::::::::::::::::

First, the events are divided into z-slices according to the parameters `zbins`, `xsectors` and `ysectors`. Each of this sectors of `x`, `y` and `z` will yield a separate **PSF** [#]_. These sectors can be identified in the output data by their central values (columns `x`, `y` and `z` of the output table). The procedure that follows is then applied to each of this datasets independently.

The hits comming from previous stages of the reconstruction chain do not contain entries with null charge [#]_. However, SiPMs with null charge should also be considered as part of the light response map. Thus, in this step, the missing hits are added to the dataset. Next, the charge distribution is normalized to 1 for event independently. Finally, the relative coordinates (:math:`x_r` and :math:`y_r`) are computed by subtracting the barycenter from each SiPM position.


.. _Compute the PSFs:

Compute the **PSF**\ s
::::::::::::::::::::::

The charge distribution for all events is then histogrammed in the coordinates :math:`x_r` and :math:`y_r`. The binning of this histograms is determined by the parameters `xrange`, `yrange`, and `bin_size_xy`. The **PSF** factor in each bin is defined as the value of the bin normalized to the number of events in the bin, i.e. the average charge observed for a specific range of values of :math:`x_r` and :math:`y_r`. An example of such histogram is shown below.

.. image:: images/eutropia/psf_2d.png
  :width: 850

A 1d slice of this histogram (for y=0) is represented below for different z-slices, demonstrating why it is necessary to generate separate **PSF**\ s for various ranges of z.

.. image:: images/eutropia/psf_1d.png
  :width: 850

.. _Combine PSFs:

Combine **PSF**\ s
::::::::::::::::::

In order to produce an accurate **PSF**, a large number of events is needed. At the same time, it is neither possible nor efficient to process large number of events at once. The approach is thus to produce **PSF**\ s with fewer events and merge them afterwards. This option is available both within the city and externally as a separate tool. Because the city accepts many input files, it will run the **PSF** generation for each file independently and merge them later. The external tool follows the exact same methodology [#]_.

A **PSF** value is by construction an average of normalized charges. Therefore, an arbitrary number of **PSF** entries with values :math:`p_k` produced with :math:`n_k` events can be combined into a single entry with value :math:`\frac{\Sigma p_k \cdot n_k}{\Sigma n_k}` and :math:`\Sigma n_k` events. Each bin of the **PSF** is combined with the corresponding bin of all **PSF**\ s available.

 .. [#] The `rebin` parameter in :doc:`penthesilea` must be set to a large number (e.g. 10000) in order to obtain **Hits** for events integrated over the longitudinal axis.
 .. [#] While it is common to generate separate **PSF**\ s for different ranges of z, so far we haven't assessed the performance of using separate **PSF**\ s for different ranges of (x, y).
 .. [#] Technically, they do not contain entries with charge below a certain configurable threshold. This value should be reasonably low to describe the tails of the **PSF** distribution accurately.

 .. [#] This allows to process each file in a separate core of a computer cluster and merge the files later. This is much faster than running them sequentally.
