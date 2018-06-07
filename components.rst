Components
----------

Data Organization
.................

Dynamo's atomic data unit is a file, which is uniquely identified with its Logical File Name (LFN). The logical file name 


Authorization
.............

Processes and Organization
..........................

To manage the data across a number of different storage sites Dynamo employs several core component that implement the conceptual design described above. Those components are:

 1. **Inventory Server** - The Inventory Server process running with the purpose to keep the entire inventory in memory and give extremely fast and controlled access to its content. It also ensures optimized synchronization with the disk based mysql database. The synchronization process is generic and additional fully synchronized inventory servers can be started in parallel to ease the load if needed. They can also be configured to automatically replace the main server if it fails.
 2. **Registry / REST API** - The registry is a set of databases with various tables which is used as a communication hub between the world and dynamo and the various components in Dynamo.
 3. **Scheduler** - The Scheduler is the process that controls the access to the inventory and schedules all requests to it. There are two different kind of accesses: readonly and read/write. To protect the integrity only one read/write process is allowed at the time while readonly processes can run in parallel.
 4. **Detox process** - The Detox process is responsible to clean sites from the least interesting data once they go over the maximum watermark. The process stops when the low watermark has been reached or when no more data can be deleted without violating a set policy.
 5. **Dealer process** - The Dealer process is responsible to organize the distribution of data. There are various reasons why data has to be distributed which have different priorities. They range from transfer requests for production input or new data, transfers required to fulfill policies, balancing the site filling status *etc.* There are limits per site and overall transfer limits per cycle *etc.*
 6. **File Operation Daemon**  - The File Operation Daemon is responsible to push all data operation requests into the FTS system and manage their completions and deal with any potential issues.
 7. **Site Consistency** - The Site Consistency tools will list the complete contents of any given storage site and compare it with the inventory and enter data deletion and transfer requests into the registry to fix missing files and delete orphan files. The tools also know about the ongoing workflows in CMS and will clean out the production areas, that are not part of the registered data, from potentially leftover files and out of date logfiles.


