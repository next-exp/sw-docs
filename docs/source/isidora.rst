Isidora
==========

*From ancient Greek, Ἰσίδωρος: gift of Isis.*

Due to the bias configuration of the PMTs, the PMT waveform does not represent the actual signal produced by the PMT, but its derivative. This effect is treated online (for trigger) and offline (for data processing) by means of the BaseLine Restoration (BRL) algorithm. *Isidora* processes **RWF**\ s coming from either the *Decoder* (detector) or from *Diomira* (MC) and applies the BLR algorithm to the waveforms. The output are the so-called Corrected Waveforms, or **CWF**.

.. note::
  *Isidora*'s job is also performed by *Irene*, but the latter does not store the intermediate step. *Isidora* is therefore meant as a **debugging tool** to inspect the performance of the BLR algorithm.

.. _Isidora input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/RD/pmtrd``
 * ``/RD/sipmrd``

.. _Isidora output:

Output
------

 * ``/BLR/pmtcwf``: the PMT corrected waveforms
 * ``/BLR/sipmrwf``: the SiPM raw waveforms (a copy of ``/RD/sipmrd``)

.. _Isidora config:

Config
------

Besides the :ref:`Common arguments to every city`, *Isidora* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``n_baseline``
     - ``int``
     - Number of waveform samples to compute the baseline.


.. _Isidora workflow:

Workflow
--------

*Isidora* performs only one task: the Deconvolution of PMT waveforms. Please read :ref:`the corresponding section <Deconvolution of PMT waveforms>` in the *Irene* page.
