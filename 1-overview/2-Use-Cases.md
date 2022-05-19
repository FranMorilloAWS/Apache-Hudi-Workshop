# Use Cases

Using Hudi, you can perform record-level inserts, updates, and deletes on S3. You create datasets and tables and Hudi manages the underlying data format. Hudi uses Apache Parquet, and Apache Avro for data storage, and includes built-in integrations with Spark, Hive, and Presto, enabling you to query Hudi datasets using the same tools that you use today with near real-time access to fresh data. These features make Hudi suitable for the following use cases:
* Working with streaming data from sensors and other Internet of Things (IoT) devices that require specific data insertion and update events.
* Complying with data privacy regulations in applications where users might choose to be forgotten or modify their consent for how their data can be used.
* Implementing a change data capture (CDC) system that allows you to apply changes to a dataset over time.
