IC cities
==========

:doc:`IC` is the NEXT reconstruction software that is structured in a set of programs named *cities*.

City Structure
----------------
IC cities follow dataflow data processing: they are designed to resemble a pipeline. By reading this pipeline one should be able to understand what operations are made and in which order.

.. image:: images/pipeline.png
  :width: 650

More details about how this data format works can be found in a previous `IC-crash-course <https://github.com/mmkekic/IC-crash-course/blob/master/presentations/Dataflow.pdf>`_.
Resembling this pipeline structure, each city needs a specific type of *input* data format, and creates a specific *output* to follow the rest of the production flow:

.. image:: images/citystructure.png
  :width: 350

How to run a city
-------------------
To run a city (once IC environment is set) you just need to run:

.. code-block:: text

  city city_name config_file_city.conf

where ``config_file_city.conf`` corresponds to the specific configuration file for that city.


Configuration files
--------------------
Cities may require some parametrisation and, as it is represented on the previous figure, some of them require additional auxiliar (*aux*) data (like maps, PSFs, etc).
Configuration files (*config*, ``config_file_city.conf``) provide to the city this specific information they require to run. Examples for each city are located in `IC/invisible_cities/config <https://github.com/next-exp/IC/tree/master/invisible_cities/config>`_. Official production config files (and production
scripts) are located in `next-exp/CERES <https://github.com/next-exp/CERES>`_ repository.

.. image:: images/configfile.png
  :width: 850


List of Cities
------------------
IC cities can be categorised depending on their purpose on the following list:

MAIN PRODUCTION:
 * Irene
 * Penthesilea
 * Dorothea
 * Esmeralda
 * Beersheba
 * Isaura
 * Eutropia

CALIBRATION:
 * Phyllis
 * Trude
 * Beredice

ONLY FOR MC:
 * Detsim
 * Buffy
 * Diomira
 * Hypathia

DEBUGGIN/CONTROL:
 * Isidora

Each of this cities include a small description in the IC repository (`IC/invisible_cities/cities <https://github.com/next-exp/IC/tree/master/invisible_cities/cities>`_) and under :doc:`prodflow` a complete review of the IC cities chain can be found for both data and simulations.
