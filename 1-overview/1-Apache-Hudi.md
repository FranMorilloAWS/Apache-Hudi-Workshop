# What is Apache Hudi

![Hudi Logo](img/Hudi-Logo.png)

Apache Hudi is an open-source data management framework used to simplify incremental data processing and data pipeline development. Apache Hudi enables you to manage data at the record-level in Amazon S3 to simplify Change Data Capture (CDC) and streaming data ingestion, and provides a framework to handle data privacy use cases requiring record level updates and deletes. Data sets managed by Apache Hudi are stored in S3 using open storage formats, and integrations with Presto, Apache Hive, Apache Spark, and AWS Glue Data Catalog give you near real-time access to updated data using familiar tools.

Hudi handles data insertion and update events without creating many small files that can cause performance issues for analytics. Apache Hudi automatically tracks changes and merges files so that they remain optimally sized. This avoids the need to build custom solutions that monitor and re-write many small files into fewer large files.
