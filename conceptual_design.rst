Conceptual Design
-----------------

To resolve the situation described in the introduction we designed Dynamo based on a few fundamental ideas.

 1. Disks that are not filled with data are for sure never used and thus wasted, so we keep site storage above a minimum watermark [#]_
 2. Storage usage should be proportional to data popularity following some metric
 3. Sites have to operate safely and space should not be filled significantly beyond a high watermark threshold
 4. Minimize the need for interaction with the storage sites involved, and run it centrally
 5. Disk space availability varies significantly with time: adjustments of the data management policies to deal with these variations have to be straight forward and transparent including the use of tape storage
 6. Partitioning of disk space should be avoided as much as possible as it reduces the flexibility of the storage usage and effectively reduces the available disk space
 7. In a heterogeneous distributed storage, inconsistencies of many different types are unavoidable and the system has to be able to automatically identify and recover from them
 8. The site reliability must be accounted for when distributing data
 9. Encapsulate the key components, with exposed interfaces to allow for usage and evolution of external packages
 10. Fast turnaround is essential to minimize race conditions

With these ideas in mind the overall storage in CMS is maintained in one big partition which is called 'Physics'. Storage sites are added to the partition with a quota for how much space they provide. Each site is kept below a high watermark threshold. If incoming transfers push the site storage above the high watermark, deletion is triggered. The deletion will be removing the least popular samples until a low watermark is reached, while at the same time fulfilling all other set policies. The system is almost [#]_ freely running, which means subscriptions to sites are made even if they push the site over the high watermark. This mechanism insures that unpopular data (replicas or last copies) will be regularly purged, while at the same time fulfilling all policies.

If the experiment runs out of storage, the system will eventually fail to delete and the sites will run full. In CMS, it is the responsibility of the data management team to alert the physics organization and call for a review of the policies to make adjustments so that data fit on the available storage. As we will discuss in the following, policies are very flexible and powerful and easy to adjust at runtime to meet the requirements.

Dynamo has a `policy language <https://github.com/SmartDataProjects/dynamo/tree/master/lib/policy>`_ which allows to write separate policy files for the given partition and the groups belonging to that partition. The `policy files <https://github.com/SmartDataProjects/dynamo-policies>`_ are parsed by the various components of the system and data actions (deletions and subscriptions) are issued such that the policy requirements are met once the actions are completed.

Dynamo maintains a number of smaller very specialized partitions with separate sites and quotas configurations. There are the Release Validation partition and the Express data partition which have very specific requirements which are implemented in Dynamo's policy language.

The interaction of the outside world with Dynamo to alter the content of the inventory (and the storage) is organized through REST APIs. Adding new datasets (injecting), removing datasets, changing their properties or simply looking at the contents are the main interactions. The majority of interactions with Dynamo happens with the production system when new data is produced. Input data have to be distributed to optimize the production process, and when the production is complete, the output is injected into the system. All other data operations are in the design foreseen to be initiated and executed by Dynamo itself. While users can request specific data subscriptions to specific sites, that is not typically needed. Dynamo monitors the CMS Global Queue for user jobs and will make their input data available potentially with several replicas.

Consistency between the inventory and the physical storage can break with two main outcomes. Either files that the inventory claims to be at a given site are not really there (missing files) or the storage keeps files that the inventory does not know about (orphan files). Missing files will cause job failures while orphan files eat up space and can cause the storage to fill up in an uncontrolled fashion. Dynamo runs a continuous process, the Site Consistency daemon, that periodically monitors all sites and identifies those problems. When missing and orphan files are detected transfer and deletion requests are made to the main Dynamo system and executed.

Dynamo is not implemented as a database application. When the system is online, the entire state of the distributed storage, described in terms of objects like datasets and sites, is loaded into the RAM of the server machine. This feature enables execution of complex algorithms (anything programmable in python) over the various objects, which in turn allows the policies to be highly sophisticated. Dynamo applications are run as subprocesses of the server process, with full instant access to the image of the storage system without any possibility of altering what the server holds.

.. rubric:: Footnotes
.. [#] In CMS we are using a low watermark of 85% while the high watermark is set to 90%.
.. [#] Subscriptions though will not go beyond the quota considering the projected size of all data at a given storage site.
