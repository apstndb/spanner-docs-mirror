This page discusses Spanner schema requirements, how to use the schema to create hierarchical relationships, and schema features. It also introduces interleaved tables, which can improve query performance when querying tables in a parent-child relationship.

A schema is a namespace that contains database objects, such as tables, views, indexes, and functions. You use schemas to organize objects, apply fine-grained access control privileges, and avoid naming collisions. You must define a schema for each database in Spanner.

You can also further segment and store rows in your database table across different geographic regions. For more information, see the [Geo-partitioning overview](/spanner/docs/geo-partitioning) .

## Strongly typed data

Data in Spanner is strongly typed. Data types include scalar and complex types, which are described in [Data types in GoogleSQL](/spanner/docs/reference/standard-sql/data-types) and [PostgreSQL data types](/spanner/docs/reference/postgresql/data-types) .

### Choose a primary key

Spanner databases can contain one or more tables. Tables are structured as rows and columns. The table schema defines one or more table columns as the table's *primary key* which uniquely identifies each row. Primary keys are always indexed for quick row lookup. If you want to update or delete existing rows in a table, then the table must have a primary key. A table with no primary key columns can only have one row. Only GoogleSQL-dialect databases can have tables without a primary key.

Often your application already has a field that's a natural fit for use as the primary key. For example, for a `  Customers  ` table, there might be an application-supplied `  CustomerId  ` that serves well as the primary key. In other cases, you might need to generate a primary key when inserting the row. This would typically be a unique integer value with no business significance (a *surrogate primary key* ).

In all cases, you should be careful not to create hotspots with the choice of your primary key. For example, if you insert records with a monotonically increasing integer as the key, you'll always insert at the end of your key space. This is undesirable because Spanner divides data among servers by key ranges, which means your inserts will be directed at a single server, creating a hotspot. There are techniques that can spread the load across multiple servers and avoid hotspots:

  - [Hash the key](/spanner/docs/schema-design#fix_hash_the_key) and store it in a column. Use the hash column (or the hash column and the unique key columns together) as the primary key.
  - [Swap the order](/spanner/docs/schema-design#fix_swap_key_order) of the columns in the primary key.
  - [Use a Universally Unique Identifier (UUID)](/spanner/docs/schema-design#uuid_primary_key) . Version 4 [UUID](https://tools.ietf.org/html/rfc4122) is recommended, because it uses random values in the high-order bits. Don't use a UUID algorithm (such as version 1 UUID) that stores the timestamp in the high order bits.
  - [Bit-reverse](/spanner/docs/schema-design#bit_reverse_primary_key) sequential values.

## Parent-child table relationships

There are two ways to define parent-child relationships in Spanner: *table interleaving* and *foreign keys* .

Spanner's table interleaving is a good choice for many parent-child relationships. With interleaving, Spanner physically colocates child rows with parent rows in storage. Co-location can significantly improve performance. For example, if you have a `  Customers  ` table and an `  Invoices  ` table, and your application frequently fetches all the invoices for a customer, you can define `  Invoices  ` as an interleaved child table of `  Customers  ` . In doing so, you're declaring a data locality relationship between two independent tables. You're telling Spanner to store one or more rows of `  Invoices  ` with one `  Customers  ` row. This parent-child relationship is enforced when interleaved with the `  INTERLEAVE IN PARENT  ` clause. `  INTERLEAVE IN  ` child tables share the same physical row interleaving characteristics, but Spanner doesn't enforce referential integrity between parent and child.

You associate a child table with a parent table by using DDL that declares the child table as interleaved in the parent, and by including the parent table primary key as the first part of the child table composite primary key.

For more information about interleaving, see [Create interleaved tables](#create-interleaved-tables) .

Foreign keys are a more general parent-child solution and address additional use cases. They are not limited to primary key columns, and tables can have multiple foreign key relationships, both as a parent in some relationships and a child in others. However, a foreign key relationship does not imply co-location of the tables in the storage layer.

Google recommends that you choose to represent parent-child relationships either as interleaved tables or as foreign keys, but not both. For more information on foreign keys and their comparison to interleaved tables, see [Foreign keys overview](/spanner/docs/foreign-keys/overview#fk-and-table-interleaving) .

### Primary keys in interleaved tables

For interleaving, every table must have a primary key. If you declare a table to be an interleaved child of another table, the table must have a composite primary key that includes all of the components of the parent's primary key, in the same order, and, typically, one or more additional child table columns.

Spanner stores rows in sorted order by primary key values, with child rows inserted between parent rows. See an illustration of interleaved rows in [Create interleaved tables](#create-interleaved-tables) later in this page.

In summary, Spanner can physically colocate rows of related tables. The [schema examples](#schema-examples) show what this physical layout looks like.

### Database splits

You can define hierarchies of interleaved parent-child relationships up to seven layers deep, which means that you can colocate rows of seven independent tables. If the size of the data in your tables is small, a single Spanner server can probably handle your database. But what happens when your related tables grow and start reaching the resource limits of an individual server? Spanner is a distributed database, which means that as your database grows, Spanner divides your data into chunks called "splits." Individual splits can move independently from each other and get assigned to different servers, which can be in different physical locations. A split holds a range of contiguous rows. The start and end keys of this range are called "split boundaries". Spanner automatically adds and removes split boundaries based on size and load, which changes the number of splits in the database.

#### Load-based splitting

As an example of how Spanner performs load-based splitting to mitigate read hotspots, suppose your database contains a table with 10 rows that are read more frequently than all of the other rows in the table. Spanner can add split boundaries between each of those 10 rows so that they're each handled by a different server, rather than allowing all the reads of those rows to consume the resources of a single server.

As a general rule, if you follow [best practices for schema design](/spanner/docs/schema-design) , Spanner can mitigate hotspots such that the read throughput should improve every few minutes until you saturate the resources in your instance or run into cases where no new split boundaries can be added (because you have a split that covers just a single row with no interleaved children).

## Named schemas

Named schemas help you organize similar data together. This helps you to quickly find objects in the Google Cloud console, apply privileges, and avoid naming collisions.

Named schemas, like other database objects, are managed using DDL.

Spanner named schemas permit you to use fully qualified names (FQNs) to query for data. FQNs let you combine the schema name and the object name to identify database objects. For example, you could create a schema called `  warehouse  ` for the warehouse business unit. The tables that use this schema could include: `  product  ` , `  order  ` , and `  customer information  ` . Or you could create a schema called `  fulfillment  ` for the fulfillment business unit. This schema could also have tables called `  product  ` , `  order  ` , and `  customer information  ` . In the first example, the FQN is `  warehouse.product  ` and in the second example, the FQN is `  fulfillment.product  ` . This prevents confusion in situations where multiple objects share the same name.

In the `  CREATE SCHEMA  ` DDL, table objects are given both an FQN, for example, `  sales.customers  ` , and a short name, for example, `  sales  ` .

The following database objects support named schemas:

  - `  TABLE  `
      - `  CREATE  `
      - `  INTERLEAVE IN [PARENT]  `
      - `  FOREIGN KEY  `
      - `  SYNONYM  `
  - `  VIEW  `
  - `  INDEX  `
  - `  SEARCH INDEX  `
  - `  FOREIGN KEY  `
  - `  SEQUENCE  `

For more information about using named schemas, see [Manage named schemas](/spanner/docs/named-schemas) .

### Use fine-grained access control with named schemas

Named schemas let you grant schema-level access to each object in the schema. This applies to schema objects that exist at the time that you grant access. You must grant access to objects that are added later.

Fine-grained access control limits access to entire groups of database objects, such as tables, columns, and rows in the table.

For more information, see [Grant fine-grained access control privileges to named schemas](/spanner/docs/named-schemas#add-fgac-to-named-schema) .

## Schema examples

The schema examples in this section show how to create parent and child tables with and without interleaving, and illustrate the corresponding physical layouts of data.

### Create a parent table

Suppose you're creating a music application and you need a table that stores rows of singer data:

Note that the table contains one primary key column, `  SingerId  ` , which appears to the left of the bolded line, and that tables are organized by rows and columns.

You can define the table with the following DDL:

### GoogleSQL

``` text
CREATE TABLE Singers (
SingerId   INT64 NOT NULL PRIMARY KEY,
FirstName  STRING(1024),
LastName   STRING(1024),
SingerInfo BYTES(MAX),
);
```

### PostgreSQL

``` text
CREATE TABLE singers (
singer_id   BIGINT PRIMARY KEY,
first_name  VARCHAR(1024),
last_name   VARCHAR(1024),
singer_info BYTEA
);
```

Note the following about the example schema:

  - `  Singers  ` is a table at the root of the database hierarchy (because it's not defined as an interleaved child of another table).
  - For GoogleSQL-dialect databases, primary key columns are usually annotated with `  NOT NULL  ` (though you can omit this annotation if you want to allow `  NULL  ` values in key columns. For more information, see [Key Columns](#notes_about_key_columns) ).
  - Columns that are not included in the primary key are called non-key columns, and they can have an optional `  NOT NULL  ` annotation.
  - Columns that use the `  STRING  ` or `  BYTES  ` type in GoogleSQL must be defined with a length, which represents the maximum number of Unicode characters that can be stored in the field. The length specification is optional for the PostgreSQL `  varchar  ` and `  character varying  ` types. For more information, see [Scalar Data Types](/spanner/docs/reference/standard-sql/data-definition-language#scalars) for GoogleSQL-dialect databases and [PostgreSQL data types](/spanner/docs/reference/postgresql/data-types) for PostgreSQL-dialect databases.

What does the physical layout of the rows in the `  Singers  ` table look like? The following diagram shows rows of the `  Singers  ` table stored by primary key ("Singers(1)", and then "Singers(2)", where the number in parentheses is the primary key value.

The preceding diagram illustrates an example split boundary between the rows keyed by `  Singers(3)  ` and `  Singers(4)  ` , with the data from the resulting splits assigned to different servers. As this table grows, it's possible for rows of `  Singers  ` data to be stored in different locations.

### Create parent and child tables

Assume that you now want to add some basic data about each singer's albums to the music application.

Note that the primary key of `  Albums  ` is composed of two columns: `  SingerId  ` and `  AlbumId  ` , to associate each album with its singer. The following example schema defines both the `  Albums  ` and `  Singers  ` tables at the root of the database hierarchy, which makes them sibling tables.

``` text
-- Schema hierarchy:
-- + Singers (sibling table of Albums)
-- + Albums (sibling table of Singers)
```

### GoogleSQL

``` text
CREATE TABLE Singers (
 SingerId   INT64 NOT NULL PRIMARY KEY,
 FirstName  STRING(1024),
 LastName   STRING(1024),
 SingerInfo BYTES(MAX),
);

CREATE TABLE Albums (
SingerId     INT64 NOT NULL,
AlbumId      INT64 NOT NULL,
AlbumTitle   STRING(MAX),
) PRIMARY KEY (SingerId, AlbumId);
```

### PostgreSQL

``` text
CREATE TABLE singers (
singer_id   BIGINT PRIMARY KEY,
first_name  VARCHAR(1024),
last_name   VARCHAR(1024),
singer_info BYTEA
);

CREATE TABLE albums (
singer_id     BIGINT,
album_id      BIGINT,
album_title   VARCHAR,
PRIMARY KEY (singer_id, album_id)
);
```

The physical layout of the rows of `  Singers  ` and `  Albums  ` looks like the following diagram, with rows of the `  Albums  ` table stored by contiguous primary key, then rows of `  Singers  ` stored by contiguous primary key:

One important note about the schema is that Spanner assumes no data locality relationships between the `  Singers  ` and `  Albums  ` tables, because they are top-level tables. As the database grows, Spanner can add split boundaries between any of the rows. This means the rows of the `  Albums  ` table could end up in a different split from the rows of the `  Singers  ` table, and the two splits could move independently from each other.

Depending on your application's needs, it might be fine to allow `  Albums  ` data to be located on different splits from `  Singers  ` data. However, this might incur a performance penalty due to the need to coordinate reads and updates across distinct resources. If your application frequently needs to retrieve information about all the albums for a particular singer, then you should create `  Albums  ` as an interleaved child table of `  Singers  ` , which colocates rows from the two tables along the primary key dimension. The next example explains this in more detail.

### Create interleaved tables

An *interleaved table* is a table that you declare to be an interleaved child of another table because you want the rows of the child table to be physically stored with the associated parent row. As mentioned earlier, the parent table primary key must be the first part of the child table composite primary key.

After you interleave a table, it's permanent. You can't undo the interleaving. Instead, you need to create the table again and migrate data to it.

As you're designing your music application, suppose you realize that the app needs to frequently access rows from the `  Albums  ` table when it accesses a `  Singers  ` row. For example, when you access the row `  Singers(1)  ` , you also need to access the rows `  Albums(1, 1)  ` and `  Albums(1, 2)  ` . In this case, `  Singers  ` and `  Albums  ` need to have a strong data locality relationship. You can declare this data locality relationship by creating `  Albums  ` as an interleaved child table of `  Singers  ` .

``` text
-- Schema hierarchy:
-- + Singers
--   + Albums (interleaved table, child table of Singers)
```

The bolded line in the following schema shows how to create `  Albums  ` as an interleaved table of `  Singers  ` .

### GoogleSQL

``` text
CREATE TABLE Singers (
 SingerId   INT64 NOT NULL PRIMARY KEY,
 FirstName  STRING(1024),
 LastName   STRING(1024),
 SingerInfo BYTES(MAX),
 );

CREATE TABLE Albums (
 SingerId     INT64 NOT NULL,
 AlbumId      INT64 NOT NULL,
 AlbumTitle   STRING(MAX),
 ) PRIMARY KEY (SingerId, AlbumId),
INTERLEAVE IN PARENT Singers ON DELETE CASCADE;
```

### PostgreSQL

``` text
CREATE TABLE singers (
 singer_id   BIGINT PRIMARY KEY,
 first_name  VARCHAR(1024),
 last_name   VARCHAR(1024),
 singer_info BYTEA
 );

CREATE TABLE albums (
 singer_id     BIGINT,
 album_id      BIGINT,
 album_title   VARCHAR,
 PRIMARY KEY (singer_id, album_id)
 )
 INTERLEAVE IN PARENT singers ON DELETE CASCADE;
```

Notes about this schema:

  - `  SingerId  ` , which is the first part of the primary key of the child table `  Albums  ` , is also the primary key of its parent table `  Singers  ` .
  - The [`  ON DELETE CASCADE  `](/spanner/docs/reference/standard-sql/data-definition-language#create_table) annotation signifies that when a row from the parent table is deleted, its child rows are automatically deleted as well. If a child table doesn't have this annotation, or the annotation is `  ON DELETE NO ACTION  ` , then you must delete the child rows before you can delete the parent row.
  - Interleaved rows are ordered first by rows of the parent table, then by contiguous rows of the child table that share the parent's primary key. For example, "Singers(1)", then "Albums(1, 1)", and then "Albums(1, 2)".
  - The data locality relationship of each singer and their album data is preserved if this database splits, provided that the size of a `  Singers  ` row and all its `  Albums  ` rows stays below the split size limit and that there is no hotspot in any of these `  Albums  ` rows.
  - The parent row must exist before you can insert child rows. The parent row can either already exist in the database or can be inserted before the insertion of the child rows in the same transaction.

Suppose you'd like to model `  Projects  ` and their `  Resources  ` as interleaved tables. Certain scenarios could benefit from `  INTERLEAVE IN  ` - the ability to not require the `  Projects  ` row to exist for entities under it to exist (say, a Project has been deleted, but its resources need to be cleaned up before deleting).

### GoogleSQL

``` text
CREATE TABLE Projects (
  ProjectId   INT64 NOT NULL,
  ProjectName STRING(1024),
) PRIMARY KEY (ProjectId);

CREATE TABLE Resources (
  ProjectId    INT64 NOT NULL,
  ResourceId   INT64 NOT NULL,
  ResourceName STRING(1024),
) PRIMARY KEY (ProjectId, ResourceId),
  INTERLEAVE IN Projects;
```

### PostgreSQL

``` text
CREATE TABLE Projects (
  ProjectId   BIGINT PRIMARY KEY,
  ProjectName VARCHAR(1024),
);

CREATE TABLE Resources (
  ProjectId    BIGINT,
  ResourceId   BIGINT,
  ResourceName VARCHAR(1024),
  PRIMARY KEY (ProjectId, ResourceId)
) INTERLEAVE IN Projects;
```

Note that in this example we use the `  INTERLEAVE IN Projects  ` clause, rather than `  INTERLEAVE IN PARENT Projects  ` . This indicates we don't enforce the parent-child relationship between Projects and Resources.

In this example, the `  Resources(1, 10)  ` and `  Resources(1, 20)  ` rows can exist in the database even if the `  Projects(1)  ` row doesn't exist. `  Projects(1)  ` can be deleted even if `  Resources(1, 10)  ` and `  Resources(1, 20)  ` still exist, and the deletion doesn't affect these `  Resources  ` rows.

### Create a hierarchy of interleaved tables

The parent-child relationship between `  Singers  ` and `  Albums  ` can be extended to more descendant tables. For example, you could create an interleaved table called `  Songs  ` as a child of `  Albums  ` to store the track list of each album:

`  Songs  ` must have a primary key that includes all the primary keys of the tables that are at a higher level in the hierarchy, that is, `  SingerId  ` and `  AlbumId  ` .

``` text
-- Schema hierarchy:
-- + Singers
--   + Albums (interleaved table, child table of Singers)
--     + Songs (interleaved table, child table of Albums)
```

### GoogleSQL

``` text
CREATE TABLE Singers (
 SingerId   INT64 NOT NULL PRIMARY KEY,
 FirstName  STRING(1024),
 LastName   STRING(1024),
 SingerInfo BYTES(MAX),
);

CREATE TABLE Albums (
 SingerId     INT64 NOT NULL,
 AlbumId      INT64 NOT NULL,
 AlbumTitle   STRING(MAX),
) PRIMARY KEY (SingerId, AlbumId),
 INTERLEAVE IN PARENT Singers ON DELETE CASCADE;

CREATE TABLE Songs (
 SingerId     INT64 NOT NULL,
 AlbumId      INT64 NOT NULL,
 TrackId      INT64 NOT NULL,
 SongName     STRING(MAX),
) PRIMARY KEY (SingerId, AlbumId, TrackId),
 INTERLEAVE IN PARENT Albums ON DELETE CASCADE;
```

### PostgreSQL

``` text
CREATE TABLE singers (
 singer_id   BIGINT PRIMARY KEY,
 first_name  VARCHAR(1024),
 last_name   VARCHAR(1024),
 singer_info BYTEA
 );

CREATE TABLE albums (
 singer_id     BIGINT,
 album_id      BIGINT,
 album_title   VARCHAR,
 PRIMARY KEY (singer_id, album_id)
 )
 INTERLEAVE IN PARENT singers ON DELETE CASCADE;

CREATE TABLE songs (
 singer_id     BIGINT,
 album_id      BIGINT,
 track_id      BIGINT,
 song_name     VARCHAR,
 PRIMARY KEY (singer_id, album_id, track_id)
 )
 INTERLEAVE IN PARENT albums ON DELETE CASCADE;
```

The following diagram represents a physical view of interleaved rows.

In this example, as the number of singers grows, Spanner adds split boundaries between singers to preserve data locality between a singer and its album and song data. However, if the size of a singer row and its child rows exceeds the split size limit, or a hotspot is detected in the child rows, Spanner attempts to add split boundaries to isolate that hotspot row along with all child rows below it.

In summary, a parent table along with all of its child and descendant tables forms a hierarchy of tables in the schema. Although each table in the hierarchy is logically independent, physically interleaving them this way can improve performance, effectively pre-joining the tables and allowing you to access related rows together while minimizing storage accesses.

### Joins with interleaved tables

If possible, join data in interleaved tables by primary key. Because each interleaved row is usually stored physically in the same split as its parent row, Spanner can perform joins by primary key locally, minimizing storage access and network traffic. In the following example, `  Singers  ` and `  Albums  ` are joined on the primary key `  SingerId  ` .

### GoogleSQL

``` text
SELECT s.FirstName, a.AlbumTitle
FROM Singers AS s JOIN Albums AS a ON s.SingerId = a.SingerId;
```

### PostgreSQL

``` text
SELECT s.first_name, a.album_title
FROM singers AS s JOIN albums AS a ON s.singer_id = a.singer_id;
```

### Locality groups

**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

Spanner uses locality groups to preserve data locality relationships across table columns. If you don't explicitly create any locality groups for your tables, Spanner groups all columns into the `  default  ` locality group and stores the data of all tables on SSD storage. You can use locality groups to do the following:

  - Use [tiered storage](/spanner/docs/tiered-storage) . Tiered storage is a fully-managed storage feature that lets you choose whether to store your data on solid-state drives (SSD) or hard disk drives (HDD). By default, without using tiered storage, Spanner stores all data on SSD storage.

  - Use column-grouping to store specified columns separately from other columns. Because the data for the specified columns are stored separately, reading data from those columns is faster than if all data is grouped together. To use column-grouping, you need to [create a locality group](/spanner/docs/create-manage-locality-groups#create-locality-group) without specifying any tiered storage options. Spanner uses locality groups to store the specified columns separately. If specified, the columns inherit their tiered storage policy from the table or default locality group. Then, use the `  CREATE TABLE  ` DDL statement to [set a locality group for the specified columns](/spanner/docs/create-manage-locality-groups#set-locality-group-column) or use the `  ALTER TABLE  ` DDL statement to [alter the locality group used by a table's column](/spanner/docs/create-manage-locality-groups#alter-column-locality-group) . The DDL statement determines the columns that are stored in the locality group. Finally, you can [read data](/spanner/docs/reads) in these columns more efficiently.

## Key columns

This section includes some notes about key columns.

### Change table keys

The keys of a table can't change; you can't add a key column to an existing table or remove a key column from an existing table.

### Store NULLs in a primary key

In GoogleSQL, if you would like to store NULL in a primary key column, omit the `  NOT NULL  ` clause for that column in the schema. (PostgreSQL-dialect databases don't support NULLs in a primary key column.)

Here's an example of omitting the `  NOT NULL  ` clause on the primary key column `  SingerId  ` . Note that because `  SingerId  ` is the primary key, there can be only one row that stores `  NULL  ` in that column.

``` text
CREATE TABLE Singers (
  SingerId   INT64 PRIMARY KEY,
  FirstName  STRING(1024),
  LastName   STRING(1024),
);
```

The nullable property of the primary key column must match between the parent and the child table declarations. In this example, `  NOT NULL  ` for the column `  Albums.SingerId  ` is not allowed because `  Singers.SingerId  ` omits it.

``` text
CREATE TABLE Singers (
  SingerId   INT64 PRIMARY KEY,
  FirstName  STRING(1024),
  LastName   STRING(1024),
);

CREATE TABLE Albums (
  SingerId     INT64 NOT NULL,
  AlbumId      INT64 NOT NULL,
  AlbumTitle   STRING(MAX),
) PRIMARY KEY (SingerId, AlbumId),
  INTERLEAVE IN PARENT Singers ON DELETE CASCADE;
```

### Disallowed types

The following columns cannot be of type `  ARRAY  ` :

  - A table's key columns.
  - An index's key columns.

## Design for multi-tenancy

You might want to implement multi-tenancy if you are storing data that belongs to different customers. For example, a music service might want to store each individual record label's content separately.

### Classic multi-tenancy

The classic way to design for multi-tenancy is to create a separate database for each customer. In this example, each database has its own `  Singers  ` table:

<table>
<caption> Database 1: Ackworth Records </caption>
<thead>
<tr class="header">
<th>SingerId</th>
<th>FirstName</th>
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>Marc</td>
<td>Richards</td>
</tr>
<tr class="even">
<td>2</td>
<td>Catalina</td>
<td>Smith</td>
</tr>
</tbody>
</table>

<table>
<caption> Database 2: Cama Records </caption>
<thead>
<tr class="header">
<th>SingerId</th>
<th>FirstName</th>
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>Alice</td>
<td>Trentor</td>
</tr>
<tr class="even">
<td>2</td>
<td>Gabriel</td>
<td>Wright</td>
</tr>
</tbody>
</table>

<table>
<caption> Database 3: Eagan Records </caption>
<thead>
<tr class="header">
<th>SingerId</th>
<th>FirstName</th>
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>Benjamin</td>
<td>Martinez</td>
</tr>
<tr class="even">
<td>2</td>
<td>Hannah</td>
<td>Harris</td>
</tr>
</tbody>
</table>

### Schema-managed multi-tenancy

Another way to design for multi-tenancy in Spanner is to have all customers in a single table in a single database, and to use a different primary key value for each customer. For example, you could include a `  CustomerId  ` key column in your tables. If you make `  CustomerId  ` the first key column, then the data for each customer has good locality. Spanner can then effectively use [database splits](#database-splits) to maximize performance based on data size and load patterns. In the following example, there is a single `  Singers  ` table for all customers:

<table>
<caption> Spanner multi-tenancy database </caption>
<thead>
<tr class="header">
<th>CustomerId</th>
<th>SingerId</th>
<th>FirstName</th>
<th>LastName</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>1</td>
<td>Marc</td>
<td>Richards</td>
</tr>
<tr class="even">
<td>1</td>
<td>2</td>
<td>Catalina</td>
<td>Smith</td>
</tr>
<tr class="odd">
<td>2</td>
<td>1</td>
<td>Alice</td>
<td>Trentor</td>
</tr>
<tr class="even">
<td>2</td>
<td>2</td>
<td>Gabriel</td>
<td>Wright</td>
</tr>
<tr class="odd">
<td>3</td>
<td>1</td>
<td>Benjamin</td>
<td>Martinez</td>
</tr>
<tr class="even">
<td>3</td>
<td>2</td>
<td>Hannah</td>
<td>Harris</td>
</tr>
</tbody>
</table>

If you must have separate databases for each tenant, there are constraints to be aware of:

  - There are [limits](/spanner/quotas) on the number of databases per instance and the number of tables and indexes per database. Depending on the number of customers, it might not be possible to have separate databases or tables.
  - Adding new tables and non-interleaved indexes [can take a long time](/spanner/docs/schema-updates#schema_update_performance) . You might not be able to get the performance you want if your schema design depends on adding new tables and indexes.

If you want to create separate databases, you might have more success if you distribute your tables across databases in such a way that each database has a [low number of schema changes per week](/spanner/docs/schema-updates#frequency) .

If you create separate tables and indexes for each customer of your application, don't put all of the tables and indexes in the same database. Instead, split them across many databases, to mitigate the [performance issues](/spanner/docs/schema-updates#large-updates) with creating a large number of indexes.

To learn more about other data management patterns and application design for multi-tenancy, see [Implementing Multi-Tenancy in Spanner](https://medium.com/google-cloud/implementing-multi-tenancy-in-cloud-spanner-3afe19605d8e)
