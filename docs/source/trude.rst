Trude
==========

*From Germanic, Gertrud, hypocorism: female friend.*

This city produces the light and dark spectrum of SiPMs for dedicated calibration runs. This is achieved by selecting regions in the SiPM waveforms where LED pulses are expected and regions which end 2 microseconds before, respectively and integrating the content within each region. The regions before LED pulses should only contain electronics noise and dark counts giving the zero external light approximation whereas those in time with the pulses will contain one or more detected photoelectrons. The waveform integrals are split into two groups: those with expected photoelectrons (light) and those without expected photoelectrons (dark). Each group produces a different spectrum.

.. _Trude input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/RD/sipmrwf``

.. _Trude output:

Output
------

##############por  aqui

* ``/HIST/sipm_dark``: histogram values of the dark spectrum.
* ``/HIST/sipm_dark_bins``: bin  values of the dark spectrum.
* ``/HIST/sipm_spe``: histogram values of the light spectrum.
* ``/HIST/sipm_spe_bins``: bin  values of the light spectrum.

.. _Trude config:

Config
------

Besides the :ref:`Common arguments to every city`, *Irene* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``min|max_bin``
     - ``float``
     - Lower/upper limit the number of ACDs of the waveform to be considered for the spectrum.

   * - ``bin_width``
     - ``int``
     - Width of the bins for the spectrum.

   * - ``proc_mode``
     - ``string``
     - Two options of subtracting the baseline. ``subtract_mode`` to subtract the mode and ``subtract_median`` to subtract the median.

   * - ``number_integrals``
     - ``int``
     - Number of integrals to be performed in the waveforms time window. Typically this value is selected depending on the number of LED pulses to have in the waveform in the same time window.

   * - ``integral_start``
     - ``int``
     - Bin where the integrals start.

   * - ``integral_width``
     - ``int``
     - Number of bins to be considered for the integral starting at ``integral_start``. With ``integral_start`` these values are ideally chosen to select the whole pulse of the LED.

   * - ``integrals_period``
     - ``int``
     - Period between integrals. This parameter won't be considered when ``number_integrals = 1``


.. _Irene workflow:

Workflow
--------

Irene performs a number of data transformations in order to obtain a **PMap**. These operations can be grouped in four main tasks, performed in the following order:

 * :ref:`Deconvolution of PMT waveforms <Deconvolution of PMT waveforms>`
 * :ref:`Baseline subtraction of SiPM waveforms <Baseline subtraction of SiPM waveforms>`
 * :ref:`Waveform calibration <Waveform calibration>`
 * :ref:`Peak finding and matching of PMT and SiPM signals <Peak finding and matching of PMT and SiPM signals>`


.. _Deconvolution of PMT waveforms:

Deconvolution of PMT waveforms
::::::::::::::::::::::::::::::

Due to the bias configuration of the PMTs, the PMT waveform does not represent the actual signal produced by the PMT, but its derivative (for details see [1]_). The typical PMT **RWF** for a Kr event looks like this:

 .. image:: images/irene/pmt_rwf.png
   :width: 850

This waveform needs to be transformed into a unipolar (positive-defined) zero-baseline waveform whose area is proportional to the number of photons detected. The part of the waveform corresponding to when the PMT doesn't receive any light is just a gaussianly-distributed noise around a baseline value. This value is estimated using the first few microseconds of the waveform; the amplitude is averaged over this time frame and subtracted from the entire waveform to produce a baseline-subtracted waveform.

The resulting waveform is still bipolar. This is addressed by the deconvolution algorithm (BLR). This process is fairly complex, but in simple terms, it consists of a high-pass filter and a signal accumulator, which inverts the effect of the PMT electronics. For greater detail on the PMT electronics and the recovery algorithm see [1]_. Finally, the polarity of the waveform is inverted to make it positive.

All the aforementioned steps are performed for each PMT separately. The output of this algorithm are the so-called *Corrected waveforms* (**CWF**\ s).

The city :doc:`isidora` allows the user to run just this stage of the reconstruction and store the **CWF**\ s for further study. Irene however, does not store them and they are fed directly into the rest of the PMap-building algorithm. The **CWF** corresponding to the **RWF** shown above is:

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

The production and manufacturing of the sensors and other electronic components does not guarantee a homogeneous response among all sensors. Thus, the waveforms are calibrated to equalize their response. The calibration consists of a constant for each sensor indicating the number of ADC corresponding to a photoelectron (calibration constant), which is a physical quantity common to all of them. The calibration technique is similar for PMTs and SiPMs. For details about the calibration procedure see the calibration cities: :doc:`berenice`, :doc:`phyllis` and :doc:`trude`.

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

These data are stored in an hdf5 file in 5 separate tables under a common group ``PMAPS``. See the :ref:`output <Irene output>` section for a full description.

 .. [1] `The electronics of the energy plane of the NEXT-White detector <https://arxiv.org/pdf/1805.08636.pdf>`_
 .. [#] The waveform at this point is in ADC, therefore, they are integer values.
 .. [#] The noise in the PMT waveforms is gaussianly distributed around the baseline with a standard deviation :math:`\sigma_{PMT}`. Assuming similar values of :math:`\sigma_{PMT}`, the addition of the PMT waveforms results in a waveform with a standard deviation :math:`\sqrt{n_{PMT}}\ \sigma_{PMT}`. However, the signal increases linearly with the number of sensors and therefore the signal-to-noise ratio improves as :math:`\sqrt{n_{PMT}}`
 .. [#] These two thresholds together reduce the data stored by a factor ~100.
