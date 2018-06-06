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

A basic policy always starts with setting up a Partition.
::
   Partition MyCache

Define a number of storage sites this partition has access to.
::
 On site.name in [ T2_US_MIT T3_US_MIT ]

Set the high and low watermakr to define the deletions.
::
 When site occupancy > 0.9
 Until site.occupancy < 0.85


 
 
Managing Quotas
...............


Injecting New Data
..................


Invalidating Data
.................


Planning Deletion Campaigns
...........................

