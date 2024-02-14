Sophronia
=========

*From ancient Greek: sensible, prudent.*


This city processes each S2 signal previously selected as pmaps in
irene assuming a unique S1 within an event to produce a set of
reconstructed energy depositions (**Hits**). Hits consist of three
dimensional coordinates with an associated energy (PMT signal) and
charge (SiPM signal). The city contains a peak/event filter, which can
be configured to find events with a certain number of S1/S2 signals
that satisfy certain properties. Currently, the city is designed to
accept only 1 S1 signal. If the configuration allows for more than one
S1, the first one will be used to estimate z. Furthermore, hits can be
optionally corrected for geometrical and lifetime effects. Besides
hits, the city also stores the global (x, y) position of each S2
signal.  The tasks performed are:
    - Classify peaks according to the filter.
    - Filter out events that do not satisfy the selector conditions.
    - Resample S2 signals.
    - Compute a set of hits for each slice in the resampled S2 signal.
    - If there are more than one hit per slice, share the energy
      according to the charge recorded in the tracking plane.
    - If correction maps are provided, apply corrections to hits.

.. _Sophronia input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/PMAPS/S1``
 * ``/PMAPS/S2``
 * ``/PMAPS/S2Si``

.. _Sophronia output:

Output
------

  * ``/DST/Events``: summary of the S1 and S2 information of the reconstructed events. See :doc:`dorothea` for more details.
  * ``/RECO/Events``: list of hit data for each event.
  * ``/Filters/s12_selector``: flag for whether an event passed the S1 and S2 selections
  * ``/Filters/valid_hit``: flag for whether an event passed the empty pmap filter

.. _Sophronia config:

Config
------

Besides the :ref:`Common arguments to every city`, *Sophronia* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``drift_v``
     - ``float``
     - Drift velocity. Used to calculate the z position from the drift time.

   * - ``s1_params``
     - ``dict``
     - Selection criteria for S1 signals (see `Peak filtering`_)

   * - ``s2_params``
     - ``dict``
     - Selection criteria for S2 signals (see `Peak filtering`_)

   * - ``global_reco_algo``
     - ``XYReco``
     - Reconstruction algorithm applied to entire peaks (kDST)

   * - ``global_reco_params``
     - ``dict``
     - Configuration parameters for the global recontruction algorithm

   * - ``rebin``
     - ``int``
     - Specifies the level of resampling or grouping of consecutive samples for reconstruction. Must be :math:`\geq` 1

   * - ``rebin_method``
     - ``RebinMethod``
     - Which resampling method to use (usually stride)

   * - ``q_thr``
     - ``float``
     - Charge threshold to be applied to the SiPMs

   * - ``sipm_charge_type``
     - ``SiPMCharge``
     - Type of SiPM charge to use (usually raw)

   * - ``same_peak``
     - ``bool``
     - Controls whether invalid hits can only be merged with other hits from the same peak

   * - ``corrections_file``
     - ``str``
     - Path to the file holding the correction maps

   * - ``apply_temp``
     - ``bool``
     - Whether to apply temporal corrections (in general, False in MC, True in data)

.. _Sophronia workflow:

Workflow
--------

Sophronia performs the following data transformations:

 * :ref:`Peak filtering <Peak filtering>`
 * :ref:`Build event summary <Build event summary>`
 * :ref:`Resampling and hit creation <Resampling and hit creation>`
 * :ref:`Hit merging <Hit merging>`
 * :ref:`Hit correction <Hit correction>`


.. _Peak filtering:

Peak filtering
::::::::::::::

:doc:`irene` finds peaks in the waveform with general characteristics,
but does not impose strong requirements on them. *Sophronia*, however,
can be more specific and select S1 and S2 signals based on the
following peak properties: width, height, and integral. These
quantities are calculated based on a (low) threshold applied to the
PMT waveform of the peak. For S2 signals, the number of SiPMs with
some signal is also taken into account [1]_. The variables that
control this filtering are provided in the parameters
``{s1,s2}_params`` and are defined as follows:

- ``{s1,s2}_w{min,max}``: minimum/maximum width of S1/S2 peaks
- ``{s1,s2}_h{min,max}``: minimum/maximum height of S1/S2 peaks in a 1-:math:`\mu`\ s sample
- ``{s1,s2}_e{min,max}``: minimum/maximum integral of S1/S2 peaks
- ``{s1,s2}_ethr``: threshold applied to the PMT-summed waveform to compute the quantities above
- ``{s1,s2}_n{min,max}``: minimum/maximum number of S1/S2 peaks that satisfy *all* of the criteria above for given event
- ``s2_nsipm{min,max}``: minimum/maximum number of SiPMs with signal in an S2 peak

.. [1] Bear in mind that :doc:`irene` requires a minimum amount of
       charge per peak and per slice for the SiPM to be considered.


.. _Build event summary:

Build event summary
:::::::::::::::::::

This part of the processing is equivalent to :doc:`dorothea` using
``global_reco_algo`` and ``global_reco_params``. See that city's
documentation for more details.

.. _Resampling and hit creation:

Resampling and hit creation
:::::::::::::::::::::::::::

In numerous ocasions, the energy deposition in a 1-:math:`\mu`\ s
sample is not enough to produce a significant signal in the SiPMs,
which results in poor reconstruction. It is therefore useful to be
able to resample the waveforms and increase the sampling period (lower
sampling rate). The variable that controls this resampling is
``rebin`` and it specifies how many consecutive 1-:math:`\mu`\ s
slices are added up together. A value of ``rebin = 1`` means no
resampling is performed. Only integer values greater than or equal to
1 are accepted.

Unlike *Penthesilea*, *Sophronia* considers the response of a SiPM in
a single (resampled) slice a *Hit*. Hits are defined as the
aggregation of the absolute position of the SiPM, the time difference
between the slice and the S1 peak time [2]_, the amplitude of the SiPM
waveform (charge) and the energy corresponding to said charge. The
corresponding energy is defined as

.. math::

   E_i = \frac{Q_i}{\sum_{k=0}^{N} Q_{k}} E_{slice}

where :math:`Q_i` and :math:`E_i` are the charge and corresponding
energy of SiPM :math:`i`, respectivly; :math:`N` is the number of
SiPMs with signal in the slice and :math:`E_{slice}` is the energy of
the slice, given by the amplitude of the PMT-summed waveform in the
slice.

.. [2] This information is used to obtain the z position of the hit by
       dividing the time difference by the drift velocity
       (``drift_v``).


.. _Hit merging:

Hit merging
:::::::::::

In some occasions, a time slice might have no SiPMs recorded. This
means that all SiPM waveforms have been disregarded in :doc:`irene` or
that no SiPM has a charge above ``q_thr`` in such slice. However,
because the slice is considered part of a peak, there is some energy
associated to it, which should not be discarded. In these cases,
*Sophronia* generates a fake hit (a.k.a. *NN-hits*) with no charge and
no position associated [3]_.

After all hits are generated, a second step is performed, in which the
fake hits are merged with existing valid hits. This operation searches
for the hits that are closest (in z) to the fake one. Each fake hit's
energy is shared among its neighbours and added to the neighbouring
slice energy. The energy is not share homogeneously, but
proportionally to the charge of each hit. The parameter ``same_peak``
controls whether only slices from the same peak are taken into account
or if other peaks might also be taken into consideration.

.. [3] Actually, the algorithm associates an unphysical charge (NN =
       -999999) and a position :math:`x = y = 0`.

.. _Hit correction:

Hit correction
::::::::::::::

If a correction map is provided (``corrections_file``), the
geometrical and lifetime corrections are applied. These corrections
are composed of three factors:

- geometrical: accounts for different light collection efficiencies in
  x,y
- lifetime: accounts for the loss of electrons (and therefore signal)
  due to electron attachment during the drift
- temporal: changes in the previous two corrections over time. This is
  applied only in data, as MC has no temporal dependence.

The geometrical correction is a 2d-function: given the hit position in
x,y, we obtain a factor that normalizes the response to the center of
the chamber. It may also scale the response to different units
(pes-to-keV, for instance).

The lifetime correction is a 3d-function: given the hit position in
x,y,z we obtain the correction factor that normalizes the response to
that of z=0.

The temporal correction is a 1d-function: given the time since the
start of the run, we estimate variations in the geometrical and
lifetime corrections that normalize the response to that of the
beginning of the run. This correction is only applied if
``apply_temp`` is set to ``True``.
