Dynamo - Dynamic Data Management System
=======================================

The development of Dynamo started in 2014 by the Computing Operations organization and was initially foreseen to be a tool working on top of the existing CMS data transfer engine PhEDEx. During the last year Dynamo evolved its capabilities to take over the organization of the data transfers and provide a complete package to manage data using, DBS as the source for the initial definition of the metadata, FTS to perform the specific data transfers and the CMS popularity service to track the usage of the data.

As the package evolved from the CMS environment it was initially coupled to the CMS environment but has in the last half year been decoupled and a standalone version ready to be used by other experiments has been produced allowing external specialized plugins for services like the popularity or a master meta data describing the data. We are looking forward to supporting other efforts where large amount of data has to be managed accross a potentially heterogeneous set of storage sites. We do support a fully integrated tape storage.


Introduction
------------

The moment we accumulate data of a larger volume the question of how to do Data Management arises. Even with this problem being a very old and well-studied one no single solution has emerged. The reason is that Data Management has to be appropriate to address the specific set of requirements.

The LHC experiments, and in particular the CMS experiment, has a tiered computing approach in which in the order of 100 sites provide storage for the data. These sites are rather heterogenoeusly organized in terms of size and local mass storage technology. Tier-1 centers (7 for CMS) provide tape systems in which most of the data is permanantly stored.


Conceptual Design
-----------------

Essential Components
--------------------

Plugins
-------
