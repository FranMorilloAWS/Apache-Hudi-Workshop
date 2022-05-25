# Querying Apache Hudi

In the last lab we have already started doing some queries in order to check if were writing the data and doing the upserts and deletes. We will now see the types of queries we can execute using Apache Spark

We will be using the same notebook, so we do not have to configure the Glue Interactive Session again.

## 1. Snapshot Query

Replace the Bucket Name
```
snapshotQueryDF = spark.read \
    .format('org.apache.hudi') \
    .load('s3://EXAMPLE-BUCKET/hudi/hudi-tickets/)
    
snapshotQueryDF.show()
```

This will return the latest version of the Data after it has been written to S3.

## 2. Incremental Queries

You can also perform incremental queries with Hudi to get a stream of records that have changed since a given commit timestamp. 

Specify the instant Time you want to retrieve all rows that have been updated from.

readOptions = {
  'hoodie.datasource.query.type': 'incremental',
  'hoodie.datasource.read.begin.instanttime': <beginInstantTime>,
}

```
incQueryDF = spark.read \
    .format('org.apache.hudi') \
    .options(**readOptions) \
    .load('s3://EXAMPLE-BUCKET/hudi/hudi-tickets/)

incQueryDF.show()
```

## 3. Time Travel Queries

We can specify a moment in time and retrieve how our table looked in that specific moment

```
timeTravelQueryDF = spark.read \
  .format("hudi"). \
  .option("as.of.instant", "2022-05-10"). \
  .load('s3://EXAMPLE-BUCKET/hudi/hudi-tickets/)
```

