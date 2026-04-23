This document describes best practices for updating schemas.

## Procedures before you begin the schema update

Before you issue a schema update:

  - Ensure that all existing data in the database complies with the constraints introduced by the schema update. Since some schema updates depend on the actual data, not just the current schema, a successful update in a test database doesn't guarantee success in a production database. Here are some common examples:
    
      - If you're adding a `NOT NULL` annotation to an existing column, check that the column does not contain any existing `NULL` values.
      - If you're shortening the allowed length of a `STRING` or `BYTES` column, check that all existing values in that column meet the length constraint.

  - If you're writing to a column, table, or index that is undergoing a schema update, ensure that the values that you're writing meet the new constraints.

  - If you're dropping a column, table, or index, make sure you are not still writing to or reading from it.

## Limit the frequency of schema updates

If you perform too many schema updates in a short period of time, Spanner may [`throttle`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseDdlMetadata#FIELDS.throttled) the processing of queued schema updates. This is because Spanner limits the amount of space for storing schema versions. Your schema update may be throttled if there are too many old schema versions within the retention period. The maximum rate of schema changes depends on many [factors](https://docs.cloud.google.com/spanner/docs/schema-updates#performance) , one of them being the total number of columns in the database. For example, a database with 2000 columns (roughly 2000 rows in [`INFORMATION_SCHEMA.COLUMNS`](https://docs.cloud.google.com/spanner/docs/information-schema#information_schemacolumns) ) is able to perform at most 1500 schema changes (fewer if the schema change requires multiple versions) within the retention period. To see the state of ongoing schema updates, use the [`gcloud spanner operations list`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/list) command and filter by operations of type `DATABASE_UPDATE_DDL` . To cancel an ongoing schema update, use the [`gcloud spanner operations cancel`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel) command and specify the operation ID.

How your DDL statements are batched, and their order within each batch, can affect the number of schema versions that result. To maximize the number of schema updates you can perform over any given period of time, you should use batching that minimizes the number of schema versions. Some guidelines are described in [large updates](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) .

As described in [schema versions](https://docs.cloud.google.com/spanner/docs/schema-updates#schema-versions) , some DDL statements will create multiple schema versions, and these are important when considering batching and order within each batch. There are two main types of statements that might create multiple schema versions:

  - Statements that might need to backfill index data, like [`CREATE INDEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create-index) .
  - Statements that force Spanner to validate existing data, like adding `NOT NULL` or length constraints.

These types of statements don't *always* create multiple schema versions, though. Spanner will try to detect when these types of statements can be optimized to avoid using multiple schema versions, which depends on batching. For example, a `CREATE INDEX` statement that occurs in the same batch as a `CREATE TABLE` statement for the index's base table, without any intervening statements for other tables, can avoid needing to backfill the index data because Spanner can guarantee that the base table is empty at the time the index is created. The [large updates](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) section describes how to use this property to create many indexes efficiently.

If you cannot batch your DDL statements to avoid creating many schema versions, you should limit the number of schema updates to a single database's schema within its retention period. Increase the time window in which you make schema updates to allow Spanner to remove earlier versions of the schema before new versions are created.

  - For some relational database management systems, there are software packages that make a long series of upgrade and downgrade schema updates to the database on every production deployment. These types of processes are not recommended for Spanner.
  - Spanner is optimized to use primary keys to partition data for [multi-tenancy solutions](https://docs.cloud.google.com/spanner/docs/implement-multi-tenancy) . If you use a multi-tenancy solution that uses separate tables for each customer, be aware that schema updates across many customers at once can result in a large backlog of schema update operations that take a long time to complete.
  - Schema updates that require validation or index backfill use more server resources because each statement creates multiple versions of the schema internally.

## Order of execution of statements in batches

If you use the Google Cloud CLI, REST API, or the RPC API, you can issue a batch of one or more `CREATE` , `ALTER` , or `DROP` [statements](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language) .

Spanner applies statements from the same batch in order, stopping at the first error. If applying a statement results in an error, that statement is rolled back. The results of any previously applied statements in the batch are not rolled back. This in-order statement application means that if you would like statements that require unavoidable backfill to run in parallel (like creating multiple indexes on large, existing tables), you should submit those statements in separate batches, because each backfill could take a long time. If you are creating a new table with indexes, on the other hand, the best practice is to [place them together (CREATE TABLE followed by CREATE INDEX)](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) in a single batch to avoid backfill entirely.

Spanner might combine and reorder statements from different batches, potentially mixing statements from different batches into one atomic change that is applied to the database. Within each atomic change, statements from different batches happen in an arbitrary order. For example, if one batch of statements contains `ALTER TABLE table_name ALTER COLUMN column_name STRING(50)` and another batch of statements contains `ALTER TABLE table_name ALTER COLUMN column_name STRING(20)` , Spanner will leave that column in one of those two states, but the state it is left in is not deterministic.

## Options for large schema updates

The best way to create a table and a large number of indexes on that table is to create all of them at the same time, so that there will be only a single schema version created. It's best practice to create the indexes immediately following the table in the list of DDL statements. You can create the table and its indexes when you create the database, or in a single large batch of statements. If you need to create many tables, each with many indexes, you can include all the statements in a single batch. You can include several thousand statements in a single batch when all the statements can be executed together using a single schema version.

When a statement requires backfilling index data or performing data validation, it can't be executed in a single schema version. This happens for `CREATE INDEX` statements when the index's base table already exists (either because it was created in a previous batch of DDL statements, or because there was a statement in the batch between the `CREATE TABLE` and `CREATE INDEX` statements that required multiple schema versions). Spanner requires that there are no more than 10 such statements in a single batch. Index creation that requires backfilling, in particular, uses several schema versions per index, and so it is a good rule of thumb to create no more than 3 new indexes requiring backfilling per day (no matter how they are batched, unless such batching can avoid backfilling).

For example, this batch of statements will use a single schema version:

### GoogleSQL

    CREATE TABLE Singers (
    SingerId   INT64 NOT NULL,
    FirstName  STRING(1024),
    LastName   STRING(1024),
    ) PRIMARY KEY (SingerId);
    
    CREATE INDEX SingersByFirstName ON Singers(FirstName);
    
    CREATE INDEX SingersByLastName ON Singers(LastName);
    
    CREATE TABLE Albums (
    SingerId   INT64 NOT NULL,
    AlbumId    INT64 NOT NULL,
    AlbumTitle STRING(MAX),
    ) PRIMARY KEY (SingerId, AlbumId);
    
    CREATE INDEX AlbumsByTitle ON Albums(AlbumTitle);

In contrast, this batch will use many schema versions, because `UnrelatedIndex` requires backfilling (since its base table must have already existed), and that forces all the following indexes to also require backfilling (even though they're in the same batch as their base tables):

### GoogleSQL

    CREATE TABLE Singers (
    SingerId   INT64 NOT NULL,
    FirstName  STRING(1024),
    LastName   STRING(1024),
    ) PRIMARY KEY (SingerId);
    
    CREATE TABLE Albums (
    SingerId   INT64 NOT NULL,
    AlbumId    INT64 NOT NULL,
    AlbumTitle STRING(MAX),
    ) PRIMARY KEY (SingerId, AlbumId);
    
    CREATE INDEX UnrelatedIndex ON UnrelatedTable(UnrelatedIndexKey);
    
    CREATE INDEX SingersByFirstName ON Singers(FirstName);
    
    CREATE INDEX SingersByLastName ON Singers(LastName);
    
    CREATE INDEX AlbumsByTitle ON Albums(AlbumTitle);

It would be better to move the creation of `UnrelatedIndex` to the end of the batch, or to a different batch, to minimize schema versions.

## Wait for API requests to complete

When making [`projects.instances.databases.updateDdl`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl) (REST API) or [`UpdateDatabaseDdl`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl) (RPC API) requests, use [`projects.instances.databases.operations.get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.operations/get) (REST API) or [`GetOperation`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations.GetOperation) (RPC API), respectively, to wait for each request to complete before starting a new request. Waiting for each request to complete allows your application to track the progress of your schema updates. It also keeps the backlog of pending schema updates to a manageable size.

## Bulk loading

When bulk loading data into a new table, you can create secondary indexes before or after loading the data. Loading data is faster if you create indexes after loading, but this means indexes must be backfilled.

If you load data first and then create indexes, data ingestion is faster because only the table is being written to, and the later index backfills can write the index data in optimized batches that are more efficient than writing the index data together with the table data. However, backfilling indexes requires multiple schema versions and has limits; as noted in [options for large updates](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) , you should create no more than 10 indexes requiring backfill in a single batch, and it is best to create no more than 3 such indexes per day.

Alternatively, you can create tables and indexes in the same batch, as described in [options for large updates](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) . This avoids index backfilling, but bulk loading data will be slower because each index must be updated as data is loaded.

Which choice is better in a given situation depends on how much data will be loaded, the specific table and index keys, how many indexes are needed, and how often the bulk load operations will be required in the same database. A rule of thumb is that it's better to create the indexes separately if a large amount of data needs to be loaded into each table and only a few indexes are required.
