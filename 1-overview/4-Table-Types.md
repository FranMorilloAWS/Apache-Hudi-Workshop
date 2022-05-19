# Table Types

When you create a Hudi dataset, you specify that the dataset is either copy on write or merge on read.

> Copy on Write (CoW): Data is stored in a columnar format (Parquet), and each update creates a new version of files during a write. CoW is the default storage type.

> Merge on Read (MoR): Data is stored using a combination of columnar (Parquet) and row-based (Avro) formats. Updates are logged to row-based delta files and are compacted as needed to create new versions of the columnar files. 

With CoW datasets, each time there is an update to a record, the file that contains the record is rewritten with the updated values. With a MoR dataset, each time there is an update, Hudi writes only the row for the changed record. MoR is better suited for write- or change-heavy workloads with fewer reads. CoW is better suited for read-heavy workloads on data that changes less frequently.


