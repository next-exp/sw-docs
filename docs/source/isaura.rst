Isaura
======

*From ancient greek, Ισαυρία: an ancient rugged region in Asia Minor.*

This city just computes tracks and extracts topology information from **deconvoluted hits**. Therefore, it is analogous to the final stage of :doc:`esmeralda`. Unlike the latter, the input hits correspond to the deconvoluted ones coming from :doc:`beersheba` (instead of :doc:`penthesilea`), allowing us to obtain a finer tracking of events.

.. _Isaura input:

Input
-----

 * ``/Run/events``
 * ``/Run/runInfo``
 * ``/DECO/Events``
 * ``/DST/Events``

.. _Isaura output:

Output
------

 * ``/Tracking/Tracks``: tracking-related information of events (in particular: track energy (``energy``), track length (``length``), number of voxels (``numb_of_voxels``), number of hits (``numb_of_hits``), minimum and maximum position computed with the hits comprising the track (``x_min``, ``y_max``, ...), blobs position (``blob1_z``, ``blob2_x``,...) and energy (``eblob1``, ``eblob2``), energy of the hits shared by both blobs (``ovlp_blob_energy``), and voxel size (``vox_size_x``, ``vox_size_y``, ``vox_size_z``). Each row corresponds to a different track, specified among the others within an event with its ``trackID``
 * ``/Summary/Events``: global information related to the event. Each row is one event
 * ``/DST/Events``: copy of the point-like information (*kdst*) of events, output of :doc:`penthesilea`
 * ``/Filters/hits_select``: flag to indicate if an event did not pass due to the lack of hits
 * ``/Filters/topology_select``: flag that indicates whether the topology has been performed properly 
 * ``MC info``: Monte Carlo information of events. Only if run number < 0

.. _Isaura config:

Config
------

Apart from the :ref:`Common arguments to every city`, the parameters to run *Isaura* are the following:

.. list-table::
   :widths: 50 100 120
   :header-rows: 1

   * - **Parameter**
     - **Type**
     - **Description**

   * - ``vox_size``
     - ``[float,float,float]``
     - X, Y, and Z dimensions of the voxels used in the voxalization of the hits.

   * - ``strict_vox_size``
     - ``bool``
     - Flag to indicate if the size of the voxels is forced to be exactly the values provided in the previous argument (*True*),

       or, on the other hand, if they are allowed to change a bit for each track, aiming to optimize the voxelization process (*False*).

   * - ``energy_threshold``
     - ``float``
     - If the energy of one of the original end-point voxels is smaller than this value,

       the voxel will be dropped and its energy redistributed to the neighbours.

   * - ``min_voxels``
     - ``int``
     - The voxel dropping procedure commented on ``energy_threshold`` can only happen if the number of voxels is larger than the value specified in this argument.

   * - ``blob_radius``
     - ``float``
     - Radius of the blobs.

   * - ``max_num_hits``
     - ``int``
     - Maximum number of hits for an event to be processed.


.. _Isaura workflow:

Workflow
--------

The basic idea behind Isaura is to run the "traditional" *Paolina* algorithm over the deconvoluted hits output in :doc:`beersheba`, in order to convert ``events`` into ``tracks`` with ``blobs``.

The first step within the algorithm consists in checking that the number of hits is lower than the value provided in the config file (``max_num_hits``). That argument was introduced because, when running *Paolina* algorithm after :doc:`penthesilea`, there were some events that comprise such large amount of hits that the tracking information extraction took ridiculously long. As the plot presented below shows [#]_, high energy (trigger2) events usually contain around 200 *Penthesilea hits*, while there are some of them with more than 10000 hits.

 .. image:: images/isaura/nhits_per_evt_r8571.jpg
   :width: 1000

The plot also shows that these events only  appear a few times within a 24h-long low-background run (around 0.05% of the total set of events). Rejecting these type of events is not a particularly worrysome issue: they would be thrown away in the posterior analysis, seeing that none of them are exclusively contained inside the fiducial volume. The spatial distributions for one of these events is presented below (where the grey dashed lines illustrate the boundaries of the chamber):


 .. image:: images/isaura/XY_Z_distributions_evt_many_hits.jpg
   :width: 1000

In any case, one can easily infer from the plots that these events are not physical. On the contrary, they seem to correspond to either some kind of flash occurring in the chamber (like a mini-spark) or some fail in the electronics (after the saturation of an alpha particle, for example). The ID of the events that are removed from the reconstruction chain because of this reason will be specified in the table ``Filters/topology_select``, in order to keep track of this information.

Another obvious condition that must fulfill all events to be processed is to contain hits. If not, the event will be also rejected, which will be displayed in the table ``Filters/hits_select``.

..
 Next step includes another (quite obvious) check: at least one hit inside the event must have a well-defined energy. If not, the event will be also rejected, since no topological information could be extracted.


.. _Exracting the topology-related information:

Extracting the topology-related information
:::::::::::::::::::::::::::::::::::::::::::
As it is well known, an excellent topological discrimation between signal and background (thanks to the usage of a gaseous medium inside the TPC) is one of the fundamental trademarks of the NEXT experiment. In order to do that, it is necessary to (1) separate the different tracks that may form the event, (2) find the extremes for each of them, and (3) compute the energy around these extremes, providing the so-called *blobs*.

In order to compute all the tracking information presented below it will be mandatory first to check that every event contain hits with well-defined energy. For instance, events with all hits outside the krypton correction map bondaries will be thrown away, since their energy cannot be corrected and their ``Ec`` variable (*corrected energy*) will be ``NaN``.

Once the previous check is done, the hits are grouped into 3D volume elements (``voxels``) with the objective of studying the connectivity. The size of these voxels is more or less fixed (depending on the ``strict_vox_size`` parameter in the config file), and their energy correspond to the sum of the energy of the hits included in the voxel. Following a Breadth-First Search (BSF) [#]_ algorithm, the voxels sharing side, edge, or corner will be part of the same **track**. The figure below shows the voxelization result of a real NEXT-White data (Run-VI) single-electron candidate of 1.73 MeV. In this case, after grouping the *deconvoluted hits* into [5 mm x 5 mm x 5 mm] voxels, the event was classified as single-track.


 .. image:: images/isaura/r8250_evt194237_dhits.png
   :width: 46%
 .. image:: images/isaura/r8250_evt194237_voxels.png
   :width: 53%


Seeing that events have already been divided into voxel-made 3D tracks, the next (and final) step comprises the computation of the **blobs** [#]_. This is a crucial stage within the reconstruction chain, since the energy of these elements will allow us to distinguish between double-electron (such as the double-beta signal) and single-electron (the majority of backgrounds) tracks.

There are different ways to define the center of the blobs, the one selected by NEXT is illustrated in the figure below. According to this procedure, the two extreme voxels of the track are localized following the BFS algorithm, on a first stage. The energy-weighted averaged position of the hits inside the voxel will correspond to the **blob center**. From that point, a 3D sphere of radius ``blob_radius`` (specified in the config file) is taken, and all hits (independent to the voxel they belong to) inside that sphere will contribute to the energy of the blob.

 .. image:: images/isaura/blobs_position_definition.png
   :width: 400
   :align: center


To carry out this procedure, and as stated before, the first thing to do is to localize the two end voxels for each track. Defining the distance between any pair of voxels as the shortest path **along the track** that connects them, the two extreme voxels will be the ones with the longest distance between them. However, there are two special cases that are important to comment:


 - It is possible that some spurious **low-energy** hits appear around the track (due to over-iterations during the *Richarson-Lucy* deconvolution process, as commented in :doc:`beersheba`; or some noise inside the chamber, for example). If these hits are reconstructed around the track but not far enough to produce a different S2 or track (taking into account the voxel size), they can be considered as a part of the main one and, being a bit separate, it is probable they end up belonging to an extreme voxel. That case would not be correct, and in order to solve it, the voxel will be dropped from the track and its energy passed to the closest one. This process is only carried out if the voxel energy is lower than: ``energy_threshold`` and the track is made by more than ``min_voxels`` voxels. Once this procedure is done, the extreme voxels are searched and found again recursively, until none of these conditions are fulfilled.

 - Another particular scenario is the one that comes up when there are multiple end-voxel candidates (one can imagine that the shorter the track the more probable is this to happen). To deal with it, the more energetic candidates will be the ones set as extremes. With this convention, we aim to minimize the voxel-dropping algorithm commented just above.


Eventually, once the extreme voxels are properly found, the computation of the center and energy of the blobs (stored in the ``Tracks/Tracking`` table as: ``blobi_x``, ``blobi_y``, ``blobi_z``, and ``eblobi`` (with ``i`` being 1, 2), respectively) will be rather straightforward, just as the previous figure pointed out. The only comment to take into account here is that not every hit falling inside the blob sphere will be considered for its total energy, but only the ones that belong to an adjacent voxel.

As convention, it is worth-remarking that the blob1 and blob2 are defined in such a way that they fulfill: ``eblob1`` > ``eblob2``.


The final step of the *Paolina* algorithm includes the computation of the ``ovlp_blob_energy`` variable. In short tracks it is common to have **overlapping blobs**, i.e. blobs that share some of their hits [#]_. Whenever this happen, the blob energies will not be correctly computed, due to an over-estimation of the total energy of the track. This variable will materialize this effect, providing a really usefull tool to get rid of these cases.


 .. image:: images/isaura/RunVI_b_evt_1720keV_XYZ.jpg
   :width: 1200

The XY (d), XZ (e) and YZ (f) projections of devonvoluted hits, along with the blobs computed with this algortihm, for the same event as the one shown before (in the voxelization plot) can be seen above. This image illustrates how the blobs seem to be computed perfectly. According to our reconstruction, it corresponds to a clear single-electron event (background), due to the noticeable difference between the energy of its blobs: ``eblob1`` = 755 keV, whereas ``eblob2`` = 104 keV.

           
`Isaura` comprises the last step within the NEXT reconstruction chain. Therefore, after it, we have access to all the relevant information to perform the analysis. This information is finally stored in different tables, just as the :ref:`Output <Isaura output>` subsection indicates.




 .. [#] When regarding the plot, it is important to realize that the hits considered are the ones from :doc:`penthesilea`. Events coming from :doc:`beersheba` comprise a much larger amount of *deconvoluted hits* (more than one order of magnitude), given the finer granularity.

 .. [#] T. H. Cormen, C. Stein, R. L. Rivest, and C. E. Leiserson, Introduction to algorithms. McGraw-Hill Higher Education, 2nd ed., 2001.

 .. [#] For those who don't know, these famous *blobs* are imaginary 3D spheres located around both ends of each track. Their energy (provided by the hits contained within the sphere) corresponds to an excellent tool to investigate whether there has been a large and sudden energy deposition in the track extreme (i.e. *Bragg peak*, indicating the stopping point of a charged particle) or not (starting point of its trajectory).

 .. [#] One could think that this effect will also happen in long intricate tracks, where both end points turn out to be close. Nevertheless, and as it has been explained above, the blob energy is only computed using the hits inside the blob sphere **and** belonging to the extreme voxel or its adjacent ones **along** the track. As a consequence, these scenarios are successfully avoided.
