User Data Concept
-----------------

The management of user data is notoriously difficult. This is probably the reason why there is no coherent implementation for user data management in CMS. In the following we layout a plan of how Dynamo will allow user data to be managed. For our design there are a few important basic assumptions:

 1. Each site dedicates a quota to user data which when added up between the sites satisfies the space required by user analysis [#]_.
 2. There has to be a reserved namespace that only Dynamo controls for each user at each site.

Following the fairly standard process of how user data comes about in CMS we will outline the data management concept. A user runs a processing task over a given official (or private) dataset to produce a root file as output. Another fairly standard task is generating events which again results in some large output files. In both cases the output file is what the user will want to further analyze and possibly hand over to managed storage.

When the user job finishes the user will inject the file into the Dynamo system at the site where the job is running. Practically this means a call to the REST API of Dynamo and a straight copy of the file into the user namespace at the site where the job is running. From there on Dynamo is in charge to organize any further transfers. In principle the user file could remain at the sites where the jobs were running but it is likely that some consolidation of the scattered files is desired. The user is free to make a subscription of the dataset to a single site which will be automatically executed by the dealer. Scattered files would be automatically cleaned up in the process.

Creation of blocks inside the dataset is left to Dynamo. We foresee blocks never to be scattered among sites, and we foresee to close blocks once they have reached a configurable maximum number of files. Dynamo will need to signaled that the processing is completed and will thus close the remaining blocks and complete the transfers.

It is also possible to think how to inject data that have been produced earlier and are available in some non-Dynamo managed location. It is up to the user to define dataset names and file names. They will be checked for uniqueness at injection time and the data will be entered into the registry (REST API) to be be copied to the reserved user namespace area. It is the users responsibility to delete the injected data once they are completed.

During the entire process the user can interact with the REST APIs to monitor the status of files and blocks produced in the analysis or being copied for the local injection case. It is clear that the user has to manage the files through Dynamo from now on; there is no other management possible.

For CMS, this would mean that ASO is not needed anymore, at least not for the output files that the user wants to inject into Dynamo.

.. rubric:: Footnotes
.. [#] To maintain the partition in a healthy state we expect the occupancy of the partition with unique data overall not to exceed 75% better to be even below 60%.
