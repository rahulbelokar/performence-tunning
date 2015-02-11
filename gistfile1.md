# SQL Server Performance Tuning Class - Day 3
## Denver 2015

-- SocietySkins (laptop skins)

### Performance Tuning Indexes
  - `sp_BlitzIndex`
  - If you don't create a clustered index then data is stored in a structure called a 'heap'
  - Usage stats shows you what was in the plan, Op Stats will tell you what was acutally used.
  - A 'scan' isn't necessarily a full scan. You need to look at the actual execution plan to see
  - Don't rebuild indexes with a page count < 2,000 since it won't make a difference
  - Index maintenance won't apply to heaps unless you actively force it
  - Goal is to create the least indexes to solve the most problems
  - The key of an index should be as specific as possible. If you have a very broad index field then that will require more scans
  - Look at Equality vs Inequality lists.
  - Indexes are ordered by the key columns, include column order doesn't matter at all
  - `(ONLINE = ON)` is only available in Enterprise Edition
  - Plan cache review is the best way to measure improvements by looking at CPU and logical read differences
  - Look at the plans of all top queries to see where more indexes can help
  - Trivial optimization never does an index request - there's only one way to execute
  - Look for the key lookup pattern and see if widening an existing index via includes could help
  - If you don't make a key unique, then SQL Server will make it unique for you with the UNIQUIFIER
  - `sp_BlitzCache` look for high average reads in the plan cache
  - If you see a column only in the Output columns and not listed as a predicate then it's a good candidate for adding as an INCLUDE to an index rather than being added as a key.
  - Good clustering key properties
    - Unique (doesn't have to be absolutely unique if you have a good natural key and can't change)
    - Narrow
    - Static (don't want rows to change this value and then get moved around a bunch)
    - Supports queries
    - Auto incrementing
  - Look for high average reads and average CPU. Also look for scans with predicates
  - If you have a scan that is going seek backwards then this will inhibit parallelism
  - A constraint (as in uniqueness) will always create an index behind the scenes

### Signs You Need to Rethink Schema (partitioning)
#### Bulk Loads
  - Filtered indexes to separate out workloads
  - Frequent maintenance
  - Loading / deleting data in chunks
  - Switching recovery modes (bad idea)
  - Filtered index is just an index with a WHERE clause
  - Filtered indexes won't be used if you use a parameter!
  - Partitioning is generally only done in data warehouses. Doesn't help in OLTP systems.
  - Load & Archival
    - Load data into an unindexed heap
    - Clean it up, add the right indexes
    - Swap the scratch space into the active table
    - SQL Server 2014 introduced WAIT_AT_LOW_PRIORITY to avoid blocking chains
      - Regular queries can continuing working while you're waiting to swap out
    - Partitioning is a data management function and not a performance function. Helps with concurrency
  - You can perform maintenance on individual partitions
  - 2014 introduced incremental statistics tracking - can then just update at the partition level

#### Concurrent Data Access
  - SQL Server 2008+ can use multiple threads per partition
  - SQL server will push the partition ID as a secret key at the front of your indexes. Microsoft calls this a 'skip scan'

#### Partitioning Benefits
  - Swapping
  - Partition level maintenance
  - Partition level lock escalation
  - Partition aware seeks

#### Column Store
  - This was designed to be used in conjunction with partitioning
  - NOT for OLTP. This is tailored for Data Warehouse installations only

### Indexed Views
  - Lots of limitations. Can be useful if you have lots more reads than writes
  - With standard edition, you must force the hint as the optimizer will not use it
    - `OPTION(NOEXPAND)`, `OPTION(EXPANDVIEWS)`

### Filtered Indexes
  - If you have known criteria then you can see some dramatic improvements with query performance
  - You're moving the work to the write rather than the read.
  - Doesn't allow for parameters so you'll have to use dynamic sql to pass literals in the sql string
  - Helps with low cardinality indexes
  - DO create a filtered index per queried value (e.g. status codes). DON'T create a filtered index per value.
  - Optimizer considers more cases to apply filtered indexes than indexed views
  -

## Diagnosing Problems with In-Memory Tables (Hekaton)
  - Lots of limitations. Very very early access at this point.
  - "RPO" - Recovery Point Objective
  - "RTO" - Recovery Time Objective

## How to Write a Prescription
  - Start with symptoms and perceived root cause
  - Choose a metric
  - Hours of wait time per hour is a good general 'how busy am I' metric for a server. Most monitoring packages will track this out of the box.
  - Blocked queries per hour via Perfmon. Extended events would be a more detailed mechanism for identifying blocking chains.
  - LCK* wait time per hour
  - Percentage waits and measures aren't really helpful as they can be misleading
  -
