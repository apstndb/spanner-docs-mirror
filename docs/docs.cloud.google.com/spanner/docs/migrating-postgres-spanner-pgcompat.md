This page explains how to migrate an open source PostgreSQL database (from now on referred to as just PostgreSQL) to a Spanner PostgreSQL-dialect database (from now on referred to as Spanner).

For information about migrating to Spanner and the GoogleSQL dialect, see [Migrating from PostgreSQL to Spanner (GoogleSQL dialect)](https://docs.cloud.google.com/spanner/docs/migrating-postgres-spanner) .

## Migration constraints

Spanner uses certain concepts differently from other enterprise database management tools, so you might need to adjust your application's architecture to take full advantage of its capabilities. You might also need to supplement Spanner with other services from Google Cloud to meet your needs.

### Stored procedures and triggers

Spanner does not support running user code in the database level, so as part of the migration, business logic implemented by database-level stored procedures and triggers must be moved into the application.

### Sequences

Spanner recommends using UUID Version 4 as the default method to generate primary key values. The `GENERATE_UUID()` function ( [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/utility-functions#generate_uuid) , [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators#utility) ) returns UUID Version 4 values represented as `STRING` type.

If you need to generate integer values, Spanner supports bit-reversed positive sequences ( [GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create-sequence) , [PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/data-definition-language#create_sequence) ), which produce values that distribute evenly across the positive 64-bit number space. You can use these numbers to avoid hotspotting issues.

For more information, see [primary key default value strategies](https://docs.cloud.google.com/spanner/docs/primary-key-default-value) .

### Access controls

Spanner supports fine-grained access control at the table and column level. Fine-grained access control for views is not supported. For more information, see [About fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) .

## Migration process

Migration involves the following tasks:

  - Mapping a PostgreSQL schema to Spanner.
  - Translating SQL queries.
  - Creating a Spanner instance, database, and schema.
  - Refactoring the application to work with your Spanner database.
  - Migrating your data.
  - Verifying the new system and moving it to production status.

## Step 1: Map your PostgreSQL schema to Spanner

Your first step in moving a database from open source PostgreSQL to Spanner is to determine what schema changes you must make.

### Primary keys

In Spanner, every table that must store more than one row must have a primary key consisting of one or more columns of the table. Your table's primary key uniquely identifies each row in a table, and Spanner uses the primary key to sort the table rows. Because Spanner is highly distributed, it's important that you choose a primary key generation technique that scales well with your data growth. For more information, see the [primary key migration strategies](https://docs.cloud.google.com/spanner/docs/migrating-primary-keys) that we recommend.

Note that after you designate your primary key, you can't add or remove a primary key column, or change a primary key value later without deleting and recreating the table. For more information on how to designate your primary key, see [Schema and data model - primary keys](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#primary_keys) .

### Indexes

PostgreSQL [b-tree indexes](https://www.postgresql.org/docs/10/static/indexes-types.html) are similar to [secondary indexes](https://docs.cloud.google.com/spanner/docs/secondary-indexes) in Spanner. In a Spanner database you use secondary indexes to index commonly searched columns for better performance, and to replace any `UNIQUE` constraints specified in your tables. For example, if your PostgreSQL DDL has this statement:

``` 
     CREATE TABLE customer (
        id CHAR (5) PRIMARY KEY,
        first_name VARCHAR (50),
        last_name VARCHAR (50),
        email VARCHAR (50) UNIQUE
     );

You can use the following statement in your Spanner DDL:

    CREATE TABLE customer (
       id VARCHAR(5) PRIMARY KEY,
       first_name VARCHAR(50),
       last_name VARCHAR(50),
       email VARCHAR(50)
       );

    CREATE UNIQUE INDEX customer_emails ON customer(email);
```

You can find the indexes for any of your PostgreSQL tables by running the [`\di`](https://www.postgresql.org/docs/10/static/app-psql.html) meta-command in `psql` .

After you determine the indexes that you need, add [`CREATE INDEX`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language#create_index) statements to create them. Follow the guidance at [Secondary indexes](https://docs.cloud.google.com/spanner/docs/secondary-indexes) .

Spanner implements indexes as tables, so indexing monotonically increasing columns (like those containing `TIMESTAMP` data) can cause a hotspot. See [What DBAs need to know about Spanner, part 1: Keys and indexes](https://cloudplatform.googleblog.com/2018/06/What-DBAs-need-to-know-about-Cloud-Spanner-part-1-Keys-and-indexes.html) for more information on methods to avoid hotspots.

Spanner implements secondary indexes in the same way as tables, so the column values to be used as index keys will have the same constraints as the primary keys of tables. This also means that indexes have the same consistency characteristics as Spanner tables.

Value lookups using secondary indexes are effectively the same as a query with a table join. You can improve the performance of queries using indexes by storing copies of the original table's column values in the secondary index using the `INCLUDE` clause, making it a [covering index](https://wikipedia.org/wiki/Database_index#Covering_index) .

Spanner's query optimizer is more likely to use a secondary index when the index itself stores all the columns being queried (a covered query). To force the use of an index when querying columns that are not stored in the index, you must use a [FORCE INDEX directive](https://docs.cloud.google.com/spanner/docs/secondary-indexes#index_directive) in the SQL statement, for example:

    SELECT *
    FROM MyTable /*@ FORCE_INDEX=MyTableIndex */
    WHERE IndexedColumn=$1;

Here is an example DDL statement creating a secondary index for the Albums table:

    CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle);

If you create additional indexes after your data is loaded, populating the index might take some time. We recommend that you limit the rate at which you add them to an average of three per day. For more guidance on creating secondary indexes, see [Secondary indexes](https://docs.cloud.google.com/spanner/docs/secondary-indexes) . For more information on the limitations on index creation, see [Schema updates](https://docs.cloud.google.com/spanner/docs/schema-updates#large-updates) .

### Views

Spanner views are read-only. They can't be used to insert, update, or delete data. For more information, see [Views](https://docs.cloud.google.com/spanner/docs/views) .

### Generated columns

Spanner supports generated columns. See [Create and manage generated columns](https://docs.cloud.google.com/spanner/docs/generated-column/how-to) for syntax differences and restrictions.

### Table interleaving

Spanner has a feature where you can define two tables as having a 1-many, [parent-child relationship](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#parent-child_table_relationships) . This feature interleaves the child data rows next to their parent row in storage, effectively pre-joining the table and improving data retrieval efficiency when the parent and children are queried together.

The child table's primary key must start with the primary key column(s) of the parent table. From the child row's perspective, the parent row primary key is referred to as a foreign key. You can define up to 6 levels of parent-child relationships.

You can define `ON DELETE` actions for child tables to determine what happens when the parent row is deleted: either all child rows are deleted, or the parent row deletion is blocked while child rows exist.

Here is an example of creating an Albums table interleaved in the parent Singers table defined earlier:

    CREATE TABLE Albums (
     SingerID      bigint,
     AlbumID       bigint,
     AlbumTitle    varchar,
     PRIMARY KEY (SingerID, AlbumID)
     )
     INTERLEAVE IN PARENT Singers ON DELETE CASCADE;

For more information, see [Create interleaved tables](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#create-interleaved-tables) .

### Data types

The following table lists the open source PostgreSQL data types that the PostgreSQL interface for Spanner doesn't support.

**Note:** The Spanner migration tool can automatically make some of the conversions that are listed in the **Use instead** column. For more information, see [Spanner migration tool evaluation and migration](https://github.com/GoogleCloudPlatform/spanner-migration-tool) .

| Data type                                   | Use instead                                                 |
| ------------------------------------------- | ----------------------------------------------------------- |
| bigserial,serial8                           | bigint, int8                                                |
| bit \[ (n) \]                               | \-                                                          |
| bit varying \[ (n) \], varbit \[ (n) \]     | \-                                                          |
| box                                         | \-                                                          |
| character \[ (n) \], char \[ (n) \]         | character varying                                           |
| cidr                                        | text                                                        |
| circle                                      | \-                                                          |
| inet                                        | text                                                        |
| integer, int4                               | bigint, int8                                                |
| interval \[fields\] \[ (p) \]               | bigint                                                      |
| json                                        | jsonb                                                       |
| line                                        | \-                                                          |
| lseg                                        | \-                                                          |
| macaddr                                     | text                                                        |
| money                                       | numeric, decimal                                            |
| path                                        | \-                                                          |
| pg\_lsn                                     | \-                                                          |
| point                                       | \-                                                          |
| polygon                                     | \-                                                          |
| realfloat4                                  | double precision, float8                                    |
| smallint, int2                              | bigint, int8                                                |
| smallserial, serial2                        | bigint, int8                                                |
| serial, serial4                             | bigint, int8                                                |
| time \[ (p) \] \[ without time zone \]      | text, using HH:MM:SS.sss notation                           |
| time \[ (p) \] with time zonetimetz         | text, using HH:MM:SS.sss+ZZZZ notation. Or use two columns. |
| timestamp \[ (p) \] \[ without time zone \] | text or timestamptz                                         |
| tsquery                                     | \-                                                          |
| tsvector                                    | \-                                                          |
| txid\_snapshot                              | \-                                                          |
| uuid                                        | text or bytea                                               |
| xml                                         | text                                                        |

## Step 2: Translate any SQL queries

Spanner has many of the open source PostgreSQL [functions](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions-and-operators) available to help reduce the conversion burden.

SQL queries can be profiled using the Spanner Studio page in the Google Cloud console to execute the query. In general, queries that perform full table scans on large tables are very expensive, and should be used sparingly. For more information on optimizing SQL queries, see the [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) documentation.

## Step 3: Create the Spanner instance, database, and schema

Create the instance and create a database in the PostgreSQL dialect. Then create your schema using the PostgreSQL data definition language (DDL).

Use [`pg_dump`](https://www.postgresql.org/docs/current/static/app-pgdump.html) to create DDL statements that define the objects in your PostgreSQL database, and then modify the statements as described in the preceding sections. After you update the DDL statements, use the DDL statements to create your database in the Spanner instance.

For more information, see:

  - [Create and manage instances](https://docs.cloud.google.com/spanner/docs/create-manage-instances)
  - [Create and manage databases](https://docs.cloud.google.com/spanner/docs/create-manage-databases)
  - [About schemas](https://docs.cloud.google.com/spanner/docs/schema-and-data-model)

## Step 4: Refactor the application

Add application logic to account for the modified schema and revised SQL queries, and to replace database-resident logic such as procedures and triggers.

## Step 5: Migrate your data

There are two ways to migrate your data:

  - By using the [Spanner migration tool](https://github.com/GoogleCloudPlatform/spanner-migration-tool) .
    
    The Spanner migration tool supports both schema and data migration. You can import a [pg\_dump](https://www.postgresql.org/docs/current/app-pgdump.html) file or a CSV file, or you can import data using a direct connection to the open source PostgreSQL database.

  - By using the `COPY FROM STDIN` command.
    
    For details, see [COPY command for importing data](https://docs.cloud.google.com/spanner/docs/psql-commands#copy-command) .
