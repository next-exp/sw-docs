Dorothea
========

*From ancient Greek, Δωροθέα: gift of God.*

The purpose of *Dorothea* is to process the **PMaps** (outcome of *Irene*) and
reconstruct point-like events. Therefore, all the information included in the S2 peaks
is collapsed into a single x, y, and z deposition with a given E (the area under the S2 peak).
Additionally, some parameters regarding the S1 and S2 peaks are provided as a tool for further selections.
Last, it is worth noting that for events where more than S1 or S2 (or both) are
present in the event, a point is reconstructed for each possible combination.


.. _Dorothea input:

Input
-----

* ``/Run/events``
* ``/Run/runInfo``
* ``/PMAPS/S1``
* ``/PMAPS/S1Pmt``
* ``/PMAPS/S2``
* ``/PMAPS/S2Pmt``
* ``/PMAPS/S2Si``

.. _Dorothea output:

Output
------

 * ``/DST/Events``: summary of the reconstructed events and their main properties. Each row represents one combination of an S1 and a S2 identified under the same ``event`` but with its corresponding  ``s1_peak`` and ``s2_peak`` identifier. The full list and description of the parameters in the table may be found later.
 * ``/Filters/s12_selector``: flag for whether an event passed the S1 and S2 selections


.. _Irene config:

Config
------

Besides the :ref:`Common arguments to every city`, *Dorothea* has the following arguments:

.. list-table::
   :widths: 50 40 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``drift_v``
     - ``float``
     - Drift velocity of the secondary electrons towards the anode.

   * - ``s1|s2_nmin|nmax``
     - ``int``
     - Lower/upper limits to the number of S1/S2 peaks per event.

   * - ``s1|s2_emin|emax``
     - ``float``
     - Lower/upper limits to the energy (integral under the peak) of each S1/S2 peak (pes).

   * - ``s1|s2_wmin|wmax``
     - ``float``
     - Lower/upper limits to the with (time over threshold) of each S1/S2 peak (ns).

   * - ``s1|s2_hmin|hmax``
     - ``float``
     - Lower/upper limits to the height (maximum amplitude) of each S1/S2 peak (pes).

   * - ``s1|s2_ethr``
     - ``float``
     - Threshold for each bin within each S1/S2 to be considered for the energy computation (pes).

   * - ``s2_nsipmmin|nsipmminax``
     - ``int``
     - Lower/upper limits to the number of SiPM sensors with a recorded signal.

   * - ``global_reco_params``
     - ``dict``
     - Set of parameters for the corona algorithm. Since the desired one is the barycenter reconstruction, the only mandatory elements in the dictionary are:

        * ``Qthr``: charge threshold, ignore all SiPMs with less than Qthr pes.
        * ``lm_radius=-1``: sets the algorithm to be the barycenter.


.. _Dorothea workflow:

Workflow
--------

The workflow for *Dorothea* is straightforward. After a filter removing all the peaks and events not satisfying the limits provided. Then, it proceeds to perform the point-like reconstruction. Basically, this algorithm collapses the whole S2 information into a single point:

 * The *x* and *y* position are determined via a charge-weighted average. This makes use of the so-called ``corona`` `algorithm <https://github.com/next-exp/IC/blob/8be75c65aa2e452eae4ce2e51494a58eab18a0d4/invisible_cities/reco/xy_algorithms.py#L61>`_, with the proper configuration to apply the barycenter computation.
 * The *z* coordinate is derived from the time difference between the maximum amplitude of the S1 and S2 considered, corrected by the drift velocity.
 * The energy is the integral under the S2 of those bins above the threshold.

 Besides, the reconstruction yields a set of useful values regarding the peaks that are also provided in the final table:

 .. list-table::
    :widths: 50 40 120
    :header-rows: 1

    * - **Parameter**
      - **Type**
      - **Description**

    * - ``event``
      - ``int``
      - Event ID. As an event can have several S1 and S2 combinations, each one representing one row, this number identifies all the reconstructed points within the same original waveform.

    * - ``time``
      - ``float``
      - Timestamp of the event.

    * - ``s1_peak``
      - ``int``
      - S1 ID. It identifies the S1 within an event.

    * - ``s2_peak``
      - ``int``
      - S2 ID. It identifies the S2 within an event.

    * - ``nS1``
      - ``int``
      - Number of S1 present in the event.
    * - ``nS2``
      - ``int``
      - Number of S2 present in the event.

    * - ``S1w``
      - ``float``
      - S1 time over threshold (ns).

    * - ``s1h``
      - ``float``
      - S1 maximum amplitude (pes).

    * - ``S1e``
      - ``float``
      - S1 PMT- and time-summed amplitude over threshold (pes).

    * - ``S1t``
      - ``float``
      -  Waveform time at maximum S1 amplitude (ns).

    * - ``S2w``
      - ``float``
      - S2 time over threshold (ns).

    * - ``s2h``
      - ``float``
      - S2 maximum amplitude (pes).

    * - ``S2e``
      - ``float``
      - S2 PMT- and time-summed amplitude over threshold (pes).

    * - ``S2t``
      - ``float``
      -  Waveform time at maximum S2 amplitude (ns).

    * - ``S2q``
      - ``float``
      -  S2 SiPM- and time-summed amplitude over threshold (pes).

    * - ``Nsipm``
      - ``int``
      - Number of SiPMs with signal over threshold.

    * - ``DT``
      - ``float``
      - Drift Time —i.e., time difference between the corresponding S1 and S2— (:math:`\mu s`).

    * - ``Z``
      - ``float``
      - Reconstructed z position coordinate —i.e., the DT times the drift velocity— (mm).

    * - ``Zrms``
      - ``float``
      -  Standard deviation of the PMT  signal in the z coordinate (mm).

    * - ``X``
      - ``float``
      - Reconstructed x position coordinate using the barycenter algorithm (mm).

    * - ``Y``
      - ``float``
      - Reconstructed y position coordinate using the barycenter algorithm (mm).

    * - ``R``
      - ``float``
      - Reconstructed radial coordinate, :math:`r^2=x^2+y^2` (mm).

    * - ``Phi``
      - ``float``
      - Reconstructed azimuthal coordinate, :math:`\phi=\arctan(y/x)` (rad).

    * - ``Xrms``
      - ``float``
      -  Standard deviation of the PMT  signal in the x coordinate (mm).

    * - ``Yrms``
      - ``float``
      -  Standard deviation of the PMT  signal in the y coordinate (mm).
