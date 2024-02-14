Hypathia
========

*From ancient Greek ‘Υπατια: highest, supreme.*

*Hypathia* is the combination of simplified versions of :doc:`diomira` and :doc:`irene`.
For future or hypothetical detectors, the energy plane electronics is not necessarily defined. In order to produce **RWF**\ s, we apply a generic and simplified model for the electronics to the **TWF**\ s produced by :doc:`buffy` (full simulation) or :doc:`detsim` (fast simulation). These effects are:

 * Gaussian noise for the PMT baseline (not really, but it should)
 * Charge fluctuations for photoelectron signals of PMTs
 * Noise sampled from PDFs for SiPMs

The **RWF**\ s produced here are not stored persistently, but rather fed into the peak search algorithm. This is equivalent to *Irene*, without the first step of deconvolution of the effects of the energy plane electronics. A detailed description of this procedure can be found in :doc:`irene`. The final product are **PMap**\ s.

.. _Hypathia input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/RD/pmtrd``
 * ``/RD/sipmrd``

.. _Hypathia output:

Output
------

Same as :doc:`irene`:

 * ``/PMAPS/S1``: the sliced PMT-summed waveform for each S1 peak. 4 columns: event number, peak number, time (:math:`\mu`\ s) and amplitude (pes)
 * ``/PMAPS/S1Pmt``: the sliced individual PMT waveforms for each S1 peak. 4 columns: event number, peak number, pmt index (`npmt`) [1]_ and amplitude (pes)
 * ``/PMAPS/S2``: the sliced PMT-summed waveform for each S2 peak. 4 columns: event number, peak number, time (:math:`\mu`\ s) and amplitude (pes)
 * ``/PMAPS/S2Pmt``: the sliced individual PMT waveforms for each S2 peak. 4 columns: event number, peak number, pmt index (`npmt`) [1]_ and amplitude (pes)
 * ``/PMAPS/S2Si``: the sliced individual SiPM waveforms for each S2 peak. 4 columns: event number, peak number, sipm index (`nsipm`) [1]_ and amplitude (pes)
 * ``/Filters/empty_pmap``: flag for whether an event passed the empty pmap filter
 * ``/Filters/s12_indices``: flag for whether an event passed the s12 indices filter

 .. [1] The sensor index is the index in the corresponding database table (DataPMT or DataSiPM in load_db.py). In the case of PMTs, it coincides with the sensor ID.

.. _Hypathia config:

Config
------

Besides the :ref:`Common arguments to every city`, *Hypathia* shares most of its parameters with *Irene*. The only exceptions are `n_baseline`, `n_mau`, and `thr_mau`. Moreover, the city has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``sipm_noise_cut``
     - ``float``
     - Threshold for SiPM noise suppression in pes.

   * - ``filter_padding``
     - ``int``
     - Number of samples to keep before and after samples passing the zero-suppression filter for SiPMs.

   * - ``pmt_wfs_rebin``
     - ``int``
     - Rebin factor to apply to PMT waveforms to match DAQ sampling.

   * - ``pmt_pe_rms``
     - ``float``
     - Standard deviation in pes of the 1 pe peak in the Single PhohoElectron (SPE) spectrum.

   * - ``thr_sipm``
     - ``float``
     - Threshold for individual SiPM samples. Can be absolute (pes) or relative (unitless), depending on ``thr_sipm_type``.

   * - ``thr_sipm_type``
     - ``ThresholdSiPM``
     - Thresholding mode for individual SiPM samples. ``common`` applies the same absolute threshold value to all SiPMs. ``individual`` uses a relative value based on the noise spectrum for each SiPM.

   * - ``s1|s2_lmin|lmax``
     - ``int``
     - Lower/upper limits to the width of S1/S2 signals expressed in number of samples.

   * - ``s1|s2_tmin|tmax``
     - ``float``
     - Lower/upper limits of the search window for S1/S2 signals.

   * - ``s1|s2_rebin_stride``
     - ``int``
     - Rebin factor for S1/S2 signals. Rarely changed. 1 for S1 and 40 for S2 signals.

   * - ``s1|s2_stride``
     - ``int``
     - Allowed range of signal fluctuations below threshold for peak merging expressed in number of samples.

   * - ``thr_csum_s1|s2``
     - ``float``
     - Threshold applied to the PMT-summed waveform in order to find S1/S2 peaks.

   * - ``thr_sipm_s2``
     - ``float``
     - Threshold applied to the time-integrated signal of each SiPM to discard SiPMs with only dark counts.

   * - ``pmt_samp_wid``
     - ``float``
     - Sampling period of PMTs. Should be removed.

   * - ``sipm_samp_wid``
     - ``float``
     - Sampling period of SiPMs. Should be removed.


.. _Hypathia workflow:

Workflow
--------

Hypathia performs a number of data transformations in order to obtain a **PMap**. These operations can be grouped in three main tasks, performed in the following order:

 * :ref:`Simulation of PMT waveforms <Simulation of PMT waveforms>`
 * :ref:`Simulation of SiPM waveforms <Simulation of SiPM waveforms>`
 * :ref:`Computation of PMaps <Computation of PMaps>`


.. _Simulation of PMT waveforms:

Simulation of PMT waveforms
:::::::::::::::::::::::::::

The **TWF**\ s produced with MC simulations (either full + *Buffy* or fast + *Detsim*) are not necessarily sampled at the same period as the DAQ. Thus, the first step is to ensure that they are sampled at the same rate. This is controlled by the parameter `pmt_wfs_rebin`. Waveforms sampled with a period :math:`p` are resampled with a period `pmt_wfs_rebin`  :math:`\cdot p`. This operation can only be performed for `pmt_wfs_rebin` :math:`\geq 1`. For instance, if the MC simulation is performed with a binning of 1 ns, we need to set `pmt_wfs_rebin` to 25 to obtain waveforms sampled at 25 ns.

The next step is to simulate the fluctuations on the PMT response for photoelectrons. For a time bin with charge :math:`q` the resulting charge comes from sampling a gaussian with :math:`\mu = q` and :math:`\sigma = \sqrt{q}\ \cdot` `pmt_pe_rms`. The resulting charge is clipped at 0 to avoid unphysical signals. The following image shows a (fake) PMT waveform with and without charge fluctuation. The algorithm is applied only to non-empty bins.

 .. image:: images/hypathia/charge_fluctuation_example.png
   :width: 850


.. _Simulation of SiPM waveforms:

Simulation of SiPM waveforms
::::::::::::::::::::::::::::

SiPM waveforms are always sampled at 1 :math:`\mu s` in simulations [#]_ and therefore do not need to be resampled. These waveforms are processed to have a charge fluctuation analogous to the one described for PMTs above. In this case the rms parameter is taken from the measured values stored in the database (through the `detector_db` and `run_number` parameters). Then, noise is added to the waveforms by sampling the individual noise distribution of each SiPM, also stored in the database.

Finally, a zero suppression algorithm is applied to mimic the DAQ bahaviour. The samples of the SiPM waveforms with amplitude below `sipm_noise_cut` are set to zero. However, in the vecinity of a sample that survives the cut the waveform is not zero suppresed. This is controlled by the parameter `filter_padding`, which is the number of samples preserved before and after a sample that survives the zero suppression cut. This is exemplified in the following image. The time bins with charge above the threshold are unmodified, while those below it are set to 0, with the exception of those falling in the green region.

 .. image:: images/hypathia/noise_suppression.png
   :width: 850

.. _Computation of PMaps:

Computation of PMaps
::::::::::::::::::::

This procedure is identical to that performed by *Irene*. For more information read the sections :ref:`Baseline subtraction of SiPM waveforms`, :ref:`Waveform calibration` and :ref:`Peak finding and matching of PMT and SiPM signals` in the *Irene* documentation.

 .. [#] So far we haven't had the need to explore different SiPM sampling rates, but if this becomes a possibility  in the future it can always be included. If you would like to implement it, let us know!
