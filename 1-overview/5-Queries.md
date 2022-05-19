# Queries

> Snapshot Queries: Queries that see the latest snapshot of the table as of a given commit or compaction action. For MoR tables, snapshot queries expose the most recent state of the table by merging the base and delta files of the latest file slice at the time of the query. 

> Incremental Queries: Queries only see new data written to the table, since a given commit/compaction. This effectively provides change streams to enable incremental data pipelines.

> Read Optimized Queries: For MoR tables, queries see the latest data compacted. For CoW tables, queries see the latest data committed.

The following table shows the possible Hudi Query types for each storage type:

| Table Type      | Possible Hudi Query Tyoes |
| ----------- | ----------- |
| Copy On Write      | snapshot, incremental      |
| Merge On Read   | snapshot, incremental, read optimized       |

> [!Tip]
> Currently, Amazon Athena only supports snapshort quueries and read optimized queries