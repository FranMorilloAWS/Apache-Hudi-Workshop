# Storage Types

When you create a Hudi dataset, you specify that the dataset is either copy on write or merge on read.

> Copy on Write (CoW): Data is stored in a columnar format (Parquet), and each update creates a new version of files during a write. CoW is the default storage type.

> Merge on Read (MoR): Data is stored using a combination of columnar (Parquet) and row-based (Avro) formats. Updates are logged to row-based delta files and are compacted as needed to create new versions of the columnar files. 

## 1. Copy on Write

By default, writes in Apache Hudi use table type Copy on Write.

With CoW datasets, each time there is an update to a record, the file that contains the record is rewritten with the updated values.

If you want to enable it manually you need to specify in the Hudi Configurations:

> "hoodie.datasource.write.table.type":"COPY_ON_WRITE"


## 2. Merge on Read

If you want to have better write performance for heavy write applications, for example a streaming application. You should use Merge On Read Table Type

With a MoR dataset, each time there is an update, Hudi writes only the row for the changed record. MoR is better suited for write- or change-heavy workloads with fewer reads

To create  Merge on Read Table (MoR) you can use the following configuration

> "hoodie.datasource.write.table.type":"MERGE_ON_READ"

We will now create a MoR table to see how we can interact with this table type.

1. Continue to use the same notebook
2. Modify the Path. We need to specify a different path, because this will be a different table stored in Amazon S3. 

```
morPath = 's3://hudi-workshop-s3hudibucket-4o8b53fyh2sx/hudi/hudi-tickets-mor/'
```

3. Modify the Configurations

```
hudiMorConfig = {'className' : 'org.apache.hudi', 
                'hoodie.datasource.hive_sync.use_jdbc':'false',  
                'hoodie.datasource.write.recordkey.field': 'id', 
                'hoodie.datasource.write.table.type':'MERGE_ON_READ',
                'hoodie.datasource.write.precombine.field': 'timestamp', 
                'hoodie.table.name': 'hudi_tickets_mor', 
                'hoodie.datasource.write.operation': 'bulk_insert'}
```

```
glueMorConfig  = {'hoodie.datasource.hive_sync.database': 'sports_tickets_hudi', 
                'hoodie.datasource.hive_sync.table': 'hudi_tickets_mor', 
                'hoodie.datasource.hive_sync.enable': 'true'}
```

```
unpartitionDataConfig = {'hoodie.datasource.hive_sync.partition_extractor_class': 'org.apache.hudi.hive.NonPartitionedExtractor', 
                         'hoodie.datasource.write.keygenerator.class': 'org.apache.hudi.keygen.NonpartitionedKeyGenerator'}
```

```
combinedMorConf = {**hudiMorConfig, **unpartitionDataConfig, **glueMorConfig}
```

4. We can now write to Amazon S3
```
initialDF.write.format("org.apache.hudi").options(**combinedMorConf).mode("overwrite").save(morPath)
```

6. Lets do some updates to the MoR Table

```
hudiUpdatesMorConfig = {'className' : 'org.apache.hudi', 
                'hoodie.datasource.hive_sync.use_jdbc':'false',  
                'hoodie.datasource.write.recordkey.field': 'id', 
                'hoodie.datasource.write.precombine.field': 'timestamp', 
                'hoodie.table.name': 'hudi_tickets_mor', 
                'hoodie.datasource.write.table.type':'MERGE_ON_READ',
                'hoodie.datasource.write.operation': 'upsert'}
```

```
combinedUpdateMorConf = {**hudiUpdatesMorConfig, **unpartitionDataConfig, **glueMorConfig}
```

```
updatesDF.write.format("org.apache.hudi").options(**combinedUpdateMorConf).mode("append").save(morPath)
```

7. Query Merge on Read Tables

 Hudi creates two tables in the metastore for MoR: a table for snapshot queries, and a table for read optimized queries.

* hudi_tickets_mor_ro
* hudi_tickets_mor_rt


> Read Optimized Queries (ro): queries the latest data compacted. This provides good performance but does not include the latest delta commits

> Snapshot Queries (rt): Snapshot queries contain the freshest data but incur some computational overhead, which makes these queries less performant. 

Using Spark.SQL we can query both tables

```
spark.sql("select * from hudi_tickets_mor_ro").show()
```

```
spark.sql("select * from hudi_tickets_mor_rt").show()
```

> [!WARNING]
> Using Glue Marketplace connector for Hudi we cant do Snapshot Queries. However we can do the query using Amazon Athena

If we compare the Snapshot Query took a little bit longer to process, because it is compacting the latest compacted data with the Delta Commits. 

This means that the RT table provides the freshed data but with the tradeoff of processing a compaction when querying.

> [!TIP]
> You can also query Hudi Tables using Amazon Athena. For Copy on Writes you can perform Snapshot Queries. For MoR, you can perform Snapshot and Read Optimized Queries. As of now, we can not do incremental or time travel queries from Athena