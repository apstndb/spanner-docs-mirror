Spanner lets you make schema updates with no downtime. You can update the schema of an existing database in several ways:

  - In the Google Cloud console
    
    Submit an `  ALTER TABLE  ` command on the **Spanner Studio** page.
    
    To access the **Spanner Studio** page, click **Spanner Studio** from the Database overview or Table overview page.

  - Using the `  gcloud spanner  ` command-line tool
    
    Submit an `  ALTER TABLE  ` command by using the [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) command.

  - Using the [client libraries](/spanner/docs/reference/libraries)

  - Using the [`  projects.instances.databases.updateDdl  `](/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl) REST API

  - Using the [`  UpdateDatabaseDdl  `](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl) RPC API

## Supported schema updates

Spanner supports the following schema updates of an existing database:

  - Add or drop a named schema.
  - Create a new table. Columns in new tables can be `  NOT NULL  ` .
  - Delete a table, if no other tables are interleaved within it, and it has no secondary indexes.
  - Create or delete a table with a foreign key.
  - Add or remove a foreign key from an existing table.
  - Add a non-key column to any table. New non-key columns cannot be `  NOT NULL  ` .
      - Drop a non-key column from any table, unless it is used by a [secondary index](/spanner/docs/secondary-indexes) , foreign key, stored generated column, or check constraint.
  - Add `  NOT NULL  ` to a non-key column, excluding `  ARRAY  ` columns.
  - Remove `  NOT NULL  ` from a non-key column.
  - Change a `  STRING  ` column to a `  BYTES  ` column or a `  BYTES  ` column to a `  STRING  ` column.
  - Change a `  PROTO  ` column to a `  BYTES  ` column or a `  BYTES  ` column to a `  PROTO  ` column.
  - Change the proto message type of a `  PROTO  ` column.
  - Add new values to an `  ENUM  ` definition and rename existing values using `  ALTER PROTO BUNDLE  ` .
  - Change messages defined in a `  PROTO BUNDLE  ` in arbitrary ways, provided that modified fields of those messages are not used as keys in any table and that the existing data satisfies the new constraints.
  - Increase or decrease the length limit for a `  STRING  ` or `  BYTES  ` type ( [including to `  MAX  `](/spanner/docs/reference/standard-sql/data-definition-language) ), unless it is a primary key column inherited by one or more child tables.
  - Increase or decrease the length limit for an `  ARRAY<STRING>  ` , `  ARRAY<BYTES>  ` , or `  ARRAY<PROTO>  ` column to the maximum allowed.
  - Enable or disable [commit timestamps](/spanner/docs/commit-timestamp) in value and primary key columns.
  - Add or remove a secondary index.
  - Add or remove a check constraint from an existing table.
  - Add or remove a stored generated column from an existing table.
  - Construct a new [optimizer statistics package](/spanner/docs/query-optimizer/overview#construct-statistics-package) .
  - Create and manage [views](/spanner/docs/views) .
  - Create and manage [sequences](/spanner/docs/sequence-tasks) .
  - Create database roles and grant privileges.
  - Set, change, or drop the default value of a column.
  - Change the database options ( `  default_leader  ` or `  version_retention_period  ` for example).
  - Create and manage [change streams](/spanner/docs/change-streams/manage) .
  - Create and manage ML models.

## Unsupported schema updates

Spanner doesn't support the following schema updates of an existing database:

  - If there is a `  PROTO  ` field of the `  ENUM  ` type that is referenced by a table or index key, you can't remove `  ENUM  ` values from the proto enums. (Removal of `  ENUM  ` values from enums used by `  ENUM<>  ` columns is supported, including when those columns are used as keys.)

  - Change a `  STRING(36)  ` column to a `  UUID  ` column or a `  UUID  ` column to a `  STRING(36)  ` column.

## Schema update performance

Schema updates in Spanner don't require downtime. When you issue a batch of DDL statements to a Spanner database, you can continue writing and reading from the database without interruption while Spanner applies the update as a [long-running operation](/spanner/docs/manage-long-running-operations) .

The time it takes to execute a DDL statement depends on whether the update requires validation of the existing data or backfill of any data. For example, if you add the `  NOT NULL  ` annotation to an existing column, Spanner must read all the values in the column to make sure that the column does not contain any `  NULL  ` values. This step can take a long time if there is a lot of data to validate. Another example is if you're adding an index to a database: Spanner backfills the index using existing data, and that process can take a long time depending on how the index's definition and the size of the corresponding base table. However, if you add a new column to a table, there is no existing data to validate, so Spanner can make the update more quickly.

In summary, schema updates that don't require Spanner to validate existing data can happen in minutes. Schema updates that require validation can take longer, depending on the amount of existing data that needs to be validated, but data validation happens in the background at a lower priority than production traffic. Schema updates that require data validation are discussed in more detail in the next section.

## Schema updates validated against view definitions

When you make a schema update, Spanner validates that the update won't invalidate the queries used to define existing views. If validation is successful, the schema update succeeds. If validation is not successful, the schema update fails. Check [Best practices when creating views](/spanner/docs/views#create-view-guidance) for details.

## Schema updates that require data validation

You can make schema updates that require validating that the existing data meets the new constraints. When a schema update requires data validation, Spanner disallows conflicting schema updates to the affected schema entities and validates the data in the background. If validation is successful, the schema update succeeds. If validation is not successful, the schema update does not succeed. Validation operations are executed as [long-running operations](/spanner/docs/manage-long-running-operations) . You can check the status of these operations to determine if they succeeded or failed.

For example, suppose you have defined the following `  music.proto  ` file with a `  RecordLabel  ` enum and `  Songwriter  ` protocol message:

``` text
  enum RecordLabel {
    COOL_MUSIC_INC = 0;
    PACIFIC_ENTERTAINMENT = 1;
    XYZ_RECORDS = 2;
  }

  message Songwriter {
    required string nationality   = 1;
    optional int64  year_of_birth = 2;
  }
```

To add a `  Songwriters  ` table in your schema:

### GoogleSQL

``` text
CREATE PROTO BUNDLE (
  googlesql.example.music.Songwriter,
  googlesql.example.music.RecordLabel,
);

CREATE TABLE Songwriters (
  Id         INT64 NOT NULL,
  FirstName  STRING(1024),
  LastName   STRING(1024),
  Nickname   STRING(MAX),
  OpaqueData BYTES(MAX),
  SongWriter googlesql.example.music.Songwriter
) PRIMARY KEY (Id);

CREATE TABLE Albums (
  SongwriterId     INT64 NOT NULL,
  AlbumId          INT64 NOT NULL,
  AlbumTitle       STRING(MAX),
  Label            INT32
) PRIMARY KEY (SongwriterId, AlbumId);
```

The following schema updates are allowed, but they require validation and might take longer to complete, depending on the amount of existing data:

  - Adding the `  NOT NULL  ` annotation to a non-key column. For example:
    
    ``` text
    ALTER TABLE Songwriters ALTER COLUMN Nickname STRING(MAX) NOT NULL;
    ```

  - Reducing the length of a column. For example:
    
    ``` text
    ALTER TABLE Songwriters ALTER COLUMN FirstName STRING(10);
    ```

  - Altering from `  BYTES  ` to `  STRING  ` . For example:
    
    ``` text
    ALTER TABLE Songwriters ALTER COLUMN OpaqueData STRING(MAX);
    ```

  - Altering from `  INT64/INT32  ` to `  ENUM  ` . For example:
    
    ``` text
    ALTER TABLE Albums ALTER COLUMN Label googlesql.example.music.RecordLabel;
    ```

  - Removing existing values from the `  RecordLabel  ` enum definition.

  - Enabling [commit timestamps](/spanner/docs/commit-timestamp#converting_a_timestamp_column_to_a_commit_timestamp_column) on an existing `  TIMESTAMP  ` column. For example:
    
    ``` text
    ALTER TABLE Albums ALTER COLUMN LastUpdateTime SET OPTIONS (allow_commit_timestamp = true);
    ```

  - Adding a check constraint to an existing table.

  - Adding a stored generated column to an existing table.

  - Creating a new table with a foreign key.

  - Adding a foreign key to an existing table.

These schema updates fail if the underlying data does not satisfy the new constraints. For example, the `  ALTER TABLE Songwriters ALTER COLUMN Nickname STRING(MAX) NOT NULL  ` statement fails if any value in the `  Nickname  ` column is `  NULL  ` , because the existing data does not meet the `  NOT NULL  ` constraint of the new definition.

Data validation can take from several minutes to many hours. The time to complete data validation depends on:

  - The size of the dataset
  - The compute capacity of the instance
  - The load on the instance

Some schema updates can change the behavior of requests to the database before the schema update completes. For example, if you're adding `  NOT NULL  ` to a column, Spanner almost immediately begins rejecting writes for new requests that use `  NULL  ` for the column. If the new schema update ultimately fails for data validation, there will have been a period of time when writes were blocked, even if they would have been accepted by the old schema.

You can cancel a long-running data validation operation using the [`  projects.instances.databases.operations.cancel  `](/spanner/reference/rest/v1/projects.instances.databases.operations/cancel) method or using [`  gcloud spanner operations  `](/sdk/gcloud/reference/spanner/operations) .

## Order of execution of statements in batches

If you use the Google Cloud CLI, REST API, or the RPC API, you can issue a batch of one or more `  CREATE  ` , `  ALTER  ` , or `  DROP  ` [statements](/spanner/docs/reference/standard-sql/data-definition-language) .

Spanner applies statements from the same batch in order, stopping at the first error. If applying a statement results in an error, that statement is rolled back. The results of any previously applied statements in the batch are not rolled back. This in-order statement application means that if you would like certain statements to run in parallel, like index backfills that could each take a long time, you should submit those statements in separate batches.

Spanner might combine and reorder statements from different batches, potentially mixing statements from different batches into one atomic change that is applied to the database. Within each atomic change, statements from different batches happen in an arbitrary order. For example, if one batch of statements contains `  ALTER TABLE MyTable ALTER COLUMN MyColumn STRING(50)  ` and another batch of statements contains `  ALTER TABLE MyTable ALTER COLUMN MyColumn STRING(20)  ` , Spanner will leave that column in one of those two states, but it's not specified which.

## Schema versions created during schema updates

Spanner uses schema versioning so that there is no downtime during a schema update to a large database. Spanner maintains the older schema version to support reads while the schema update is processed. Spanner then creates one or more new versions of the schema to process the schema update. Each version contains the result of a collection of statements in a single atomic change.

The schema versions don't necessarily correspond one-to-one with either batches of DDL statements or individual DDL statements. Some individual DDL statements, such as index creation for existing base tables or statements that require data validation, result in multiple schema versions. In other cases, several DDL statements can be batched together in a single version. Old schema versions can consume significant server and storage resources, and they are retained until they expire (no longer needed to serve reads of earlier versions of data).

**Note:** If you need to add multiple indexes, see the [options for large updates](#large-updates) .

The following table shows how long it takes Spanner to update a schema.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Schema operation</th>
<th>Estimated duration</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       CREATE TABLE      </code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       CREATE INDEX      </code></td>
<td><p>Minutes to hours, if the base table is created before the index.</p>
<p>Minutes, if the statement is executed at the same time as the <code dir="ltr" translate="no">        CREATE TABLE       </code> statement for the base table.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       DROP TABLE      </code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       DROP INDEX      </code></td>
<td>Minutes</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ALTER TABLE ... ADD COLUMN      </code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ALTER TABLE ... ALTER COLUMN      </code></td>
<td><p>Minutes to hours, if <a href="#updates-that-require-validation">background validation</a> is required.</p>
<p>Minutes, if background validation is not required.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ALTER TABLE ... DROP COLUMN      </code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ANALYZE      </code></td>
<td><p>Minutes to hours, depending on the database size.</p></td>
</tr>
</tbody>
</table>

## Data type changes and change streams

If you change the data type of a column that a [change stream](/spanner/docs/change-streams) watches, the `  column_types  ` field of relevant subsequent [change stream records](/spanner/docs/change-streams/details#data-change-records) reflects its new type, as does the `  old_values  ` JSON data within the records' `  mods  ` field.

The `  new_values  ` of a change stream record's `  mods  ` field always matches a column's current type. Changing a watched column's data type does not affect any change stream records predating that change.

In the particular case of a `  BYTES  ` -to- `  STRING  ` change, Spanner [validates the column's old values](#updates-that-require-validation) as part of the schema update. As a result, Spanner has safely decoded the old `  BYTES  ` -type values into strings by the time it writes any subsequent change stream records.

## Best practices for schema updates

The following sections describe best practices for updating schemas.

### Procedures before you issue the schema update

Before you issue a schema update:

  - Verify that all of the existing data in the database that you're changing meets the constraints that the schema update is imposing. Because the success of some types of schema updates depends on the data in the database and not just its current schema, a successful schema update of a test database does not guarantee a successful schema update of a production database. Here are some common examples:
    
      - If you're adding a `  NOT NULL  ` annotation to an existing column, check that the column does not contain any existing `  NULL  ` values.
      - If you're shortening the allowed length of a `  STRING  ` or `  BYTES  ` column, check that all existing values in that column meet the length constraint.

  - If you're writing to a column, table, or index that is undergoing a schema update, ensure that the values that you're writing meet the new constraints.

  - If you're dropping a column, table, or index, make sure you are not still writing to or reading from it.

### Limit the frequency of schema updates

If you perform too many schema updates in a short period of time, Spanner may [`  throttle  `](/spanner/docs/reference/rest/v1/UpdateDatabaseDdlMetadata#FIELDS.throttled) the processing of queued schema updates. This is because Spanner limits the amount of space for storing schema versions. Your schema update may be throttled if there are too many old schema versions within the retention period. The maximum rate of schema changes depends on many [factors](/spanner/docs/schema-updates#performance) , one of them being the total number of columns in the database. For example, a database with 2000 columns (roughly 2000 rows in [`  INFORMATION_SCHEMA.COLUMNS  `](/spanner/docs/information-schema#information_schemacolumns) ) is able to perform at most 1500 schema changes (fewer if the schema change requires multiple versions) within the retention period. To see the state of ongoing schema updates, use the [`  gcloud spanner operations list  `](/sdk/gcloud/reference/spanner/operations/list) command and filter by operations of type `  DATABASE_UPDATE_DDL  ` . To cancel an ongoing schema update, use the [`  gcloud spanner operations cancel  `](/sdk/gcloud/reference/spanner/operations/cancel) command and specify the operation ID.

How your DDL statements are batched, and their order within each batch, can affect the number of schema versions that result. To maximize the number of schema updates you can perform over any given period of time, you should use batching that minimizes the number of schema versions. Some rules of thumb are described in [large updates](#large-updates) .

As described in [schema versions](#schema-versions) , some DDL statements will create multiple schema versions, and these ones are important when considering batching and order within each batch. There are two main types of statements that might create multiple schema versions:

  - Statements that might need to backfill index data, like `  CREATE INDEX  `
  - Statements that might need to validate existing data, like adding `  NOT NULL  `

These types of statements don't *always* create multiple schema versions, though. Spanner will try to detect when these types of statements can be optimized to avoid using multiple schema versions, which depends on batching. For example, a `  CREATE INDEX  ` statement that occurs in the same batch as a `  CREATE TABLE  ` statement for the index's base table, without any intervening statements for other tables, can avoid needing to backfill the index data because Spanner can guarantee that the base table is empty at the time the index is created. The [large updates](#large-updates) section describes how to use this property to create many indexes efficiently.

If you cannot batch your DDL statements to avoid creating many schema versions, you should limit the number of schema updates to a single database's schema within its retention period. Increase the time window in which you make schema updates to allow Spanner to remove earlier versions of the schema before new versions are created.

  - For some relational database management systems, there are software packages that make a long series of upgrade and downgrade schema updates to the database on every production deployment. These types of processes are not recommended for Spanner.
  - Spanner is optimized to use primary keys to partition data for [multi-tenancy solutions](/spanner/docs/schema-and-data-model#multitenancy) . Multi-tenancy solutions that use separate tables for each customer can result in a large backlog of schema update operations that take a long time to complete.
  - Schema updates that require validation or index backfill use more server resources because each statement creates multiple versions of the schema internally.

### Options for large schema updates

The best way to create a table and a large number of indexes on that table is to create all of them at the same time, so that there will be only a single schema version created. It's best practice to create the indexes immediately following the table in the list of DDL statements. You can create the table and its indexes when you create the database, or in a single large batch of statements. If you need to create many tables, each with many indexes, you can include all the statements in a single batch. You can include several thousand statements in a single batch when all the statements can be executed together using a single schema version.

When a statement requires backfilling index data or performing data validation, it can't be executed in a single schema version. This happens for `  CREATE INDEX  ` statements when the index's base table already exists (either because it was created in a previous batch of DDL statements, or because there was a statement in the batch between the `  CREATE TABLE  ` and `  CREATE INDEX  ` statements that required multiple schema versions). Spanner requires that there are no more than 10 such statements in a single batch. Index creation that requires backfilling, in particular, uses several schema versions per index, and so it is a good rule of thumb to create no more than 3 new indexes requiring backfilling per day (no matter how they are batched, unless such batching can avoid backfilling).

For example, this batch of statements will use a single schema version:

### GoogleSQL

``` text
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
```

In contrast, this batch will use many schema versions, because `  UnrelatedIndex  ` requires backfilling (since its base table must have already existed), and that forces all the following indexes to also require backfilling (even though they're in the same batch as their base tables):

### GoogleSQL

``` text
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
```

It would be better to move the creation of `  UnrelatedIndex  ` to the end of the batch, or to a different batch, to minimize schema versions.

### Wait for API requests to complete

When making [`  projects.instances.databases.updateDdl  `](/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl) (REST API) or [`  UpdateDatabaseDdl  `](/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl) (RPC API) requests, use [`  projects.instances.databases.operations.get  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/get) (REST API) or [`  GetOperation  `](/spanner/docs/reference/rpc/google.longrunning#google.longrunning.Operations.GetOperation) (RPC API), respectively, to wait for each request to complete before starting a new request. Waiting for each request to complete allows your application to track the progress of your schema updates. It also keeps the backlog of pending schema updates to a manageable size.

### Bulk loading

If you are bulk loading data into your tables after they are created, it is usually more efficient to create indexes after the data is loaded. If you are adding several indexes, it might be more efficient to create the database with all tables and indexes in the initial schema, as described in the [options for large updates](#large-updates) .
