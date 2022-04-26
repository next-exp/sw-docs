NEXUS
=====

NEXUS stands for *NEXt Utility for Simulation* and can be found on Github as `next-exp/nexus <https://github.com/next-exp/nexus>`_.

**Language**: C++, Geant4; **Based**: Geant4 toolkit, divided in logical blocks.

This section is focused on reviewing how the code is structured and where the installation guides can be found.

.. note::
  The main NEXUS documentation can be found on the `Wiki <https://github.com/next-exp/nexus/wiki>`_  of its Github repository.

.. _NEXUScode:

Code
------------

NEXUS code is `structured <https://github.com/next-exp/nexus/wiki>`_  in a set of common tools (basic geometries, physics lists, materials database, etc)
to ensure data consistency to be easily extended to include new geometries
and configurations. NEXUS is written in C++, and depends on the following third-party libraries:

 * **Geant4**: NEXT simulations are based on Geant4.
 * **ROOT**: used for I/O and histogramming.
 * **gsl**: NEXUS depends on the GNU Scientific Library for the simulation of double beta decay events.
 * **hdf5**: output files are written as hdf5.

Important information about NEXUS code:
 * `User-guide <https://github.com/next-exp/nexus/wiki/User-guide>`_ can be found on the NEXUS Wiki.
 * A summary of the `output format <https://github.com/next-exp/nexus/wiki/Output-format>`_ can also be found there.
 * Examples of output format are uploaded on doc-db in `NEXT-doc-1313-v1 <https://next.ific.uv.es/cgi-bin/DocDB/private/ShowDocument?docid=1313>`_ .

Tags
------------
A list of the different NEXUS `tags <https://github.com/next-exp/nexus/wiki/Tags>`_ can be found on the Wiki.
Starting from nexus tag v7_00_00, the following versions are needed:
 * Geant4 version >= 4.11.0.
 * ROOT >= 6.26.

 .. note::
   Geant4 version 10.7.x cannot be used with any nexus tag or even commit. For previous tags, Geant4.10.6 or Geant4.10.5 were used.

.. _NEXUSinstallation:

Installation
------------

A complete installation guide about NEXUS and all the third-party libraries can be found `here <https://github.com/next-exp/nexus/wiki/Installing-and-running-nexus>`_.
Additional details about `GEANT4 installation <https://github.com/next-exp/nexus/wiki/GEANT4-installation>`_ can also be found.

Contact
------------

If you are missing something, or you would like to contribute,
contact any of our **MC Managers**

 * `Paola Ferrario <paola.ferrario@gmail.com>`_
 * `Justo Martín-Albo <justo.martin-albo@ific.uv.es>`_

If you have any question, or you would like to discuss something related to NEXUS with other users or developers,
you can also write on the **Slack Channels**:

 * *#nexus_support*: Support channel for users to raise issues and questions.
 * *#next100sim*: Channel focused on next-100 simulation details: geometry issues, production, etc.
