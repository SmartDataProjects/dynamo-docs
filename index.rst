.. Dynamo documentation master file, created by CP on Tue June 5 10:40:30 2018.

Dynamo - Dynamic Data Management System
=======================================

.. this is just hear to make sure sphinx does not warn about this file not being used
.. toctree::
    :hidden:

    README


History Preface
---------------

The development of Dynamo started in 2014 by the CMS Computing Operations organization and was initially foreseen to be a tool working on top of the existing CMS data transfer engine PhEDEx. During the last year Dynamo evolved its capabilities to take over the organization of the data transfers and provide a complete package to manage data using, DBS as the source for the initial definition of the metadata, FTS to perform the specific data transfers and the CMS popularity service to track the usage of the data.

As the package evolved from the CMS environment it was initially coupled to the CMS environment but has in the last half year been decoupled and a standalone version, ready to be used by other experiments, has been produced allowing external specialized plugins for services like the popularity or a master metadata source describing the data. We are looking forward to supporting other efforts where large amount of data has to be managed across a potentially heterogeneous set of storage sites. We do support a fully integrated tape storage.
	    
.. toctree::
    :maxdepth: 3

    introduction
    conceptual_design
    components
    external_packages
    installation
    typical_operations
    

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
