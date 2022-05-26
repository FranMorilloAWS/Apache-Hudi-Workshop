# Apache Hudi on AWS Glue

Previously, we have created a Hudi Connector for AWS Glue. We will now use a new feature from AWS Glue called Glue Notebooks. Glue Notebooks, deploys a Jupyter Notebook in which we can run Spark Code interactively. 

## 1. Start AWS Glue Notebook

1. Go to the [AWS Glue Console](https://console.aws.amazon.com/glue/)
2. On the left Menu, click on **Jobs**
3. Select from the options **Jupyter Notebook** and click **Create**
![Hudi-1](img/dhudi-fundamentals-1.png)
4. Use *Hudi Fundamentals* for **Job Name**
5. For **IAM Role**, select the *<stack-name>-glue-workshop-role*. Role will start with the name of the CloudFormation stack
6. Wait a few minutes to the Notebook to be available.

## 2. Configure Hudi Connector for Glue Notebook

Before you execute any cell in the notebook, we need to modify the configuration, so we can use the Hudi Connectors

### a. Using Hudi Connector from Jars

Add a new cell below the Available Magics clicking on the plus sign in the Jupyter Notebook Menu

You need to run this cell first:

```
# Execute this cell to configure and start your interactive session.
%session_id_prefix my-session-w5cct
%idle_timeout 120
%glue_version 3.0
# Replace <hudiworkshop-s3bucket> with the S3 bucket name where the Hudi jars are.
%extra_jars s3://hudi-workshop-s3bucket-y70ndp9i7scz/hudi-jars/spark-avro_2.12-3.1.1.jar, s3://hudi-workshop-s3bucket-y70ndp9i7scz/hudi-jars/httpclient-4.5.9.jar, s3://hudi-workshop-s3bucket-y70ndp9i7scz/hudi-jars/httpcore-4.4.11.jar ,s3://hudi-workshop-s3bucket-y70ndp9i7scz/hudi-jars/calcite-core-1.16.0.jar
%connections hudi-connection
%%configure 
{
    "--conf": "spark.serializer=org.apache.spark.serializer.KryoSerializer --conf spark.sql.hive.convertMetastoreParquet=false", 
    "--enable-glue-datacatalog" : "true"
}
```

You will need to replace in the extra_jars for all 4 jars the s3 bucket name. It will be the same you used to create the custom connector.

Execute the cell to configure the interactive session.

Now we can start executing the rest of the cells

### b. Using Glue Marketplace connector for Hudi

You will need to add an additional cell before the Import Libraries Cell. 

Add the follow code to the cell:
```
#Execute this cell to configure and start your interactive session.
%idle_timeout 120
%glue_version 3.0
%connections hudi-marketplace-connector
%%configure 
{"--conf": "spark.serializer=org.apache.spark.serializer.KryoSerializer --conf spark.sql.hive.convertMetastoreParquet=false","--enable-glue-datacatalog": "true"}
```
You can now execute the next cell

## 3. Create Hudi Table

1. Import the necessary libraries

```
import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.job import Job
  
sc = SparkContext.getOrCreate()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
```

2. Load the Sporting Event Ticket table from Amazon S3 using Spark SQL

> [!WARNING]
> Using Glue 3.0 we need to stop the Spark Context and resume in order to be able to use Spark SQL with the AWS Glue Catalog.

```
sc.stop()
sc = SparkContext.getOrCreate()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
```

```
spark.sql("use sports_tickets")
df = spark.sql("select * from sporting_event_ticket")
df.show(5)
```
3. Lets verify how many sport ticket ids we have
```
df.count()
```

From Exploring the data we know that this is not the actual number of tickets we should have in our Data Lake. Lets use Hudi to have ACID operations in Amazon S3

4. Gather the initial state of the Database

We will be filtering the Op column to only show the null values. This would be the rows that were first migrated into Amazon S3
```
initialDF = df.filter(df.op.isNull())
```

5. We are going to specify the path to where we are going to be writing the Hudi Table.

Remember to replace the Bucket Name with the second bucket the CloudFormation created. It will have *s3hudibucket* in the name
```
path = 's3://hudi-workshop-s3hudibucket-4o8b53fyh2sx/hudi/hudi-tickets/'
```

6. Lets configure the Hudi Options

```
hudiConfig = {'className' : 'org.apache.hudi', 
                'hoodie.datasource.hive_sync.use_jdbc':'false',  
                'hoodie.datasource.write.recordkey.field': 'id', 
                'hoodie.datasource.write.precombine.field': 'timestamp', 
                'hoodie.table.name': 'hudi_tickets', 
                'hoodie.datasource.write.operation': 'bulk_insert'}
```


Here we are specifying the partition key which we want our records to be identified by. For this table we want the *ID* to be unique.
> 'hoodie.datasource.write.recordkey.field': 'id'

In order to determine which would be the latest version of the ID, we will pick the column timestamp in order for Hudi to only consider the latest timestamp when doing the upserts into the table
> hoodie.datasource.write.precombine.field': 'timestamp'

We need to also specify the type of operation we want to perform when writing the table. Since we are only considering the initial state of the database, we can do a bulk_insert, and not worry about doing any upsert operation.
> hoodie.datasource.write.operation': 'bulk_insert'

7. Now we are going to set the Glue Configuration

We do this in order to have our tables written to the Glue Catalog, which would allow us to do queries with Amazon Athena as well.
```
glueConfig  = {'hoodie.datasource.hive_sync.database': 'sports_tickets_hudi', 
                'hoodie.datasource.hive_sync.table': 'hudi_tickets', 
                'hoodie.datasource.hive_sync.enable': 'true'}
```
8. We are going to add additional configuration to specify that we do not want to partition our data when writing to Amazon S3
```
unpartitionDataConfig = {'hoodie.datasource.hive_sync.partition_extractor_class': 'org.apache.hudi.hive.NonPartitionedExtractor', 
                         'hoodie.datasource.write.keygenerator.class': 'org.apache.hudi.keygen.NonpartitionedKeyGenerator'}
```

9. Lets join the configurations
```
combinedConf = {**commonConfig, **unpartitionDataConfig, **initLoadConfig}
```

10. We can now write to Amazon S3 using Apache Hudi
```
initialDF.write.format("org.apache.hudi").options(**combinedConf).mode("overwrite").save(path)
```
11. After finishing lets perform a query in order to see what was written.
```
spark.sql("use sports_tickets_hudi")
spark.sql("select * from hudi_tickets limit 5").show()
```

## 4. Updates in Apache Hudi

1. First we will filter the DF for all the updates
```
updatesDF = df.filter(df.op=='U')
```

2. We can now directly perform the write operation to Amazon S3, we just need to modify the operation in the Hudi Configuration

```
hudiUpdatesConfig = {'className' : 'org.apache.hudi', 
                'hoodie.datasource.hive_sync.use_jdbc':'false',  
                'hoodie.datasource.write.recordkey.field': 'id', 
                'hoodie.datasource.write.precombine.field': 'timestamp', 
                'hoodie.table.name': 'hudi_tickets', 
                'hoodie.datasource.write.operation': 'upsert'}
```

We are modifiying the **hoodie.datasource.write.operation** to *upserts*

3. Update the combined Conf
```
UpdatesConf = {**hudiUpdatesConfig, **glueConfig, **unpartitionDataConfig}
```

4. Lets write the updates to Amazon S3
```
updatesDF.write.format("org.apache.hudi").options(**UpdatesConf).mode("append").save(path)
```

5. Lets verify that we actually did the update. 

> [!WARNING]
> We need to refresh the Spark Catalog so it updates the Metadata of the Table so we can work with the latest version of the Hudi Table.

```
spark.catalog.refreshTable("hudi_tickets")
```

We are going to use the same ticket ID we worked in the Exploring the Data Lab

*ID = 1.45250511E8*

```
initialDF.filter(df.id=='1.45250511E8').select("id,ticketholder_id").show()
```
```
updatesDF.filter(updatesDF.id=='1.45250511E8').select("timestamp,id,ticketholder_id").show())
```

```
spark.sql("select timestamp,id,ticketholder_id from hudi_tickets where id = '1.45250511E8'").show()
```

## 5. Deletes in Apache Hudi

1. We will filter out the deletes
```
deletesDF = df.filter(df.op=='D')
```

2. We can now directly perform the delete operation to Amazon S3, we just need to modify the operation in the Hudi Configuration

```
hudiDeleteConfig = {'className' : 'org.apache.hudi', 
                'hoodie.datasource.hive_sync.use_jdbc':'false',  
                'hoodie.datasource.write.recordkey.field': 'id', 
                'hoodie.datasource.write.precombine.field': 'timestamp', 
                'hoodie.table.name': 'hudi_tickets', 
                'hoodie.datasource.write.operation': 'delete'}
```

We are modifiying the **hoodie.datasource.write.operation** to *delete*

3. Update the combined Conf
```
DeletesConf = {**hudiDeleteConfig, **glueConfig, **unpartitionDataConfig}
```

4. Before doing the delete lets verify that in the Hudi Table we have one of the ticket IDs

```
id = deletesDF.select('id').show(1)
hudi_tickets = spark.sql("select * from hudi_tickets")
hudi_tickets.filter(hudi_tickets.id==id).show()
```

As we can see we have the ticket ID in the Hudi Table

4. Lets perform the deletes to Amazon S3

```
deletesDF.write.format("org.apache.hudi").options(**DeletesConf).mode("append").save(path)
```

5. Lets verify that we actually did the deletes. 

Remember we need to refresh the Catalog in the Interactive Session

```
spark.catalog.refreshTable("hudi_tickets")
```

```
hudi_tickets = spark.sql("select * from hudi_tickets")
hudi_tickets.filter(hudi_tickets.id==id).show()
```

We have done an initial bulk_insert to a Hudi Table, we have performed Updates into the table, as well as deletes.

Now we are going to see the different types of queries we can perform to Hudi Datasets

