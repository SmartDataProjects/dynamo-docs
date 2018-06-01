Dynamo - Dynamic Data Management System
=======================================

The development of Dynamo started in 2014 by the CMS Computing Operations organization and was initially foreseen to be a tool working on top of the existing CMS data transfer engine PhEDEx. During the last year Dynamo evolved its capabilities to take over the organization of the data transfers and provide a complete package to manage data using, DBS as the source for the initial definition of the metadata, FTS to perform the specific data transfers and the CMS popularity service to track the usage of the data.

As the package evolved from the CMS environment it was initially coupled to the CMS environment but has in the last half year been decoupled and a standalone version ready to be used by other experiments has been produced allowing external specialized plugins for services like the popularity or a master metadata source describing the data. We are looking forward to supporting other efforts where large amount of data has to be managed accross a potentially heterogeneous set of storage sites. We do support a fully integrated tape storage.


Introduction
------------

The moment we accumulate data of a large volume the question of how to do Data Management arises. Even with this problem being a very old and well-studied one no single solution has emerged. The reason is that Data Management has to be appropriate to address the specific set of requirements and work in the given environment. Those factors have a strong influence on the design of such tools.

The LHC experiments, and in particular the CMS experiment, has a tiered computing approach in which in the order of 100 sites provide storage for the data. These sites are rather heterogenoeus in terms of size, local mass storage technology, level of support *etc*. Tier-1 centers (7 for CMS) provide tape systems in which most of the data is permanently stored but not immediatly accessible. Some of the data are accessed very frequently other less frequently and the majority of data in the end will be accessed only in exceptional situations. The available disk storage is not sufficient to keep all data readily available, thus important decisions have to be taken which data to keep on disk and in how many copies and where.

Data usage in the experiment can be subdivided into two big classes: production access and analysis/user access. The biggest difference between the two is that production access is predictable while user analysis is inherently unpredictable. As an example the re-processing of the 2016 data is carefully planned and the necessary inputs can be staged from tape without causing any delay to the reprocessing schedule. User analysis would immediately suffer if data is not available on disk and in sufficient copies to avoid bottlenecks.

The initial approach in CMS towards data management was to ensure that datasets[#f1]_ can be efficiently and safely transfered from one storage site to the other implementing a rich set of permissions to identify who is allowed to do what. Sites were put in charge to install local software agents to execute transfers and communicate with the central agents about their progress.

The intelligence about which data was supposed to be available and at what sites was supposed to be implemented by the data managers that were appointed by the specific physics or detector groups who claimed ownership of the specific data. Each group had specific 3-5 Tier-2 sites at their disposal to be filled with the datasets of interest. It required some coordination to decide who was in charge of the large detector data because most of the data samples for example SingleMuons are used by all physics groups. The Monte Carlo simulation data though was a completely different level of coordination which quickly caused a lot of headaches because there were much more samples and the ownership issue was similarly complex.

For the first few years this concept worked because there was ample space, a lot of interest and support from the sites and the data managers, and relatively few datasets. Over time, sites and data managers had less resources and with the rapidly growing amount of data and number of datasets a real problem developed. There was a large need for automation which was particularly evident in the Computing Operations Organization at the time.

Dynamo was developed with the goal to eliminate human interactions with the Data Management system and at the same time optimize the way the storage is used to hold the data for the user analysis and for the data production system (Detector and Monte Carlo simulation).


Conceptual Design
-----------------

Essential Components
--------------------

Plugins
-------

.. rubric:: Footnotes

.. [#f1] Data in CMS or mainly organized in datasets which ultimately contain a bunch of files.
