Official Production
====================

This page collects all the information related to the official **simulation** production of NEXT detectors.

.. warning::
  This webpage is still under construction. If you would like to contribute, reach `me <helena.almamol@gmail.com>`_!


NEXT-White
------------
Latest official production was generated using

.. list-table::
   :widths: 40 60
   :header-rows: 0

   * - **nexus**
     - ``v5_07_00``/``v5_07_01``
   * - **detsim**
     - ``v0_09_01``
   * - **IC: diomira, irene, and dorothea**
     - ``v1.1.0``
   * - **IC: penthesilea**
     - ``v1.2.0``
   * - **IC: isaura**
     - ``v1.2.alberto``
   * - **Background Model**
     - ``v9``

It can be found in ``neutrinos1.ific.uv.es`` under the following path:

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/

For the different productions the corresponding latest **tags** are:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Production
     - Folder
   * - **0nubb**
     - ``NEXT_v1_05_02_NEXUS_v5_07_00_bkg_v9``
   * - **Xe2nu**
     - ``NEXT_v1_05_02_NEXUS_v5_07_01_bkg_v9``
   * - **Calibration**
     - ``NEXT_v1_05_02_NEXUS_v5_07_01_bkg_v9``
   * - **NEW Background Model**
     - ``NEXT_v1_05_02_NEXUS_v5_07_01_bkg_v9``


**Output hdf5** files together with the corresponding config files used to run each of the cities can be found there. E.g. the background production of deconvoluted hits (``cdst``) can be found in

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/Background/NEXT_v1_05_02_NEXUS_v5_07_01_bkg_v9/cdst/output

Where the following **config** files have been used,

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/Background/NEXT_v1_05_02_NEXUS_v5_07_01_bkg_v9/cdst/conf


.. note::
  It is important to stress some aspects regarding the production:
    * The production is based on the v9 version of the background model. The details of the model can be found `here <https://next.ific.uv.es/cgi-bin/DocDB/private/ShowDocument?docid=182>`_.
    * The two versions of ``nexus`` (v5_07_00 and v5_07_01) are identical but for an update concerning the calibration ports.
    * The detsim package refers to the ``c++`` and ``art`` version. The repository may be found `here <https://next.ific.uv.es:8888/nextsw/detsim>`_.
    * The EventMixer package refers to the ``gate`` version. The repository may be found `here <https://next.ific.uv.es:8888/nextsw/PyToNE/blob/master/PyToNE/EventMixer.py>`_.
    * The IC package is considered in different versions since the goal is to match the official Canfranc production for data (``v1.1.0``). However, several updates implied the usage of more recent versions, even a custom one before the officialization of ``ìsaura``.

NEXT-100
------------
A first production using fast simulations (:doc:`detsim`) has been produced using

.. list-table::
   :widths: 40 60
   :header-rows: 0

   * - **nexus: radiogenics + signal**
     - ``v7_02_00``
   * - **nexus: muons**
     - ``v7_03_00``
   * - **nexus: LTs and PSFs for Detsim**
     - ``v7_01_00(+ G4 op bug)``
   * - **IC**
     - ``8e1de90c1cec5a1c3d25b35eca456f87f4f7e64c``
   * - **IC: muons**
     - `PR#823 <https://github.com/next-exp/IC/pull/823>`_
   * - **NEXT-100 Background Model**
     - ``v4``


This production can be found in ``neutrinos1.ific.uv.es`` under the following path:

.. code-block:: text

 /data4/NEXT/NEXT100_prod/

**Nexus macros** for the current production can be found on GitHub `here <https://github.com/gondiaz/NEXT100-0nubb-analysis/tree/main/nexus_job_templates/ft3>`_.

Detsim *light tables* (**LTs**) and *point spread functions* (**PSFs**) can be found in ``neutrinos1.ific.uv.es`` in

.. code-block:: text

  Note: these files disappeared due to an incident in the neutrinos cluster.
  This is just for reference.
  /data4/NEXT/NEXT100_prod/LightTables/ (for productions before May 2024)
  /lustre/neu/data4/NEXT/NEXT100/MC/Tables/202405_Krishan (for productions after May 2024)

**Config** files for the rest of the production chain can also be found on `here <https://github.com/gondiaz/NEXT100-0nubb-analysis/tree/main/ic_processing/templates>`_ on Github.
