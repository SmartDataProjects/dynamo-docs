Dealer
------

Before explaining the balancer, we should describe the Dealer architecture. Dealer consists of one central engine and one or more plugins. Plugins each have one purpose and produce a proposal of data transfers. For example, the aforementioned replication module is one plugin that proposes copies of popular replicas. Examples of other plugins are:

  * request handler, collecting information from the copy request registry table and proposing transfers as specified in the requests;
  * undertaker, proposing transfers of protected data out of sites in the morgue state; and
  * balancer, proposing replications of last-copy data from sites with high protected fraction.

The proposals are then collected by the central engine, which sorts them by the priorities assigned to each plugin (specified in the Dealer configuration file), and issues the transfer requests until the transfer volume hits the preset maximum. Unhandled proposals are dropped for this cycle. Requests for transfers are of course tracked and thus will come back in the next cycle of the regular sequence.

The balancer is, as mentioned above, a Dealer plugin that proposes replicating last-copy datasets that reside in sites with high protection fraction. Replication results in balancing when combined with Detox, as illustrated in the following example.

Suppose a dataset D originally has the last copy (and therefore protected) at site S1, whose quota is filled up with protected data. Dealer then creates a replica of dataset D at another site S2 where only 70% of data is protected. Then some time later, Detox runs. As soon as the copy of dataset D at site S2 is complete, the one at site S1 will be unprotected, and since in this case site S1 was filled up, Detox will delete the copy at site S1. The net effect is a transfer of protected data from site S1 to site S2, making the protected fraction at the two sites slightly closer. A freshly copied dataset gets a temporary boost in popularity to avoid the fresh copy to be deleted when site S2 runs full before site S1.
