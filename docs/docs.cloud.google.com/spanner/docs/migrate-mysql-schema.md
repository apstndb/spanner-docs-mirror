This page shows how to migrate your MySQL schema to Spanner schema. We recommend using the [Spanner migration tool](https://googlecloudplatform.github.io/spanner-migration-tool/ui/schema-conv) for building a Spanner schema from an existing MySQL schema. The tool maps most of the MySQL data types to Spanner types, and highlight choices and provide suggestions to avoid potential migration issues.

## Data type comparison

Map the following list of MySQL data types to their Spanner equivalent:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>MySQL data type</th>
<th>Spanner equivalent</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INTEGER      </code> , <code dir="ltr" translate="no">       INT      </code> , <code dir="ltr" translate="no">       BIGINT      </code> , <code dir="ltr" translate="no">       MEDIUMINT      </code> , <code dir="ltr" translate="no">       SMALLINT      </code> , <code dir="ltr" translate="no">       TINYINT      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       TINYINT      </code> , <code dir="ltr" translate="no">       BOOL      </code> , <code dir="ltr" translate="no">       BOOLEAN      </code> ,</td>
<td><code dir="ltr" translate="no">       BOOLEAN      </code></td>
<td><code dir="ltr" translate="no">       TINYINT(1)      </code> values are used to represent boolean values of 'true' (nonzero) or 'false' (0).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       BIT      </code></td>
<td><code dir="ltr" translate="no">       BOOLEAN      </code> , <code dir="ltr" translate="no">       INT64      </code></td>
<td></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       CHAR      </code> , <code dir="ltr" translate="no">       VARCHAR      </code> , <code dir="ltr" translate="no">       TINYTEXT      </code> , <code dir="ltr" translate="no">       TEXT      </code> , <code dir="ltr" translate="no">       MEDIUMTEXT      </code> , <code dir="ltr" translate="no">       LONGTEXT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Spanner uses Unicode UTF8 strings throughout and doesn't have configurable collations.<br />
<code dir="ltr" translate="no">       VARCHAR      </code> supports a maximum length of 65,535 bytes, while Spanner supports up to 2,621,440 characters.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       FLOAT      </code></td>
<td><code dir="ltr" translate="no">       FLOAT32      </code></td>
<td></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       DOUBLE      </code></td>
<td><code dir="ltr" translate="no">       FLOAT64      </code></td>
<td></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       DECIMAL      </code> , <code dir="ltr" translate="no">       NUMERIC      </code></td>
<td><code dir="ltr" translate="no">       NUMERIC      </code></td>
<td>In MySQL, the <code dir="ltr" translate="no">       NUMERIC      </code> and <code dir="ltr" translate="no">       DECIMAL      </code> data types support up to a total 65 digits of precision and scale, as defined in the column declaration. The Spanner <code dir="ltr" translate="no">       NUMERIC      </code> data type supports up to 38 digits of precision and 9 decimal digits of scale.<br />
If you require greater precision, see <a href="/spanner/docs/storing-numeric-data">Store arbitrary precision numeric data</a> for alternative mechanisms.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       BINARY      </code> , <code dir="ltr" translate="no">       VARBINARY      </code> , <code dir="ltr" translate="no">       TINYBLOB      </code> , <code dir="ltr" translate="no">       BLOB      </code> , <code dir="ltr" translate="no">       MEDIUMBLOB      </code> , <code dir="ltr" translate="no">       LONGBLOB      </code></td>
<td><code dir="ltr" translate="no">       BYTES      </code></td>
<td>Small objects (less than 10 MiB) can be stored as <code dir="ltr" translate="no">       BYTES      </code> . Consider using alternative Google Cloud offerings such as Cloud Storage to store larger objects.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       DATE      </code></td>
<td><code dir="ltr" translate="no">       DATE      </code></td>
<td>Both Spanner and MySQL use the ' <code dir="ltr" translate="no">       yyyy-mm-dd      </code> ' format for dates, so no transformation is necessary. SQL functions are provided to convert dates to a formatted string.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       DATETIME      </code> , <code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td><code dir="ltr" translate="no">       TIMESTAMP      </code></td>
<td>Spanner stores time independent of time zone. If you need to store a time zone, you must use a separate <code dir="ltr" translate="no">       STRING      </code> column. SQL functions are provided to convert timestamps to a formatted string using time zones.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TEXT      </code> , <code dir="ltr" translate="no">       TINYTEXT      </code> , <code dir="ltr" translate="no">       ENUM      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Small <code dir="ltr" translate="no">       TEXT      </code> values (less than 10 MiB) can be stored as <code dir="ltr" translate="no">       STRING      </code> . Consider using alternative Google Cloud offerings such as Cloud Storage to support larger <code dir="ltr" translate="no">       TEXT      </code> values.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ENUM      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Validation of <code dir="ltr" translate="no">       ENUM      </code> values must be performed in the application.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SET      </code></td>
<td><code dir="ltr" translate="no">       ARRAY&lt;STRING&gt;      </code></td>
<td>Validation of <code dir="ltr" translate="no">       SET      </code> element values must be performed in the application.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       LONGBLOB      </code> , <code dir="ltr" translate="no">       MEDIUMBLOB      </code></td>
<td><code dir="ltr" translate="no">       BYTES      </code> or <code dir="ltr" translate="no">       STRING      </code> containing URI to object.</td>
<td>Small objects (less than 10 MiB) can be stored as <code dir="ltr" translate="no">       BYTES      </code> . Consider using alternative Google Cloud offerings such as Cloud Storage to store larger objects.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       LONGTEXT      </code> , <code dir="ltr" translate="no">       MEDIUMTEXT      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code> (either containing data or URI to external object)</td>
<td>Small objects (less than 2,621,440 characters) can be stored as <code dir="ltr" translate="no">       STRING      </code> . Consider using alternative Google Cloud offerings such as Cloud Storage to store larger objects.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       JSON      </code></td>
<td><code dir="ltr" translate="no">       JSON      </code></td>
<td>Small JSON strings (less than 2,621,440 characters) can be stored as <code dir="ltr" translate="no">       JSON      </code> . Consider using alternative Google Cloud offerings such as Cloud Storage to store larger objects.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       GEOMETRY      </code> , <code dir="ltr" translate="no">       POINT      </code> , <code dir="ltr" translate="no">       LINESTRING      </code> , <code dir="ltr" translate="no">       POLYGON      </code> , <code dir="ltr" translate="no">       MULTIPOINT      </code> , <code dir="ltr" translate="no">       MULTIPOLYGON      </code> , <code dir="ltr" translate="no">       GEOMETRYCOLLECTION      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code> , <code dir="ltr" translate="no">       ARRAY      </code></td>
<td>Spanner doesn't support geospatial data types. You must store this data using standard data types, and implement any searching or filtering logic in the application.</td>
</tr>
</tbody>
</table>

In many cases, multiple MySQL types map into a single Spanner type. This is because MySQL has a set of types for the same concept that have different length limits, and in Spanner there is one overall type that has a single, relatively large limit.

Consider the following examples:

  - MySQL has `  TEXT  ` , `  TINYTEXT  ` , `  MEDIUMTEXT  ` , `  LONGTEXT  ` . In Spanner, there is a single type `  STRING  ` with a character-length parameter that can be set to any value up to 2,621,440 characters.

  - MySQL has `  INTEGER  ` , `  INT  ` , `  BIGINT  ` , `  MEDIUMINT  ` , `  SMALLINT  ` and `  TINYINT  ` . Spanner has a single type `  INT64  ` that stores 8-byte signed integer values. The main difference is that Spanner's `  INT64  ` consumes more storage than `  MEDIUMINT  ` , `  SMALLINT  ` and `  TINYINT  ` . In addition, `  INT64  ` doesn't capture the range limitations of `  MEDIUMINT  ` , `  SMALLINT  ` and `  TINYINT  ` , although these can be enforced by adding `  CHECK  ` constraints.

Spanner doesn't support geospatial types. You can store values of these types by encoding them as strings, bytes, or arrays. Any filtering, operations, and functions must be performed at the application level.

## Queries

Spanner uses the [ANSI 2011 dialect of SQL with extensions](/spanner/docs/query-syntax) , and has many functions and operators to help translate and aggregate your data. Any SQL queries using MySQL-specific dialect, functions, and types need to be converted to be compatible with Spanner.

Although Spanner doesn't support structured data as column definitions, you can use structured data in SQL queries using `  ARRAY<>  ` and `  STRUCT<>  ` types. For example, you can write a query that returns all Albums for an artist using an `  ARRAY  ` of `  STRUCT  ` s (taking advantage of the pre-joined data). For more information see the [Subqueries](/spanner/docs/query-syntax#subqueries) section of the documentation.

You can run SQL queries on the Spanner Studio page in the Google Cloud console. In general, queries that perform full table scans on large tables are very expensive, and should be used sparingly. For more information on optimizing SQL queries, see the [SQL best practices](/spanner/docs/sql-best-practices) documentation.

## Stored procedures and triggers

Spanner doesn't support running user code at the database level. As part of the schema migration, move the stored procedures and business logic triggers that you implemented at the MySQL database-level into your application.

## Sequences

Spanner recommends using UUID Version 4 as the default method to generate primary key values. The [`  GENERATE_UUID()  `](/spanner/docs/reference/standard-sql/utility-functions#generate_uuid) function returns UUID Version 4 values represented as `  STRING  ` type.

If you need to generate integer values, Spanner supports [bit-reversed positive sequences](/spanner/docs/reference/standard-sql/data-definition-language#create-sequence) , which produce values that distribute evenly across the positive 64-bit number space. You can use these numbers to avoid hot spotting issues.

For more information, see [primary key default value strategies](/spanner/docs/primary-key-default-value) .

## What's next

  - [Use SMT to migrate schema from MySQL](/spanner/docs/use-smt-migrate-mysql-schema) .
