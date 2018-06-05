.. contents:: a title for the contents
    :depth: 2

Dynamo - Dynamic Data Management System
=======================================

The development of Dynamo started in 2014 by the CMS Computing Operations organization and was initially foreseen to be a tool working on top of the existing CMS data transfer engine PhEDEx. During the last year Dynamo evolved its capabilities to take over the organization of the data transfers and provide a complete package to manage data using, DBS as the source for the initial definition of the metadata, FTS to perform the specific data transfers and the CMS popularity service to track the usage of the data.

As the package evolved from the CMS environment it was initially coupled to the CMS environment but has in the last half year been decoupled and a standalone version, ready to be used by other experiments, has been produced allowing external specialized plugins for services like the popularity or a master metadata source describing the data. We are looking forward to supporting other efforts where large amount of data has to be managed across a potentially heterogeneous set of storage sites. We do support a fully integrated tape storage.


Introduction
------------

The moment we accumulate data of a large volume the question of how to do Data Management arises. Even with this problem being a very old and well-studied one no single solution has emerged. The reason is that Data Management has to be appropriate to address the specific set of requirements and work in the given environment. Those factors have a strong influence on the design of such tools.

The LHC experiments, and in particular the CMS experiment, has a tiered computing approach in which in the order of 100 sites provide storage for the data. These sites are rather heterogeneous in terms of size, local mass storage technology, level of support *etc*. Tier-1 centers (7 for CMS) provide tape systems in which most of the data is permanently stored but not immediately accessible. Some of the data are accessed very frequently other less frequently and the majority of data in the end will be accessed only in exceptional situations. The available disk storage is not sufficient to keep all data readily available, thus important decisions have to be taken which data to keep on disk and in how many copies and where.

Data usage in the experiment can be subdivided into two big classes: production access and analysis/user access. The biggest difference between the two is that production access is predictable while user analysis is inherently unpredictable. As an example the re-processing of the 2016 data is carefully planned and the necessary inputs can be staged from tape without causing any delay to the reprocessing schedule. User analysis would immediately suffer if data is not available on disk and in sufficient copies to avoid bottlenecks.

The initial approach in CMS towards data management was to ensure that datasets [#]_ can be efficiently and safely transferred from one storage site to the other implementing a rich set of permissions to identify who is allowed to do what. Sites were put in charge to install local software agents to execute transfers and communicate with the central agents about their progress.

The intelligence about which data was supposed to be available and at what sites was supposed to be implemented by the data managers that were appointed by the specific physics and detector groups who claimed ownership of the specific data. Each group was assigned 3-5 specific Tier-2 sites to be filled with the datasets of their interests. It required some coordination to decide who was in charge of the large detector data because most of the data samples for example *SingleMuons* are used by all physics groups. The Monte Carlo simulation data though required a completely different level of coordination which quickly caused a lot of headaches because there were much more samples and the ownership issues were equally complex.

For the first few years this concept worked, because there was enough disk space, a lot of interest and support from the sites and the data managers, and relatively few datasets. Over time, sites and data managers had less resources and with the rapidly growing amount of data and number of datasets a real problem developed. Another important development was that the strict rule of re-processing the detector and Monte Carlo simulation data only at the Tier-1 sites placed a major bottleneck on the production process. Moving the re-processing also to the Tier-2 sites meant a substantial increase in data transfers which became impossible to support with the Computing Operations team.
In short, there was a large need for automation which was particularly evident in the Computing Operations organization at the time.

Dynamo was developed with the goal to eliminate or at least minimize human interactions with the Data Management system and at the same time optimize the way the storage is used to hold the data for the user analysis and for the data production system (Detector and Monte Carlo simulation). There were a number of key conclusions reached.

 1. Users should not care where their jobs run as long as they finish successfully and quickly
 2. Physics and detector groups did not want and could not manage their own data
 3. Sites did not want to manage the exact data content of their storage
 4. Production desperately needed an automatic way to spread the data around on all production sites with the least amount of effort

Becoming aware of those important points made CMS rethink their data management model. A number of important simplifications were introduced and new features added.

 1. Sites were opened to any datasets that users or production were interested in
 2. The majority of owner groups were deprecated while maintaining two main groups: Analysis and Production
 3. Policies were introduced to fill disk space automatically with popular data replicas, while removing less popular data replicas
 4. A fully automatized site consistency enforcement was introduced to address any failures in the data management system
 5. A fully automatic site evacuation was introduced to quickly and efficiently deal with major site failures
 6. An interface to the batch submission system to allow for automatic tape staging for data required by the users but not available on disk was provided
    
Conceptual Design
-----------------

To resolve the above described problem we designed Dynamo based on a few fundamental ideas.

 1. Disks that are not filled with data are for sure never used and thus wasted, so we keep site storage above a minimum watermark [#]_
 2. Storage usage should be proportional to data popularity following some metric
 3. Sites have to operate safely and space should not be filled significantly beyond a high watermark threshold
 4. Minimize the need for interaction with the storage sites involved, and run it centrally
 5. Disk space availability varies significantly with time: adjustments of the data management policies to deal with these variations have to be straight forward and transparent including the use of tape storage
 6. Partitioning of disk space should be avoided as much as possible as it reduces the flexibility of the storage usage and effectively reduces the available disk space
 7. In a heterogeneous distributed storage, inconsistencies of many different types are unavoidable and the system has to be able to automatically identify and recover them
 8. The site reliability must be accounted for when distributing data
 9. Make the key components pluggable to allow for evolution of the external packages
 10. Fast turnaround is essential to minimize race conditions

With these ideas in mind the overall storage in CMS is maintained in one big partition which is called 'Physics'. Storage sites are added to the partition with a quota for how much space they provide. Each site is kept below a high water mark threshold. If incoming transfers push the site storage above the high watermark deletion is triggered. The deletion will be removing the least popular samples until a low watermark is reached, while at the same time fulfilling all other set policies. The system is almost [#]_ freely running, which means subscriptions to sites are made even if they push the site over the high water mark.  This mechanism insures that unpopular data (replicas or last copies) will be regularly purged, while at the same time fulfilling all policies.

If the experiment runs out of storage the system will eventually fail to delete and the sites will run full. In CMS it is the responsibility of the data management team to alert the physics organization and call for a review of the policies to make adjustments so that data fit on the available storage. As we will discuss in the following, policies are very flexible and powerful and easy to adjust at runtime to meet the requirements.

Dynamo has a `policy language <https://github.com/SmartDataProjects/dynamo/tree/master/lib/policy>`_ which allows to write separate policy files for the given partition and the groups belonging to that partition. The `policy files <https://github.com/SmartDataProjects/dynamo-policies>`_ are parsed by the various components of the system and data actions (deletions and subscriptions) are issued such that the policies are met once the actions are completed.

Dynamo maintains a number of smaller very specialized partitions with separate sites and quotas configurations. There are the Release Validation partition and the Express data partition which have very specific requirements which are implemented in Dynamo's policy language.

The interaction of the outside world with Dynamo to alter the content of the inventory (and the storage) is organized through REST APIs. Adding new datasets (injecting), removing datasets, changing their properties or simply looking at the contents are the main interactions. The majority of interactions with Dynamo come from the production system when new data is produced and possibly input data have to be distributed to optimize the production process and when the production output is injected into the system. All other data operations are in the design foreseen to be initiated and executed by Dynamo. While users can request specific data subscriptions to specific sites they are not needed. Dynamo monitors the CMS Global Queue for user jobs and will make their input data available potentially with several replicas.

Consistency between the inventory and the physical storage can break with two main outcomes. Either files that the inventory claims to be at a given site are not really there (missing files) or the storage keeps files that the inventory does not know about (orphan files). Missing files will cause job failures while orphan files eat up space and can cause the storage to fill up in an uncontrolled fashion. Both features are undesirable and Dynamo implements a continuous process, the Site Consistency daemon, that runs periodically at all site and identifies those problems and communicates with the Registry to make transfer or deletion requests to fix these issues.

Dynamo's development environment is very open. Developers have controlled and very fast access to the life inventory which allows a very rapid development without setting up a complex test instance. *Yutaro you need to flesh this out a little bit*.

Essential Components
--------------------

To manage the data across a number of different storage sites Dynamo employs several core component that implement the conceptual design described above. Those components are:

 1. **Inventory Server** - The Inventory Server process running with the purpose to keep the entire inventory in memory and give extremely fast and controlled access to its content. It also ensures optimized synchronization with the disk based mysql database. The synchronization process is generic and additional fully synchronized inventory servers can be started in parallel to ease the load if needed. They can also be configured to automatically replace the main server if it fails.
 2. **Registry / REST API** - The registry is a set of databases with various tables which is used as a communication hub between the world and dynamo and the various components in Dynamo.
 3. **Scheduler** - The Scheduler is the process that controls the access to the inventory and schedules all requests to it. There are two different kind of accesses: readonly and read/write. To protect the integrity only one read/write process is allowed at the time while readonly processes can run in parallel.
 4. **Detox process** - The Detox process is responsible to clean sites from the least interesting data once they go over the maximum watermark. The process stops when the low watermark has been reached or when no more data can be deleted without violating a set policy.
 5. **Dealer process** - The Dealer process is responsible to organize the distribution of data. There are various reasons why data has to be distributed which have different priorities. They range from transfer requests for production input or new data, transfers required to fulfill policies, balancing the site filling status *etc.* There are limits per site and overall transfer limits per cycle *etc.*
 6. **File Operation Daemon**  - The File Operation Daemon is responsible to push all data operation requests into the FTS system and manage their completions and deal with any potential issues.
 7. **Site Consistency** - The Site Consistency tools will list the complete contents of any given storage site and compare it with the inventory and enter data deletion and transfer requests into the registry to fix missing files and delete orphan files. The tools also know about the ongoing workflows in CMS and will clean out the production areas, that are not part of the registered data, from potentially leftover files and out of date logfiles.


External dependencies
---------------------

Dynamo interacts with a number of systems in CMS which are not part of Dynamo. Those systems are listed in the following and their basic functionality are described. For any non CMS experiment it should be straight forward to have other tools fill in the required information.

 1. **DBS** - The ultimate description of any dataset in CMS. It provides access to the full metadata.
 2. **Popularity DB** - Popularity DB records access of any datasets in terms of the number of files, the CPU hours and the ....
 3. **Global Queue** - The Global Queue is used to run user and production jobs which describe which datasets are being used or are scheduled to being used.
 4. **McM** - The Monte Carlo Management system is used to plan any Monte Carlo production request. It will track it through the system, and thus allows access to the presently active input and output data.

Before PhEDEx is replaced Dynamo also interacts with PhEDEx but there is no long term relationship planned.

.. rubric:: Footnotes

.. [#] Data in CMS or mainly organized in datasets which ultimately contain a bunch of files.
.. [#] In CMS we are using a low watermark of 85% while the high watermark is set to 90%.
.. [#] Subscriptions though will not go beyond the quota considering the projected size of all data at a given storage site.
