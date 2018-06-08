Typical Operations
------------------

Dynamo has been written to reduce the number of interactions of humans with the storage system to a minimum. The main task for humans is to write policies for the storage. Policies can be very simple but they can also offer a very flexible use of the storage, accounting for many different requirements each setup implies. Making sure that the system has enough storage at its disposal is the other task that the operators have to perform.

The task of adding new data to the system (injection) and invalidating data that are already in the system are occasionally done by hand, so operators need to know how to do this safely. Even more rarely general deletion campaigns are useful to remove data from the system that really are not useful anymore. In CMS, this process happens maybe twice a year after for example a big reprocessing has finished and users have moved away from the original processing. Deletion campaigns have to be approved by the physics organization and Dynamo offers a rich set of tools to study deletions following requirements on many different metadata.


Policy Building Blocks
......................

Policies are statements based on operators, regular expression matching and the basic Dynamo building blocks. Building blocks are:
 1. Partition
 2. sites
 3. replica
 4. blockreplica
 5. dataset

Their detailed description is given `here <https://github.com/SmartDataProjects/dynamo/blob/master/lib/policy/variables.py>`_.


Adding a Storage Site to the System
...................................

In a distributed system like the one in CMS or ATLAS there are systems which keep track of the sites and their capabilities. In CMS there is a tool called siteDB [#]_ which can be used to pick up the various properties of a site, but in principle others exist. In Dynamo, sites are represented as objects in the inventory. Sites can be added at runtime using tools like `dynamo-inject`. An example is given in the :ref:`addstorage-ref` section.

It is straightforward to write a tool to extract this information from an external database as to keep this information up-to-date.


Setting Up a Basic Detox Policy
...............................

For the following example we have a Tier-3 center with limited disk space and a Tier-2 center where more data can be kept. The analysis data can be entirely maintained inside the Tier-2 quota but the Tier-3 can not hold more than maybe a third. In the below policy stack we simply maintain the most popular samples on the Tier-3 site as long as they fit within the available quota. For the setup here we can assume that the Tier-2 site has a quota large enough to contain all data, thus deletion will never be triggered, but in principle if the usage would be pushed to the high watermark on the Tier-2, deletion of the least popular one should be the safest option. More sophisticated schemes can of course be setup.

First, the partition in which we operate has to be defined. Partition definitions are given in a file pointed to by the server configuration (`/etc/dynamo/server_config.json`) parameter `partition_def_path`, which is `/usr/local/dynamo/etc/default_partitions.txt` by default. In the file, there is already a partition named Default defined:

.. code-block:: c

   Default:  dataset.name == *

which says everything (because * matches all dataset names) is included in the default partition. Each line of the partition definition file defines a partition, with a syntax `<Name>: <condition>`. The conditions are applied to *block replicas*, and all matching replicas are included in the partition. For this example we set up a partition named `MyCache`, which consists of all block replicas at the Tier-2 and Tier-3 sites:

.. code-block:: c

   MyCache:  site.name in [ T2_US_XYZ T3_US_XYZ ]

We then proceed to building the Detox policy. Open a new file `MyCache.txt`. The first line of the file is the partition declaration:

.. code-block:: c

   Partition MyCache

Define a number of storage sites this partition has access to. Because we defined our partition to be everything on these sites, this line is redundant, but is here for illustration purposes.

.. code-block:: c
   
   On site.name in [ T2_US_XYZ T3_US_XYZ ]

Set the deletion start and stop triggers (high and low water marks in this case).

.. code-block:: c
   
   When site.occupancy > 0.9
   Until site.occupancy < 0.85

Note that the above three lines refer to *site attributes* (`site_variables` in the `variables.py <https://github.com/SmartDataProjects/dynamo/blob/master/lib/policy/variables.py>`_), whereas the rest of the policy file is written in terms of *replica attributes* (`replica_variables`).

The lines succeeding the trigger definitions are called the *policy stack* and is in general the main part of the policy file. Each line starts with either `Protect`, `Delete`, or `Dismiss` (action keywords) [#]_, followed by a condition that is evaluated against dataset replicas. Each dataset replica in the partition is pushed through the policy stack from the top. The action of the first line with a matching condition is applied to the replica. (It is therefore important order the policy lines carefully.) If the action is `Protect`, the replica is not deleted. With `Delete`, it is unconditionally deleted. Replicas matching a `Dismiss` line will be candidates for deletion, but are only deleted when deletion is triggered at the site.

In this example, we will define a one-line policy stack to protect replicas that have just been transferred (inferred by the creation date of the last block replica):

.. code-block:: c

   Protect replica.last_block_created newer_than 1 day ago

The last line of the policy stack sets the default action for all dataset replicas with no matching lines. We want the replicas to be deletable if necessary:

.. code-block:: c
   
   Dismiss

Now decide what should be deleted first. The setup here uses the rank of the dataset. If two datasets have identical ranking, the smaller dataset is deleted first. The rank is a number which is calculated to indicate how popular the dataset is. The CMS definition is approximately [#]_ the number of days the dataset was not used (we call that the idle days). So, the higher the rank the less popular the sample is.

.. code-block:: c
  
   Order decreasing dataset.usage_rank increasing replica.size

Once the policy file is written, you can execute the application Detox to actually perform the deletions.
::

  dynamo '/usr/local/dynamo/exec/detox --config /etc/dynamo/detox_config.json --policy /full/path/to/MyCache.txt' --write-request --title detox

Note that `detox` must be authorized as a read/write executable beforehand (see `Application Authorization`_).

 
Managing Quotas
...............

Quotas are defined per site per partition and can be changed at any time. The quota Dynamo uses may be completely disconnected from the reality; it is simply a number Dynamo is told that the site has for a given partition.

Usually increasing the quota is less of an issue, though empty space does attract transfers. Sometimes it is best to slowly increase the quota to limit the number of transfers, though there is an internal limit on how much data is subscribed per cycle to a specific site and there is a limit on the pending transfer volume to a site.

Decreasing the quota can put sites in a situation where they are not able to clean out enough data in a single Detox cycle to meet the requested quota. This is not a problem if the site still has enough disk space, but it will cause a warning until the balancer has loaded off the essential data to other sites.

To manage the quota, use the `set_quotas.py` script in the `utilities` directory. Volume is measured in terabytes.
::
  
  dynamo '/usr/local/dynamo/utilities/set_quota.py --site T2_US_XYZ --dump'
  dynamo '/usr/local/dynamo/utilities/set_quota.py --site T2_US_XYZ --volume 100' --write-request --title set_quota # set_quota must be authorized first


Injecting New Data
..................

Data injection happens usually when the Monte Carlo production system or the Detector data processing system produce new data samples that should be made available to the users. Once files become available they are injected into Dynamo by using `dynamo-inject` that we also use to populate the inventory during the installation. The injecting system is responsible for defining the metadata.


Invalidating Data
.................

Data invalidation (deletion of metadata in Dynamo inventory) means data that was once valid will be turned into invalid data. While this seems obvious it is important to ponder on this for a moment. Invalidated data become orphan files and can be deleted at any time by the Site Consistency tool. Therefore, **the action of invalidation cannot be reverted**, and it is essential to think very carefully before invalidating data. Usually, data is invalidated when a major mistake was found in the production process and thus the data are useless, or when files are completely lost, which means that there are no proper copy in the system anymore. The former happens more frequently than the latter, but with many million of files, data loss does happen eventually.

The tool for data invalidation is also `dynamo-inject`, but with a `--delete` option. The format for the JSON file for invalidation is similar to the one in the `Initial Data Injection`_ section. The only difference is that the items only need their names. As an example, to invalidate a file `/store/user/me/lost_file.root` which belongs to the block `abcd` of the dataset `/A/B/C`, write a JSON file with content
::

  {"dataset":
    [
      {"name": "/A/B/C",
       "blocks":
        [
          {"name": "abcd",
           "files":
            [
              {"name": "/store/user/me/lost_file.root"}
            ]
          }
        ]
      }
    ]
  }

and then execute (as a user with `admin` role)
::

  dynamo-delete <json file>


Planning Deletion Campaigns
...........................

While policies are very powerful, sometimes it is more effective to explicitly remove data from the storage. The process of deletion from disk only is usually already rather tedious, but removing them altogether including tape copies is painful and sometimes scary. The reson for this is that in bigger collaborations it is hard to track who really needs the data and sometimes unforeseen events might make certain data useful again. Planning data deletion is therefore very important and good tools are needed to coral the data that should be removed. In CMS, the physics organization gets involved and it can take weeks to converge on an agreeable list.

The Detox application has a *test run* option, where test policy files can be evaluated without altering the inventory state or issuing any actual deletions.
::
  
  dynamo '/usr/local/dynamo/exec/detox_cms --test-run --config /etc/dynamo/detox_config.json --policy <test policy file>' --write-request --title detox


.. rubric:: Footnotes
.. [#] On the longer run siteDB will be replaced by CRIC.
.. [#] Actually there are a few more actions that can be taken. See the `Detox policy <https://github.com/SmartDataProjects/dynamo/blob/master/lib/detox/detoxpolicy.py>`_ module for details.
.. [#] There are some corrections to the simple number of idle days to make sure that data that has just been copied it not deleted immediately and some adjustments for the size of the sample.
       
