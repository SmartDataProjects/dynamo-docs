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


Setting Up a Basic Policy
.........................

It is important to note that the ordering in the policy file matters. Each sample is pushed through the policy stack and the moment a policy applies the sample is not processed further.

For the following example we have a Tier-3 center with limited disk space and a Tier-2 center where more data can be kept. The analysis data can be entirely maintained inside the Tier-2 quota but the Tier-3 can not hold more than maybe a third. In the below policy stack we simply maintain the most popular samples on the Tier-3 site as long as they fit within the available quota. For the setup here we can assume that the Tier-2 site has a quota large enough to contain all data, thus deletion will never be triggered, but in principle if the usage would be pushed to the high wtaermark on the Tier-2, deletion of the least popular one should be the safest option. More sophisticated schemes can of course be setup.

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

Decreasing the quota can put sites in a situation where they are not able to cleanout enough data in a single cycle to meet the requested quota. This is not a problem if the site still has enough disk space but it will cause a warning until the balancer has loaded off the essential data to other sites.

.. code-block:: c
  
   *setting the quota -- Yutaro here please*


Injecting New Data
..................

Data injection happens ususally when the Monte Carlo production system or the Detector data processing system produce new data samples that should be made avaiable to the users. Once files become available they are injected into Dynamo by using the standard REST API that we also use to populate the database during the installation. There are various options. It can be done file by file.

.. code-block:: c
  
   *setting the quota -- Yutaro here please*

or in larger chunks using the power of json formatted strings. The injecting system is responsible to define the metadata. Please check out the detailed interface `here please fix <https://github.com/SmartDataProjects/dynamo>`_.



Invalidating Data
.................


Planning Deletion Campaigns
...........................

.. rubric:: Footnotes
.. [#] There are some corrections to the simple number of idle days to make sure that data that has just been copied it not deleted immediately and some adjustments for the size of the sample.
