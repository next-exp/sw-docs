Repositories
===============
NEXT software can be found on Github as `next-exp <https://github.com/next-exp/>`_. The main code is structured in two main repositories:

* :doc:`NEXUS`: Simulation tool in charge of the detector geometries and particle generators
* :doc:`IC`: Reconstruction tool for detector data and simulations

Other reconstruction tools can be found there (as :doc:`ICAROS`, :doc:`olivia`) that work as a support to the main software code. `CERES <https://github.com/next-exp/CERES>`_ is also a repository under next-exp used for IC official production scripts and config files during NEXT-White data acquisition. 
Any NEXT repository can be forked into your own Github account. :doc:`workflow` explains in detail the way users and developers can use of these repositories.

.. warning::
  Some of these external reconstruction tools (:doc:`ICAROS`, :doc:`olivia`, etc) will be included in the main reconstruction code (:doc:`IC`) in the next future. Stay tuned!

All the relevant information about these repositories and links to the installation instructions is gathered in the following links:

.. toctree::
   :maxdepth: 2

   NEXUS
   IC
   ICAROS
   olivia
