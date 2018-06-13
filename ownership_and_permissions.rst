Ownership, Permissions and Quotas
---------------------------------

Dynamo is a Data Management system that is centrally organized, but has to be able to handle data at many different sites. To reduce the complication of data ownership and permissions to a minimum, all data that are managed by Dynamo are also owned by Dynamo. Each site has to make sure it gives Dynamo access to its site, no other users need to be managed. This is a very simple operation and all other complication can now be managed centrally at the server. It follows a more detailed explanation of the present implementation in CMS.


Authentication
..............

Dynamo relies on ssl authentication which is based on the x509 certificate provided by the user. Other authentication should be straight forward to adapt if so required as the system is centrally organized.


Authorization
.............

Any operation on the data in the system is reflected in an operation on the inventory. There are only a limited number of different operations on the inventory:

   * listing of data,
   * adding metadata for existing data,
   * subscriptions of existing data to a specific site,
   * removal of replica files and their metadata, and
   * removal of all metadata for a given entity (datasets, sites)

Each of the interactions with the inventory are called a target. Permissions to the targets is explicitly given as a (role, target) tuple. It is checked centrally whether a given user has a role that allows to perform the requested operation.


Quotas
......

Quotas are used to manage resources for a given entity. The resources in the data management system refer to storage space. We distinguish :ref:`partitionquota-ref` applied to the big partition of data available at each site and :ref:`userquota-ref` that applies to data that user generate in the course of performing their analysis and decide to inject into Dynamo.

.. _partitionquota-ref:
   
Partition Quotas
++++++++++++++++

Partition quotas are set per site for data partitions of the official production data and user data to protect each site individually from running full. While due to the somewhat random nature of data distribuion, the size of the sites, and the usage of the sites it is conceivable that a single site runs beyond its quota, it should never happen though that the entire system runs out of space. The result would be that data recording or new sample simulation would have to drop data on the floor or required policies for data would have to be violated. The resource availability has to be managed carefully by the experiment and policies need to be designed to have space available.

Partition quotas are therefore assigned as a few numbers per site. In CMS there are three partitions, the official data is further subdivided in data primarily used by analysis and the data used by the production system. These two groups (AnalysisOps and DataOps) each have a set quota share and are part of a super partition called Physics. Users have a separate partition with their own quota [#]_.

By design Dynamo keeps the partitions full by generating more copies of popular data the quotas add up to approximately the full amount of storage.

.. _userquota-ref:


User Quotas
+++++++++++

The main complication for permissions arises when quotas for users have to be met. Even if a user has permission to write data if the quota is filled no more data writing can be allowed. For production the quota is different because production can never run out because otherwise the experiment will have to stop taking data. Production has a large tape library as a backup and quota implementation can be handled differently.

For users it will happen more frequently and an active user might end up running out of quota not being able to transfer and inject more data into the system. User quota is not a number set per site but it is the amount of unique data the user is responsible for in the system. If a dataset is replicated at another site the additional copy will not count towards the users quota. On the other hand, when the overall user partition, which has a partition quota (set per site), runs out of space the second copy is a candidate for deletion.

.. rubric:: Footnotes
.. [#] At present CMS does not have a data management system for user data. Dynamo lines out a clear strategy how to implement the management of user data.
