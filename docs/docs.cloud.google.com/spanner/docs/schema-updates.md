Spanner lets you make schema updates with no downtime. You can update the schema of an existing database in several ways:

  - In the Google Cloud console
    
    Submit an `ALTER TABLE` command on the **Spanner Studio** page.
    
    To access the **Spanner Studio** page, click **Spanner Studio** from the Database overview or Table overview page.

  - Using the `gcloud spanner` command-line tool
    
    Submit an `ALTER TABLE` command by using the [`gcloud spanner databases ddl update`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update) command.

  - Using the [client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries)

  - Using the [`projects.instances.databases.updateDdl`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl) REST API

  - Using the [`UpdateDatabaseDdl`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl) RPC API

<span id="supported_schema_updates"></span>

## Supported schema updates

Spanner supports the following schema updates of an existing database:

  - Add or drop a named schema.
  - Create a new table. Columns in new tables can be `NOT NULL` .
  - Delete a table, if no other tables are interleaved within it, and it has no secondary indexes.
  - Create or delete a table with a foreign key.
  - Add or remove a foreign key from an existing table.
  - Add a non-key column to any table. New non-key columns cannot be `NOT NULL` .
      - Drop a non-key column from any table, unless it is used by a [secondary index](https://docs.cloud.google.com/spanner/docs/secondary-indexes) , foreign key, stored generated column, or check constraint.
  - Add `NOT NULL` to a non-key column, excluding `ARRAY` columns.
  - Remove `NOT NULL` from a non-key column.
  - Change a `STRING` column to a `BYTES` column or a `BYTES` column to a `STRING` column.
  - Change a `PROTO` column to a `BYTES` column or a `BYTES` column to a `PROTO` column.
  - Change the proto message type of a `PROTO` column.
  - Add new values to an `ENUM` definition and rename existing values using `ALTER PROTO BUNDLE` .
  - Change messages defined in a `PROTO BUNDLE` in arbitrary ways, provided that modified fields of those messages are not used as keys in any table and that the existing data satisfies the new constraints.
  - Increase or decrease the length limit for a `STRING` or `BYTES` type ( [including to `MAX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language) ), unless it is a primary key column inherited by one or more child tables.
  - Increase or decrease the length limit for an `ARRAY<STRING>` , `ARRAY<BYTES>` , or `ARRAY<PROTO>` column to the maximum allowed.
  - Enable or disable [commit timestamps](https://docs.cloud.google.com/spanner/docs/commit-timestamp) in value and primary key columns.
  - Add or remove a secondary index.
  - Add or remove a check constraint from an existing table.
  - Add or remove a stored generated column from an existing table.
  - Construct a new [optimizer statistics package](https://docs.cloud.google.com/spanner/docs/query-optimizer/overview#construct-statistics-package) .
  - Create and manage [views](https://docs.cloud.google.com/spanner/docs/views) .
  - Create and manage [sequences](https://docs.cloud.google.com/spanner/docs/sequence-tasks) .
  - Create database roles and grant privileges.
  - Set, change, or drop the default value of a column.
  - Change the database options ( `default_leader` or `version_retention_period` for example).
  - Create and manage [change streams](https://docs.cloud.google.com/spanner/docs/change-streams/manage) .
  - Create and manage ML models.

## Unsupported schema updates

Spanner doesn't support the following schema updates of an existing database:

  - If there is a `PROTO` field of the `ENUM` type that is referenced by a table or index key, you can't remove `ENUM` values from the proto enums. (Removal of `ENUM` values from enums used by `ENUM<>` columns is supported, including when those columns are used as keys.)

  - Change a `STRING(36)` column to a `UUID` column or a `UUID` column to a `STRING(36)` column.

<span id="schema_update_performance"></span>

## Schema update performance

> **Note:** For more information, see [schema updates best practices](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices) .

Schema updates in Spanner don't require downtime. When you issue a batch of DDL statements to a Spanner database, you can continue writing and reading from the database without interruption while Spanner applies the update as a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) .

The time it takes to execute a DDL statement depends on whether the update requires validation of the existing data or backfill of any data. For example, if you add the `NOT NULL` annotation to an existing column, Spanner must read all the values in the column to make sure that the column does not contain any `NULL` values. This step can take a long time if there is a lot of data to validate. Another example is if you're adding an index to a database: Spanner backfills the index using existing data, and that process can take a long time depending on how the index's definition and the size of the corresponding base table. However, if you add a new column to a table, there is no existing data to validate, so Spanner can make the update more quickly.

In summary, schema updates that don't require Spanner to validate existing data can happen in minutes. Schema updates that require validation can take longer, depending on the amount of existing data that needs to be validated, but data validation happens in the background at a lower priority than production traffic. Schema updates that require data validation are discussed in more detail in the next section.

<span id="validate_view_definitions"></span>

## Schema updates validated against view definitions

When you make a schema update, Spanner validates that the update won't invalidate the queries used to define existing views. If validation is successful, the schema update succeeds. If validation is not successful, the schema update fails. Check [Best practices when creating views](https://docs.cloud.google.com/spanner/docs/views#create-view-guidance) for details.

## Schema updates that require data validation

You can make schema updates that require validating that the existing data meets the new constraints. When a schema update requires data validation, Spanner disallows conflicting schema updates to the affected schema entities and validates the data in the background. If validation is successful, the schema update succeeds. If validation is not successful, the schema update does not succeed. Validation operations are executed as [long-running operations](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . You can check the status of these operations to determine if they succeeded or failed.

For example, suppose you have defined the following `music.proto` file with a `RecordLabel` enum and `Songwriter` protocol message:

``` 
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

To add a `Songwriters` table in your schema:

### GoogleSQL

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

The following schema updates are allowed, but they require validation and might take longer to complete, depending on the amount of existing data:

  - Adding the `NOT NULL` annotation to a non-key column. For example:
    
        ALTER TABLE Songwriters ALTER COLUMN Nickname STRING(MAX) NOT NULL;

  - Reducing the length of a column. For example:
    
        ALTER TABLE Songwriters ALTER COLUMN FirstName STRING(10);

  - Altering from `BYTES` to `STRING` . For example:
    
        ALTER TABLE Songwriters ALTER COLUMN OpaqueData STRING(MAX);

  - Altering from `INT64/INT32` to `ENUM` . For example:
    
        ALTER TABLE Albums ALTER COLUMN Label googlesql.example.music.RecordLabel;

  - Removing existing values from the `RecordLabel` enum definition.

  - Enabling [commit timestamps](https://docs.cloud.google.com/spanner/docs/commit-timestamp#converting_a_timestamp_column_to_a_commit_timestamp_column) on an existing `TIMESTAMP` column. For example:
    
        ALTER TABLE Albums ALTER COLUMN LastUpdateTime SET OPTIONS (allow_commit_timestamp = true);

  - Adding a check constraint to an existing table.

  - Adding a stored generated column to an existing table.

  - Creating a new table with a foreign key.

  - Adding a foreign key to an existing table.

These schema updates fail if the underlying data does not satisfy the new constraints. For example, the `ALTER TABLE Songwriters ALTER COLUMN Nickname STRING(MAX) NOT NULL` statement fails if any value in the `Nickname` column is `NULL` , because the existing data does not meet the `NOT NULL` constraint of the new definition.

Data validation can take from several minutes to many hours. The time to complete data validation depends on:

  - The size of the dataset
  - The compute capacity of the instance
  - The load on the instance

Some schema updates can change the behavior of requests to the database before the schema update completes. For example, if you're adding `NOT NULL` to a column, Spanner almost immediately begins rejecting writes for new requests that use `NULL` for the column. If the new schema update ultimately fails for data validation, there will have been a period of time when writes were blocked, even if they would have been accepted by the old schema.

You can cancel a long-running data validation operation using the [`projects.instances.databases.operations.cancel`](https://docs.cloud.google.com/spanner/reference/rest/v1/projects.instances.databases.operations/cancel) method or using [`gcloud spanner operations`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations) .

## Schema versions created during schema updates

Spanner uses schema versioning so that there is no downtime during a schema update to a large database. Spanner maintains the older schema version to support reads while the schema update is processed. Spanner then creates one or more new versions of the schema to process the schema update. Each version contains the result of a collection of statements in a single atomic change.

The schema versions don't necessarily correspond one-to-one with either batches of DDL statements or individual DDL statements. Some individual DDL statements, such as index creation for existing base tables or statements that require data validation, result in multiple schema versions. In other cases, several DDL statements can be batched together in a single version. Old schema versions can consume significant server and storage resources, and they are retained until they expire (no longer needed to serve reads of earlier versions of data).

> **Note:** If you need to add multiple indexes, see the [options for large updates](https://docs.cloud.google.com/spanner/docs/schema-updates-best-practices#large-updates) .

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
<td><code dir="ltr" translate="no">CREATE TABLE</code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">CREATE INDEX</code></td>
<td><p>Minutes to hours, if the base table is created before the index.</p>
<p>Minutes, if the statement is executed at the same time as the <code dir="ltr" translate="no">CREATE TABLE</code> statement for the base table.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">DROP TABLE</code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">DROP INDEX</code></td>
<td>Minutes</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">ALTER TABLE ... ADD COLUMN</code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">ALTER TABLE ... ALTER COLUMN</code></td>
<td><p>Minutes to hours, if <a href="https://docs.cloud.google.com/spanner/docs/schema-updates#updates-that-require-validation">background validation</a> is required.</p>
<p>Minutes, if background validation is not required.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">ALTER TABLE ... DROP COLUMN</code></td>
<td>Minutes</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">ANALYZE</code></td>
<td><p>Minutes to hours, depending on the database size.</p></td>
</tr>
</tbody>
</table>

## Data type changes and change streams

If you change the data type of a column that a [change stream](https://docs.cloud.google.com/spanner/docs/change-streams) watches, the `column_types` field of relevant subsequent [change stream records](https://docs.cloud.google.com/spanner/docs/change-streams/details#data-change-records) reflects its new type, as does the `old_values` JSON data within the records' `mods` field.

The `new_values` of a change stream record's `mods` field always matches a column's current type. Changing a watched column's data type does not affect any change stream records predating that change.

In the particular case of a `BYTES` -to- `STRING` change, Spanner [validates the column's old values](https://docs.cloud.google.com/spanner/docs/schema-updates#updates-that-require-validation) as part of the schema update. As a result, Spanner has safely decoded the old `BYTES` -type values into strings by the time it writes any subsequent change stream records.
