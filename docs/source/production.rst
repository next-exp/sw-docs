Official Production
====================

This page collects all the information related to the official simulation production of NEXT detectors.

.. warning::
  This webpage is still under construction. If you would like to contribute, reach `me <helena.almamol@gmail.com>`_!


NEXT-White
------------
Latest official production was generated using

.. list-table::
   :widths: 40 60
   :header-rows: 0

   * - **nexus**
     - ``to be added``
   * - **detsim**
     - ``to be added``
   * - **IC**
     - ``to be added``
   * - **Background Model**
     - ``to be added``

It can be found in ``neutrinos1.ific.uv.es`` under the following path:

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/

For the different productions the corresponding latest **tags** are:

.. list-table::
   :widths: 40 60
   :header-rows: 1

   * - Production
     - Tag
   * - **0nubb**
     - ``NEXT_v1_05_02_NEXUS_v5_07_00_bkg_v9``
   * - **Xe2nu**
     - ``NEXT_v1_05_02_NEXUS_v5_07_10_bkg_v9``
   * - **Calibration**
     - ``NEXT_v1_05_02_NEXUS_v5_07_10_bkg_v9``
   * - **Background**
     - ``NEXT_v1_05_02_NEXUS_v5_07_10_bkg_v9``


**Output hdf5** files together with the corresponding config files used to run each of the cities can be found there. E.g. the background production of deconvoluted hits (``cdst``) can be found in

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/Background/NEXT_v1_05_02_NEXUS_v5_07_10_bkg_v9/cdst/output

Where the following **config** files have been used,

.. code-block:: text

  /lustre/neu/data4/NEXT/NEXTNEW/MC/Background/NEXT_v1_05_02_NEXUS_v5_07_10_bkg_v9/cdst/conf



NEXT-100
------------
This sample is currently being produced. It is generated using

.. list-table::
   :widths: 40 60
   :header-rows: 0

   * - **nexus**
     - ``to be added``
   * - **detsim**
     - ``to be added``
   * - **IC**
     - ``to be added``
   * - **Background Model**
     - ``to be added``

**Nexus macros** for the current production can be found on GitHub `here <https://github.com/gondiaz/NEXT100-0nubb-analysis/tree/main/nexus_job_templates/ft3>`_.

Detsim *light tables* (**LTs**) and *point spread functions* (**PSFs**) can be found in ``neutrinos1.ific.uv.es`` in

.. code-block:: text

  /data5/users/gdiaz/NEXT100/kr83m/LightTables

**Config** files for the rest of the production chain can also be found on `here <https://github.com/gondiaz/NEXT100-0nubb-analysis/tree/main/ic_processing/templates>`_ on Github.
