IC
=====

IC stands for *Invisible Cities* and can be found on Github as `next-exp/IC <https://github.com/next-exp/IC>`_.

**Language**: Python; **Based**: Numpy, Pandas

This section is focused on reviewing how the code is structured and where the installation guides can be found.

.. _ICcode:

Code
------------

IC is a reconstruction software structured in algorithms/programs named *cities*. The code is written Python, but it uses `Numpy <https://numpy.org/>`_ and `Pandas <https://pandas.pydata.org/>`_ for data analysis and as a manipulation tool.
Each of the *cities* is focused on different reconstruction tools of the code (e.g. raw waveforms integration, energy correction, deconvolution, etc...), and follow *dataflow* data processing: they resemble a pipeline of data transformations. More details about the dataformat can be found in a previous IC-crash-course in `Dataflow <https://github.com/mmkekic/IC-crash-course/blob/master/presentations/Dataflow.pdf>`_ section.

Important information about IC code:
 * A complete review about the IC repository can be found on this previous IC-crash-course under `IC-structure <https://github.com/mmkekic/IC-crash-course/blob/master/presentations/IC_structure.pdf>`_.
 * Each of the cities in the repository (`IC/invisible_cities/cities <https://github.com/next-exp/IC/tree/master/invisible_cities/cities>`_) includes a small description the city

 .. image:: images/cityfunctionality.png
   :width: 800

 * The structure of :doc:`ICcities` is described in this documentation, together with some guidelines about how to run them and a complete list of the current ones.
 * Under :doc:`prodflow` a complete review of the IC cities chain can be found for both data and simulations.

.. _ICinstallation:

Installation
------------

There is a *Quickstart guide* at the next-exp/IC repository with anything related to its installation. When compiling it, `conda <https://docs.conda.io/projects/conda/en/latest/index.html>`_ will be installed and it will be created the appropriate IC environment, as well as it will be set the environment variables which are needed for the correct functioning of IC.

Contact
------------

If you are missing something, or you would like to contribute,
contact any of our **Software Manager**: `Gonzalo Martínez-Lema <gonzaponte@gmail.com>`_

If you have any question, or you would like to discuss something related to NEXUS with other users or developers,
you can also write on the **Slack Channels**:
 * *#IC_support*: Support channel for users to raise issues and questions
