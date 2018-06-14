Data Popularity
---------------

There are two pieces of code that deal with popularity information in the CMS instance of Dynamo. Both are small plugins that fetch information from the respective external source, compute the popularity value for each dataset, and attach the value as a *named attribute* to the dataset objects. The two information sources are the CMS PopDB and the CMS global queue schedd.

 * PopDB: the last date of access by CRAB for each dataset is used to rank the datasets.
 * Global Queue: rank the datasets by last submission time of user analysis jobs (CRAB) running on them.

The named attributes are then used in the Detox deletion policy (for ordering the deletion within each site, unpopular datasets go first) or the data replication module within Dealer (make copies of popular data).

The design is such that the producers and consumers of the attributes are fully detached and are not aware of each other. For example, the Dealer replication module simply looks for a numeric attribute named "request_weight", but does not specify in any way how this number should be computed. There could therefore be an entirely different module, written for the specific installation of Dynamo and using the information source available to that installation, that computes "request_weight" by its own algorithm. It is the Dealer and Detox configuration files that determine what plugins are to be used to produce various attributes.
