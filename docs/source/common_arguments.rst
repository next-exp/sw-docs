Common arguments to every city
==============================

All cities in IC require at least the following arguments

.. list-table::
   :widths: 40 120 120
   :header-rows: 1

   * - Parameter
     - Type(s)
     - Description

   * - ``files_in``
     - ``str`` or ``Sequence[str]``
     - Input file name(s).

   * - ``file_out``
     - ``str``
     - Output file name.

   * - ``compression``
     - ``str``
     - Compression option. Always ``"ZLIB4"``.

   * - ``event_range``
     - ``int``,  ``(int, int)``, ``(int, last)`` or ``all``
     - Range of events to process. If an integer N is provided, the first N events are taken. Two integers (N, M) will run from event N to event M. If (N, ``last``) the first N events will be skipped. If ``all``, all events will be processed.

   * - ``print_mod``
     - ``int``
     - How frequently to print progress to the std output.

   * - ``detector_db``
     - ``str``
     - Name of the detector for database access.

   * - ``run_number``
     - ``int``
     - Run number corresponding to the data. Needed to load the appropriate sensor parameters. If negative, the processing is assumed to be a MC run with the corresponding to the detector conditions of run ``abs(run_number)``.
