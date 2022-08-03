Buffy
==========

*sorts MC sensors info into buffers, slays vampires, etc*

Full simulations generate sensors (PMTs and SiPMs) response,
including the time and the detected charge of a specific signal. However, this sensor
information does not have the waveform format obtained from data taking.
*Buffy* takes nexus sensors information, and sorts them into *true waveforms* (**TWF**).
**TWF**\ s represent the signal amplitude of a given sensor without any type of
distortion or effect, within a certain time interval and given the sensor sampling time.
This type of time ordering of the sensor signal in a data-like format is what is called *bufferisation*.
**TWF**\ s generated from *Buffy* can be transformed into *raw waveforms* **RWF**\ s with :doc:`diomira`.
For detector geometries without sensor electronics, **TWF**\ s can be transformed directly into **PMap**\ s
with :doc:`hypathia`.

.. _Buffy input:

Input
-----

 * ``/MC/hits``
 * ``/MC/particles``
 * ``/MC/sns_response``

.. _Buffy output:

Output
------

 * ``/Run/runInfo``: run info table
 * ``/Run/events``: event info table
 * ``/Run/eventMap``: event and nexus event for each index
 * ``/RD/pmtrd/``: time ordered signal amplitude of the PMTs in true photoelectrons (PMT buffers). array: number of events, number of PMTs, length of PMT waveform (:math:`\mu`\ s).
 * ``/RD/sipmrd/``: time ordered signal amplitude of the SiPMs in true photoelectrons (SiPM buffers). array: number of events, number of SiPMs, length of SiPM waveform (:math:`\mu`\ s).

.. _Buffy config:

Config
------

Besides the :ref:`Common arguments to every city`, *Buffy* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``max_time``
     - ``int``
     - Maximal length of the event that will be taken into account starting from the first detected signal. All signals after that are lost. Must be greater than ``buffer_length``. If not, raises a warning and set ``max_time`` == ``buffer_length``

   * - ``buffer_length``
     - ``float``
     - Configured buffer length in :math:`\mu`\ s.

   * - ``pre_trigger``
     - ``float``
     - Time in buffer before identified signal in :math:`\mu`\ s.

   * - ``trigger_threshold``
     - ``int``
     - Trigger threshold for selection in pes.


.. _Buffy workflow:

Workflow
--------
For full simulations, NEXUS ``/MC/sns_response`` table stores for each event (``event_id``) the list of sensors (``sensor_id``) that detect a specific number of photons (``charge``) in an ordered number of the time bin (``time_bin``).

 .. image:: images/buffy/MC_sns_response_table.png
   :width: 300

This type of tables do not have the same shape that the waveforms collected when data taken, and in addition they only provide the information from the sensors and time bins when some charge is detected. Buffy takes this information (``event_id``, ``sensor_id`` and ``time_bin``), and transforms it into the waveform shape of the **TWF** expected for each type of sensor: ``pmtrd`` and ``sipmtrd`` (PMTs and SiPMs respectively) based on the *bufferisation* parameters provided.

.. image:: images/buffy/pmt_wft.png
  :width: 350
.. image:: images/buffy/sipm_wft.png
  :width: 350


This process is separated in the following tasks in the city:

• :ref:`Histogram creation <Histogram>`
• :ref:`Signal Search <Signal-Search>`
• :ref:`Synchronisation and trigger separation <Trigg-Separation>`

.. note::
  Historically, Buffy is based in an initial code of detsim (https://github.com/next-exp/IC/tree/master/invisible_cities/detsim) and most of its functions are located in that path but they are independent to :doc:`detsim` city.


.. _Histogram:

Histogram creation
::::::::::::::::::

Takes NEXUS information about sensor hits (``/MC/sns_response``). Checks time stamp of an event according to the sensors response, and defines a histogram between [:math:`t_{min}`, :math:`t_{max}`], being:

• :math:`t_{min}`: the time stamp of the first charge deposition of the event,
• :math:`t_{max}`: defined considering that ``max_time`` =  :math:`t_{max}` - :math:`t_{min}`.

Once these histograms are defined, they are sampled according to the binning of each sensor (``pmt_width`` and ``sipm_width``). Sampling widths are included in the simulation parameters (``/MC/info``), and depends on the type of sensor and detector. Normally corresponds to 25 :math:`\mu`\ s for PMTs and and 1 :math:`\mu`\ s for SiPMs.

.. _Signal-Search:

Signal Search
::::::::::::::::::

Charge is distributed in previously defined histograms. The code searches for signal-like charge according to a given threshold (``trigger_threshold``). Once it is found, it defines a trigger time, :math:`t_{trigger}`. PMT/SiPM sum is ordered by the given buffer length (``buffer_length``) considering :math:`t_{trigger}` and the ``pre_trigger`` configuration. Simple threshold value is applied on binned charge.

.. image:: images/buffy/bufferisation.png
  :width: 600

Using this information, waveforms are defined for each sensor in a specific length based on ``buffer_length``/``sensor_width``.

.. _Trigg-Separation:

Synchronisation and trigger separation
:::::::::::::::::::::::::::::::::::::::

Since the ``sensor_width`` is different for each sensor, it is necessary to align and synchronises the clocks between SiPMs and PMTs. Waveforms are sliced then according to binning (``pmt_width`` and ``sipm_width``), trigger time and configured pre-trigger (``pre_trigger``). In addition, it pads with zeross where it is necessary: arrays with zeros where there is no recorder pes in nexus (in the other sensors). Arrays with zeros where no signal and nexus recorded pes otherwise. If more than one trigger is found separated from each other by more than a buffer width, the nexus event can be split into multiple data-like triggers.
