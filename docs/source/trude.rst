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

* ``/HIST/sipm_dark``: histogram values of the dark spectrum.
* ``/HIST/sipm_dark_bins``: bin  values of the dark spectrum.
* ``/HIST/sipm_spe``: histogram values of the light spectrum.
* ``/HIST/sipm_spe_bins``: bin  values of the light spectrum.

.. _Trude config:

Config
------

Besides the :ref:`Common arguments to every city`, *Trude* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``min|max_bin``
     - ``float``
     - Lower/upper limit of the number of ACDs of the waveform to be considered for the spectrum.

   * - ``bin_width``
     - ``int``
     - Width of the bins for the spectrum.

   * - ``proc_mode``
     - ``string``
     - Two options of subtracting the baseline: ``subtract_mode`` to subtract the mode and ``subtract_median`` to subtract the median.

   * - ``number_integrals``
     - ``int``
     - Number of integrals to be performed in the waveform's time window. Typically this value is selected depending on the number of LED pulses in the waveform.

   * - ``integral_start``
     - ``int``
     - Bin where the integrals start.

   * - ``integral_width``
     - ``int``
     - Number of bins to be considered for the integral starting at ``integral_start``. With ``integral_start`` these values are ideally chosen to select the whole pulse of the LED.

   * - ``integrals_period``
     - ``int``
     - Period between integrals. This parameter won't be considered when ``number_integrals = 1``.


.. _Trude workflow:

Workflow
--------

Trude performs the following operations to obtain the spectra:

 * :ref:`Selection of integration limits <Selection of integration limits>`
 * :ref:`Baseline subtraction of SiPM waveforms <Baseline subtraction of SiPM waveforms>`
 * :ref:`Spectrum histogram <Spectrum histogram>`


.. _Selection of integration limits:

Selection of integration limits
::::::::::::::::::::::::::::::::

Given the config parameters, the first step in this city would be to selected the dark and light bins to compute the dark and light spectra.

The light bins will be centered in the light pulse(s) and the dark bins are chosen as an interval of ``integral_width`` bins which ends 2 microseconds before the light pulse begins, as shown here:

 .. image:: images/trude/wf_intervals.png
   :width: 850

.. _Baseline subtraction of SiPM waveforms:

Same procedure as described in :ref:`Baseline subtraction of SiPM waveforms` section of the *Irene* documentation with the option of using the mean instead of the mode of the waveform for the baseline subtraction.

Spectrum histogram
:::::::::::::::::

The last step would be the integration of the dark and light bins in order to obtain the respective spectrum histograms.

For each of the regions it sums all the ADCs in the intervals and adds an entry to the histogram. It repeats this for each waveform of the same sensor.

The end result will be a h5 file with ``/HIST/sipm_dark`` and ``/HIST/sipm_spe`` with a table per time bin and an entry per sensor.
