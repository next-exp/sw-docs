Production Flow
====================

This page reviews the production flow for both data and simulations. Even if the reconstruction chain is the same for both samples, there are some discrepancies that need to be taken into account.
Additionally, a different chain will be follow depending on the type of simulation, and detector geometry. The cities mentioned along this list are summarised in :doc:`IC`.
It is also important to remark that during data acquisition, two different trigger configurations are considered:

 * *Trigger 1*: to select low energy events, i.e. Kr events.
 * *Trigger 2*: to select high energy events, e.g. Tl events, bb events, etc.

Simulations will be separated depending on which type of generator is used, as Trigger 1 or 2 like events.

.. warning::
  Software is in a constant evolution process, this is a current picture, but it might change in few months. Stay tuned!

For both samples, when representing the production flow, the type of same legend is used:

  .. image:: images/legend.png
    :width: 850


Data
------------
Data collected in NEXT detectors is in form of *Raw Waveforms* (**RWF**, in ADCs). The following production flow is used over the **RWFs** for both type of Triggers:

  .. image:: images/prodflow_data.png
    :width: 850

 * Sensor parameters for **Irene** need to be updated before any reconstruction. These are obtained from calibration using **Phyllis**, **Trude**, and **Beredice** IC cities. These parameters are updated regularly during detector operation, and are stored in a database. Updates to the database are pushed to the repository, so keeping up to date with the main branch is very much recommended.
 * Correction maps are obtained from Krypton events (Trigger 1) using :doc:`ICAROS`. Official production correction maps can be found in :doc:`production`. A complete review about how this maps are produced can be found in "How to :ref:`krmaps`".
 * PSFs are nedeed to run Richardson Lucy deconvolution (**Beersheba**). Krypton events (Trigger 1) are used under a specific configuration of **Penthesilea** and **Eutropia**. Official production PSFs for deconvolution can be found in :doc:`production`. A review about how this PSFs are produced can be found in "How to :ref:`psfdeco`".

 .. note::
   *dst* stands for *data summary tape*

Simulations
------------
Simulations do not produce directly **RWF**, for that reason is required to run additional cities in the production flow. The first type of data format that is constructed from NEXUS files is *True Waveforms* (**TWF**, in photoelectrons).
To construct them is necessary to take into account **which type of simulation** is used. NEXUS simulations can be produced including sensors information, like time and detected charge of the sensors (*full simulation*)
or just with the information from the true hits of deposit energy (*fast simulations*). Depending of this output, a different reconstruction chain will need to be implemented: **Buffy** or **Detsim**.

   .. image:: images/prodflow_nexus_TWF.png
     :width: 850

**Detsim** files require of Lighttables and PSFs. The ones created for official production can be found in :doc:`production`. Otherwise, a review about how they can be created can be found in "How to :ref:`lighttables`".

Once **TWFs** are created, the simulation production flow take a different way depending on **which type of detector** is simulated. This is caused by the fact that some of the geometries do not have electronic parameters included in their database.
It currently happens for NEXT-100 and NEXT-FLEX geometries. In that case, we produce **pmaps** directly using **Hypathia** (pseudo-**RWF** are created on the fly but not stored). These pseudo-rwf contain only gaussian electronic noise and gain fluctuations in the PMTs while SiPMs have the same processing. For detectors with the electronic parameters included (like DEMOPP or NEXT-White), we can transform **TWF** into **RWF** using **Diomira**.

  .. image:: images/prodflow_TWF_pmaps.png
    :width: 850

From this point on, the same production flow than data is used in simulations (see image from Data section).

.. note::
  For simulations, there is not distinction between Trigger 1 or 2 to transform NEXUS files into **pmaps**.
