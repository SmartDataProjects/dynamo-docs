Installation
------------

Presently our installation relies on the person installing having root access and performing a git clone operation and than a number of steps to setup the database, performing some configuration and then importing the data and validating the full setup.

Prerequisites
.............

The main server machine needs to have an installation of python (>2.7, <3.0), mysql, ....

Depending on the number of data objects in the storage the machine needs to be configured with sufficient memory. For the CMS setup we are running a single machine with 96 GB of memory. The official data take less than 10% of the available memory for the full inventory to be loaded.

There has to be a x509 certificate that is allowed to interact with Dynamo. Different users with different permissions can be configured.


Basic Installation
..................


Configuration
.............


Initial Data Import
...................


Validate Full Setup
...................
