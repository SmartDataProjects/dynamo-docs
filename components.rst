Components
----------

Data Organization
.................

The main data entity used in organization and communication is the dataset. A dataset consists of one or more files that share a common metadata. For example, a Monte Carlo simulation sample of Higgs decays to two photons generated with a specific set of configurations has a number of files which end up in one dataset. If the user is interested in accessing one of the files, the user is very likely also interested in accessing the rest of the files in the given dataset. It thus makes sense to group them together and perform data operations together on them.

As it turns out, some of those data samples can get very large, which can be problematic when managing them. Therefore an intermediate level of organization is introduced. Files are actually first organized into blocks, and datasets are made of one or more blocks.

Dynamo's atomic data unit is a file, which is uniquely identified with its Logical File Name (LFN). The logical file name will have to be translated to a Physical File Name (PFN) when trying to find the physical location of the file. Therefore each site defines its own namespace as a part of the site-specific configuration.


Authorization
.............

In Dynamo, there is ultimately only one owner of the data at all storage sites, which is the user under which the Dynamo server runs. User management is therefore about controlling who can access and modify the *metadata* the system manages - for example, a privileged user can instruct Dynamo to go delete a dataset at a site, while non-privileged users can only read what data is available where. Users are identified by x509 certificates, and are given explicit authorization by the administrator of the system. Aside from private data produced and owned by individual analysts, typically only the operation team personnel needs access to the system.

The current authorization scheme is very basic but there is a clear path for expansion if so desired.


Processes and Organization
..........................

To manage the data across a number of different storage sites Dynamo employs several core components that implement the conceptual design described above. Those components are:

 1. **Inventory Server** - The Inventory Server (also referred to as simply the Dynamo server) process running with the purpose to keep the entire inventory in memory and give extremely fast and controlled access to its content. It also ensures optimized synchronization with the disk based on mysql database. The synchronization process is generic, and additional fully synchronized inventory servers can be started in parallel to ease the load if needed. They can also be configured to automatically replace the main server in case of failures.
 2. **Registry / REST API** - The registry is a set of databases with various tables which is used as a communication hub between the world and the various components in Dynamo.
 3. **Scheduler** - The Scheduler is the process that controls the access to the inventory and schedules all requests to it. There are two different kinds of accesses: readonly and read/write. To protect the integrity of the inventory, only one read/write process is allowed at a time, while readonly processes can run in parallel.
 4. **Detox process** - The Detox process is responsible for deleting from sites the least interesting data once the sites go over the maximum watermark. The process stops when the low watermark has been reached or when no more data can be deleted without violating a set policy.
 5. **Dealer process** - The Dealer process is responsible for organizing the distribution of data. There are various reasons why data has to be distributed, with different priorities. They range from transfer requests for production input or new data, transfers required to fulfill policies, balancing the site filling status *etc.* There are limits per site and overall transfer limits per cycle *etc.*
 6. **File Operation**  - The File Operation Manager is responsible for initiating and organizing the actual transfer and deletion operation tasks. The tasks can then be pushed to the FTS system or to the standalone File Operation Daemon included in the Dynamo package.
 7. **Site Consistency** - The Site Consistency tools will list the complete contents of any given storage site and compare it with the inventory. Recoveries of missing files orphan files are requested to the File Operation Manager. The tools also know about the ongoing workflows in CMS and will clean out the production areas, that are not part of the registered data, from potentially leftover files and out-of-date log files.
