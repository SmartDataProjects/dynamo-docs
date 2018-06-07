External Packages
-----------------

Dynamo interacts with a number of systems in CMS which are not part of Dynamo. Those systems are listed in the following and their basic functionality are described. For any non CMS experiment it should be straight forward to have other tools fill in the required information.

 1. **DBS** - The ultimate description of any dataset in CMS. It provides access to the full metadata.
 2. **Popularity DB** - Popularity DB records access of any datasets in terms of the number of files, the CPU hours and the ....
 3. **Global Queue** - The Global Queue is used to run user and production jobs which describe which datasets are being used or are scheduled to being used.
 4. **RequestManager** - The Request Management system is used to keep track of any ongoing any Monte Carlo production request. It will track it through the system, and thus allows access to the presently active input and output data.

Before PhEDEx is replaced in CMS, Dynamo also interacts with PhEDEx, but there is no long term relationship planned and no other experiment is recommended to use it.
