Introduction
------------

The moment we accumulate data of a large volume the question of how to do data management arises. Even with this problem being a very old and well-studied one, no single solution has emerged. The reason is that data management has to be appropriate to address the specific set of requirements and work in the given environment. Those factors have a strong influence on the design of such tools.

The LHC experiments, and in particular the CMS experiment, has a tiered computing approach in which in the order of 100 sites provide storage for the data. These sites are rather heterogeneous in terms of size, local mass storage technology, level of support *etc*. Tier-1 centers (7 for CMS) provide tape systems in which most of the data is permanently stored but not immediately accessible. Some of the data are accessed very frequently, other less frequently, and the majority of data in the end will be accessed only in exceptional situations. The available disk storage is not sufficient to keep all data readily available, thus important decisions have to be taken on which data to keep on disk, in how many copies, and where.

Data usage in the experiment can be subdivided into two big classes: production access and analysis/user access. The biggest difference between the two is that production access is predictable while user analysis is inherently unpredictable. As an example, the re-processing of the 2016 data is carefully planned and the necessary inputs can be staged from tape without causing any delay to the reprocessing schedule. User analysis would immediately suffer if data is not available on disk and in sufficient copies to avoid bottlenecks.

The initial approach in CMS towards data management was to ensure that datasets [#]_ can be efficiently and safely transferred from one storage site to another, with a rich set of permissions to identify who is allowed to do what. Sites were put in charge to install local software agents to execute transfers and communicate with the central agents about their progress.

The intelligence about which data was supposed to be available and at what sites was supposed to be provided by the data managers that were appointed by the specific physics and detector groups who claimed ownership of the specific data. Each group was assigned 3-5 specific Tier-2 sites to be filled with the datasets of their interests. It required some coordination to decide who was in charge of the large detector data because most of the data samples, for example *SingleMuon*, are used by all physics groups. The Monte Carlo simulation data though required a completely different level of coordination which quickly caused a lot of headaches because there were much more samples and the ownership issues were equally complex.

For the first few years this concept worked, because there was enough disk space, a lot of interest and support from the sites and the data managers, and relatively few datasets. Over time, sites and data managers had less resources, and with the rapidly growing amount of data and number of datasets a real problem developed. Another important development was that the strict rule of re-processing the detector and Monte Carlo simulation data only at the Tier-1 sites placed a major bottleneck on the production process. Moving the re-processing also to the Tier-2 sites meant a substantial increase in data transfers which became impossible to support with the Computing Operations team.
In short, there was a large need for automation which was particularly evident in the Computing Operations organization at the time.

Dynamo was developed with the goal to eliminate or at least minimize human interactions with the Data Management system and at the same time optimize the way the storage is used to hold the data for the user analysis and for the data production system (Detector and Monte Carlo simulation). There were a number of key conclusions reached.

 1. Users should not care where their jobs run as long as they finish successfully and quickly
 2. Physics and detector groups did not want and could not manage their own data
 3. Sites did not want to manage the exact data content of their storage
 4. Data production system desperately needed an automatic way to spread the data around on all production sites with the least amount of effort

Becoming aware of those important points made CMS rethink their data management model. A number of important simplifications were introduced and new features added.

 1. Sites were opened to any datasets that users or production were interested in
 2. The majority of owner groups were deprecated while maintaining two main groups: Analysis and Production
 3. Policies were introduced to fill disk space automatically with popular data replicas, while removing less popular data replicas
 4. A fully automatized site consistency enforcement was introduced to address any failures in the data management system
 5. A fully automatic site evacuation was introduced to quickly and efficiently deal with major site failures
 6. An interface to the batch submission system to allow for automatic tape staging for data required by the users but not available on disk was provided


.. rubric:: Footnotes
.. [#] Data in CMS or mainly organized in datasets which ultimately contain a bunch of files.
