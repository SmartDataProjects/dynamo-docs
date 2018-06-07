Typical Operations
------------------

Dynamo has been written to reduce the number of interactions of humans with the storage system to a minimum. The main task humans have to do is to write policies for the storage. Policies can be very simple but they can also offer a very flexible use of the storage accounting for many different requirements each setup implies. Making sure that the system has enough storage at its disposal is the other task that the operators have to perform.

The task of adding new data to the system (injection) and invalidating data that are already in the system are occasionally done by hand so operators need to know how to do this safely. Even more rarely general deletion campaigns are useful to remove data from the system that really are not useful anymore. In CMS this process happens maybe twice a year after for example a big reprocessing has finished and users have moved away from the original processing. Deletion campaigns have to be approved by the physics organization and Dynamo offers a rich set of tools to study deletions following requirements on many different metadata.


Policy Building Blocks
......................

Policies are statements based on operators, regular expression matching and the basic Dynamo building blocks. Building blocks are:
 1. Partition
 2. sites
 3. replica
 4. blockreplica
 5. dataset

Their detailed description is given in `here <https://github.com/SmartDataProjects/dynamo/blob/master/lib/policy/variables.py>`_.

Adding a Storage Site to the System
...................................




Setting Up a Basic Policy
.........................

It is important to note that the ordering in the policy file matters. Each sample is pushed through the policy stack and the moment a policy applies the sample is not processed further.

For the following example we have a Tier-3 center with limited disk space and a Tier-2 center where more data can be kept. The analysis data can be entirely maintained inside the Tier-2 quota but the Tier-3 can not hold more than maybe a third. In the below policy stack we simply maintain the most popular samples on the Tier-3 site as long as they fit within the available quota. For the setup here we can assume that the Tier-2 site has a quota large enough to contain all data, thus deletion will never be triggered, but in principle if the usage would be pushed to the high watermark on the Tier-2, deletion of the least popular one should be the safest option. More sophisticated schemes can of course be setup.

A basic policy always starts with setting up a Partition.

.. code-block:: c

   Partition MyCache

Define a number of storage sites this partition has access to.

.. code-block:: c
   
   On site.name in [ T2_US_XYZ T3_US_XYZ ]

Set the high and low water mark to define the deletions.

.. code-block:: c
   
   When site.occupancy > 0.9
   Until site.occupancy < 0.85

Set the default decision for potential deletion which is 'yes' dimiss.

.. code-block:: c
   
   Dismiss

Now decide what should be deleted first. The setup here uses the rank of the dataset and if they are the same it starts with the small datasets first. The rank is a number which is calculated to indicate how popular the dataset is. The CMS definition is approximately [#]_ the number of days the dataset was not used (we call that the idle days). So, the higher the rank the less popular the sample is.

.. code-block:: c
  
   Order decreasing dataset.usage_rank increasing replica.size
 
Managing Quotas
...............

Quotas per site are recorded in a database. A REST API gives access to the quotas set for each partition. Quotas can be changed at any time but depending on what the available storage is and what data is presently on the storage site problems can occur.

Usually increasing the quota is less of an issue, though empty space does attract transfers. Sometimes it is best to slowly increase the quota to limit the number of transfers, though there is an internal limit how much data is subscribed per cycle to a specific site and there is a limit on the pending transfer volume to a site. In general increasing the quota is straight forward.

Decreasing the quota can put sites in a situation where they are not able to clean out enough data in a single cycle to meet the requested quota. This is not a problem if the site still has enough disk space but it will cause a warning until the balancer has loaded off the essential data to other sites.

.. code-block:: c
  
   *setting the quota -- Yutaro here please*


Injecting New Data
..................

Data injection happens usually when the Monte Carlo production system or the Detector data processing system produce new data samples that should be made available to the users. Once files become available they are injected into Dynamo by using the standard REST API that we also use to populate the database during the installation. There are various options. It can be done file by file.

.. code-block:: c
  
   *setting the quota -- Yutaro here please*

or in larger chunks using the power of json formatted strings. The injecting system is responsible to define the metadata. Please check out the detailed interface `here please fix <https://github.com/SmartDataProjects/dynamo>`_.



Invalidating Data
.................

Data invalidation means data that was once valid will be turned into invalid data. While this seems obvious it is important to ponder on this for a moment. Data in Dynamo once invalidated can be deleted at any time and thus **the action of invalidation cannot be reverted**. The reason why we write *can* be deleted just refers to the fact that Dynamo might need some time to execute the deletions at all storage sites. The metadata though stays in the system for historical purposes.

Therefore it is essential to think very carefully before invalidating data. Usually data is invalidated when a major mistake was found in the production process and thus the data are useless or files were completely lost which means that there are no proper copy in the system anymore. The former happens more often while the latter happens rarely but with many million of files it does happen eventually. File invalidation will need a number of actions in the system in particular is the data are still available on disk. In general when Dynamo finds invalid data they will be deleted to save storage space.

To invalidate single files:
.. code-block:: c
  
   *invalidate a file -- Yutaro here please*

To invalidate blocks or entire datasets:
.. code-block:: c
  
   *invalidate a block or a dataset  -- Yutaro here please*

If so desired removing INVALID or DEPRECATED data can be switched off or tuned to remain for a grace period in the storage.


Planning Deletion Campaigns
...........................

While policies are very powerful sometimes it is more effective to explicitly remove data from the storage. The process of deletion from disk only is usually already rather tedious, but removing them altogether including a tape copy is painful and sometimes scary. The reson for this is that in bigger collaborations it is hard to track who really needs the data and sometimes unforeseen events might make certain data useful again. Planning data deletion is therefore very important and good tools are needed to coral the data that should be removed. In CMS the physics organization gets involved and it can take weeks to converge on an agreable list.

Dynamo provides an easy to use interface with fully exposed metadata to tests policies setup to identify data that can be deleted. The idea is to write a policy file, execute it and get in return the list of dataset that would be removed.

.. code-block:: c
  
   *run deletion campaign interface  -- Benedikt here please*

.. rubric:: Footnotes
.. [#] There are some corrections to the simple number of idle days to make sure that data that has just been copied it not deleted immediately and some adjustments for the size of the sample.
