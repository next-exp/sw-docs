Irene
=====

*From ancient Greek, Εἰρήνη, meaning peace.*

The majority of a sensor's waveform does not contain any useful information. The S1 and S2 signals are localized to relatively short time intervals. Thus, *Irene* processes **RWF**\ s to find these time slices (peaks) and disregard the rest of the waveform. During this procedure, PMT and SiPM waveforms are matched and combined into a single structure. The collection of all peaks in an event is called a Peak-map or **PMap**.

.. _Irene input:

Input
:::::

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/RD/pmtrd``
 * ``/RD/sipmrd``

.. _Irene output:

Output
::::::

 * ``/PMAPS/S1``
 * ``/PMAPS/S1Pmt``
 * ``/PMAPS/S2``
 * ``/PMAPS/S2Pmt``
 * ``/PMAPS/S2Si``
 * ``/Filters/empty_pmap``
 * ``/Filters/s12_indices``

.. _Irene config:

Config
::::::

Besides the arguments common to all cities <link>, *Irene* accepts the following arguments:

.. list-table::
   :widths: 40 60 120
   :header-rows: 1

   * - Parameter
     - Type
     - Description

   * - ``n_baseline``
     - ``int``
     - Number of waveform samples to compute the baseline.

   * - ``n_mau``
     - ``int``
     - Number of waveform samples to compute the mau.

   * - ``thr_mau``
     - ``float``
     - Threshold for mau calculation in pes.

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


.. _Irene workflow:

Workflow
--------------

Irene performs a number of data transformations in order to obtain a **PMap**. These operations can be grouped in four main tasks:

 * :ref:`Deconvolution of PMT waveforms <Deconvolution of PMT waveforms>`
 * :ref:`Baseline subtraction of SiPM waveforms <Baseline subtraction of SiPM waveforms>`
 * :ref:`Waveform calibration <Waveform calibration>`
 * :ref:`Peak finding and matching of PMT and SiPM signals <Peak finding and matching of PMT and SiPM signals>`


.. _Deconvolution of PMT waveforms:

Deconvolution of PMT waveforms
::::::::::::::::::::::::::::::

Due to the bias configuration of the PMTs, the PMT waveform does not represent the actual signal produced by the PMT, but its derivative (for details see <NEW energy plane paper>). The typical PMT **RWF** for a Kr event looks like this:

 .. image:: images/irene/pmt_rwf.png
   :width: 850

This waveform needs to be transformed into a unipolar (positive-defined) zero-baseline waveform whose area is proportional to the number of photons detected. The part of the waveform corresponding to when the PMT doesn't receive any light is just a gaussianly-distributed noise around a baseline value. This value is estimated using the first few microseconds of the waveform; the amplitude is averaged over this time frame and subtracted from the entire waveform to produce a baseline-subtracted waveform.

The resulting waveform is still bipolar. This is addressed by the deconvolution algorithm (BLR). This process is fairly complex, but in simple terms, it consists of a high-pass filter and a signal accumulator, which inverts the effect of the PMT electronics. For greater detail on the PMT electronics and the recovery algorithm see <NEW energy plane paper>. Finally, the polarity of the waveform is inverted to make it positive.

All the aforementioned steps are performed for each PMT separately. The output of this algorithm are the so-called *Corrected waveforms* (**CWF**\ s).

The city *Isidora* allows the user to run just this stage of the reconstruction and store the **CWF**\ s for further study. Irene however, does not store them and they are fed directly into the rest of the PMap-building algorithm. The **CWF** corresponding to the **RWF** shown above is:

 .. image:: images/irene/pmt_cwf.png
   :width: 850


.. _Baseline subtraction of SiPM waveforms:

Baseline subtraction of SiPM waveforms
::::::::::::::::::::::::::::::::::::::

Unlike PMTs, SiPM waveforms are already unipolar and positive-defined. The baseline computation for SiPMs is slightly different. Instead of averaging a fraction of the waveform, the mode [#]_ of the entire waveform is used. The baseline is estimated and substracted on an event-by-event basis and for each SiPM independently. The following figure shows a comparison between a SiPM **RWF** and a baseline-subtracted SiPM waveform.

 .. image:: images/irene/sipm_rwf.png
   :width: 850


.. _Waveform calibration:

Waveform calibration
::::::::::::::::::::

The production and manufacturing of the sensors and other electronic components does not guarantee a homogeneous response among all sensors. Thus, the waveforms are calibrated to equalize their response. The calibration consists of a constant for each sensor indicating the number of ADC corresponding to a photoelectron (calibration constant), which is a physical quantity common to all of them. The calibration technique is similar for PMTs and SiPMs. For details about the calibration procedure see <reference to calibration procedures>.

The calibration constants are measured regularly while the detector is in operation. The calibration constants are fetched from the database automatically and indexed by run number.

The calibration step is rather simple. The **CWF** of each PMT and the baseline-subtracted waveform of each SiPM are scaled up according to their corresponding calibration constants. The resulting set of waveforms are sometimes called **CCWF**\ s (*Calibrated Corrected Waveforms*).


.. _Peak finding and matching of PMT and SiPM signals:

Peak finding and matching of PMT and SiPM signals
:::::::::::::::::::::::::::::::::::::::::::::::::

The peak finding and waveform slicing is arguably the most complex part of the **RWF** processing. The algorithm must be able to find two very different types of signals (S1 and S2), while accurately establishing the limits on those peaks to maintain the energy resolution capabilities of the detector.

In order to optimize the peak search, PMT **CCWF**\ s are used as they have a higher sampling rate and therefore better time resolution. On top of that, these waveforms are PMT-summed to increase the signal-over-noise ratio [#]_. S1 and S2 signals are searched independently.

The PMT-summed waveform is searched for samples above a certain threshold (``thr_csum_sX``), which may depend on the event type. The samples below the threshold are initially ignored. However, fluctuations in the PMT signal close to the threshold can lead to a split in an otherwise continuous peak. This is particularly relevant for S1 signals due to their small amplitude in low-energy events.
To minimize this effect, signal regions separated by a short time (configurable via the ``sX_stride`` arguments) are joined back together. This stride may also depend on the event type.
In order to reduce the amount of spurious or unphysical peaks, the search can be restricted to certain time spans (``sX_tmin``, ``sX_tmax``) in the waveform.
Furthermore, the resulting peaks are filtered based on their width (via ``sX_lmin``, ``sX_lmax``), improving the efficiency of finding peaks corresponding to a true signal.
The beginning and end of the signal region is kept for each peak. This information is then used to slice each PMT and SiPM waveforms.

To create a S2 peak, the sliced PMT waveforms are resampled according to ``s2_rebin_stride``. By default, this resamples from 40 MHz (25 ns) to 1 MHz (1 :math:`\mu`\ s) to match the sampling rate of SiPMs. Also, SiPMs are noisier than PMTs, producing spurious photoelectron pulses. In order to minimize this effect, a threshold ``thr_sipm`` is applied to each sample of each SiPM, suppressing values below it. This threshold can be ``common`` to all SiPMs, or applied to each ``individual`` SiPM, based on their measured noise spectrum. This behaviour can be controlled via the ``thr_sipm_type`` argument. Finally, due to the characteristics of the tracking plane, most SiPMs don't contain signal. Hence, another threshold ``the_sipm_s2`` is applied to the time-integrated signal of each SiPM for a given peak [#]_.
The resulting PMT and SiPM waveforms are then time-matched and stored in a single object (``Peak``).

S1 signals on the other hand, are weak enough to be detected only by PMTs, therefore the SiPMs are ignored during the S1 search. The waveforms can also be resampled using the ``s1_rebin_stride``, however this parameter is usually set to 1 to keep the optimal time resolution of S1 signals.

The following figure shows the performance of this algorithm on a typical Kr event.

  .. image:: images/irene/s1_identification.png
    :width: 32%
  .. image:: images/irene/s2_identification_pmt.png
    :width: 32%
  .. image:: images/irene/s2_identification_sipm.png
    :width: 32%

Finally all peaks are stored in a single ``PMap`` object. A ``PMap`` contains a list S1 peaks and a list of S2 peaks. Each Peak contains the times of the samples within the peak and a ``SensorResponse`` object for PMTs a ``SensorResponse`` object for SiPMs. Each ``SensorResponse`` object contains the IDs and the sliced waveforms of each sensor that contains signal in an event.

These data are stored in a file in 5 separate tables under a common group ``PMAPS``:

 * ``S1``: contains the sliced PMT-summed waveform for each S1 peak. 4 columns: event number, peak number, time (:math:`\mu`\ s) and amplitude (pes)
 * ``S2``: contains the sliced PMT-summed waveform for each S2 peak. 4 columns: event number, peak number, time (:math:`\mu`\ s) and amplitude (pes)
 * ``S1Pmt``: contains the sliced individual PMT waveforms for each S1 peak. 4 columns: event number, peak number, pmt id and amplitude (pes)
 * ``S2Pmt``: contains the sliced individual PMT waveforms for each S2 peak. 4 columns: event number, peak number, pmt id and amplitude (pes)
 * ``S2Si``: contains the sliced individual SiPM waveforms for each S2 peak. 4 columns: event number, peak number, sipm id and amplitude (pes)

 .. [#] The waveform at this point is in ADC, therefore, they are integer values.
 .. [#] The noise in the PMT waveforms is gaussianly distributed around the baseline with a standard deviation :math:`\sigma_{PMT}`. Assuming similar values of :math:`\sigma_{PMT}`, the addition of the PMT waveforms results in a waveform with a standard deviation :math:`\sqrt{n_{PMT}}\ \sigma_{PMT}`. However, the signal increases linearly with the number of sensors and therefore the signal-to-noise ratio improves as :math:`\sqrt{n_{PMT}}`
 .. [#] These two thresholds together reduce the data stored by a factor ~100.
