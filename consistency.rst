Consistency
-----------

Data and metadata consistency is fundamental for a data management system to work properly. Consistency has various aspects. They reach from the consistency between data catalog and the physical file presence at any given site in prestine conditions and the consistency of the Dynamo metadata with the metadata in systems external to Dynamo [#]_.

The most common form of inconsistency observed in the past in CMS is the comparison of what is called the inventory in Dynamo and the physical file presence at the given storage sites. The inconsistencies can be subdivided into data that inventory carries but are not in the foreseen location (missing data) and data that are physically at the site but are not entered in the inventory (orphan or dark data).

Missing files lead to failures of those jobs that try to access them, while orphan files fill up the site storage without being properly accounted for in the site occupancy.

As much as one wants to ensure such inconsistencies do not occur, with the complexity of the system and realistic site operations it is unrealistic to think a system can be designed to avoid them altogether. To mitigate the problem Dynamo provides a package that constantly checks and enforces consistencies by copying missing data or deleting data that is not supposed to be there.

The package called dynamo-consistency is explained in detail in its `documentation pages <https://github.com/SmartDataProjects/dynamo-consistency/blob/master/README.rst>`.

.. rubric:: Footnotes
.. [#] In CMS DBS is the ultimate source for the composition of any dataset.
