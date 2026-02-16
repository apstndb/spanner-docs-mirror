This page contains information about the PostgreSQL system catalog tables available in Spanner.

[PostgreSQL system catalog tables](https://www.postgresql.org/docs/current/catalogs.html) store schema data, such as information about tables and columns, and internal bookkeeping information. These system catalog tables are part of a namespace called `  pg_catalog  ` .

## Limitations and differences

This section lists the columns that are missing and other differences between the Spanner implementation of PostgreSQL system catalog tables and PostgreSQL.

### Unsupported columns

Spanner doesn't support every column in its implementation of PostgreSQL system catalog tables. The following columns aren't supported and therefore don't show up in the table.

<table>
<thead>
<tr class="header">
<th>Catalog table name</th>
<th>Missing column names</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       pg_am      </code></td>
<td><code dir="ltr" translate="no">       amhandler      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       pg_attribute      </code></td>
<td><code dir="ltr" translate="no">       attacl      </code> , <code dir="ltr" translate="no">       attmissingval      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pg_class      </code></td>
<td><code dir="ltr" translate="no">       relacl      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       pg_namespace      </code></td>
<td><code dir="ltr" translate="no">       nspacl      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pg_proc      </code></td>
<td><code dir="ltr" translate="no">       prosupport      </code> , <code dir="ltr" translate="no">       proacl      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       pg_sequences      </code></td>
<td><code dir="ltr" translate="no">       data_type      </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pg_type      </code></td>
<td><code dir="ltr" translate="no">       typsubscript      </code> , <code dir="ltr" translate="no">       typinput      </code> , <code dir="ltr" translate="no">       typoutput      </code> , <code dir="ltr" translate="no">       typreceive      </code> , <code dir="ltr" translate="no">       typsend      </code> , <code dir="ltr" translate="no">       typmodin      </code> , <code dir="ltr" translate="no">       typmodout      </code> , <code dir="ltr" translate="no">       typanalyze      </code> , <code dir="ltr" translate="no">       typacl      </code></td>
</tr>
</tbody>
</table>

### Empty columns

Some columns in supported system catalog tables don't contain data. These columns could have content added later so avoid queries that rely on these columns being empty. For each table listed on this page, supported columns are listed, along with information about whether or not the column has content.

### Empty tables

Some tables and views have no data in them. This is so that queries can refer to them without encountering errors. This improves compatibility with tools without requiring rewrites to avoid those tables.

### Difference from PostgreSQL

In PostgreSQL, you can write to these tables directly to modify the database. In Spanner, system tables are read-only.

The Spanner implementation contains the following changes to PostgreSQL system catalog columns:

  - `  pg_node_tree  ` columns (most notably `  pg_attrdef.adbin  ` and `  pg_proc.prosqlbody  ` ) are text columns that contain the SQL expressions.

  - `  oidvector  ` columns (most notably `  pg_proc.proargtypes  ` ) are `  oid[]  ` columns.

  - `  int2vector  ` columns are `  int8[]  ` columns.

  - `  pg_class.relnatts  ` column is `  NULL  ` for sequences because you can't select them in Spanner.

  - `  pg_collation  ` only contains default and C. General collation support isn't available.

## PostgreSQL system catalog tables list

This section contains the PostgreSQL system catalog tables that Spanner supports.

### `     pg_am    `

The [`  pg_am  `](https://www.postgresql.org/docs/current/catalog-pg-am.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       amname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       amtype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
</tbody>
</table>

### `     pg_attrdef    `

The [`  pg_attrdef  `](https://www.postgresql.org/docs/current/catalog-pg-attrdef.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       adrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       adnum      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       adbin      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
</tbody>
</table>

The `  adbin  ` column should contain a `  text  ` value that represents the serialized value of the default value.

### `     pg_attribute    `

The [`  pg_attribute  `](https://www.postgresql.org/docs/current/catalog-pg-attribute.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       attrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       atttypid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attstattarget      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attlen      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attnum      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attndims      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attcacheoff      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       atttypmod      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attbyval      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attalign      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attstorage      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attcompression      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attnotnull      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       atthasdef      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       atthasmissing      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attidentity      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attgenerated      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attisdropped      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attislocal      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attinhcount      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attcollation      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       attoptions      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attfdwoptions      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_class    `

The [`  pg_class  `](https://www.postgresql.org/docs/current/catalog-pg-class.html)

has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relnamespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       reltype      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       reloftype      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relowner      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relam      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relfilenode      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       reltablespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relpages      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       reltuples      </code></td>
<td><code dir="ltr" translate="no">       float8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relallvisible      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       reltoastrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relhasindex      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relisshared      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relpersistence      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relkind      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relnatts      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relchecks      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relhasrules      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relhastriggers      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relhassubclass      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relrowsecurity      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relforcerowsecurity      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relispopulated      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relreplident      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relispartition      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relrewrite      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       relfrozenxid      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relminmxid      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       reloptions      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relpartbound      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_collation    `

The [`  pg_collation  `](https://www.postgresql.org/docs/current/catalog-pg-collation.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collnamespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collowner      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collprovider      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collisdeterministic      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collencoding      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collcollate      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collctype      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       colliculocale      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collversion      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_constraint    `

The [`  pg_constraint  `](https://www.postgresql.org/docs/current/catalog-pg-constraint.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       connamespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       contype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       condeferrable      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       condeferred      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       convalidated      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       contypid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conindid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       conparentid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       confrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       confupdtype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       confdeltype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       confmatchtype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conislocal      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       coninhcount      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       connoinherit      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       conkey      </code></td>
<td><code dir="ltr" translate="no">       int8[]      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       confkey      </code></td>
<td><code dir="ltr" translate="no">       int8[]      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       conpfeqop      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conppeqop      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       conffeqop      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conexclop      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       confdelsetcols      </code></td>
<td><code dir="ltr" translate="no">       int8[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       conbin      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_description    `

The [`  pg_description  `](https://www.postgresql.org/docs/current/catalog-pg-description.html) table has no content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       objoid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       classoid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       objsubid      </code></td>
<td><code dir="ltr" translate="no">       int64      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       description      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_enum    `

The [`  pg_enum  `](https://www.postgresql.org/docs/current/catalog-pg-enum.html)

table has no content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       enumtypid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collnamespace      </code></td>
<td><code dir="ltr" translate="no">       float8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       enumlabel      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_index    `

The [`  pg_index  `](https://www.postgresql.org/docs/current/catalog-pg-index.html)

table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       indexrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indnatts      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indnkeyatts      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indisunique      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indnullsnotdistinct      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indisprimary      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indisexclusion      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indimmediate      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indisclustered      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indisvalid      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indcheckxmin      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indisready      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indislive      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indisreplident      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indkey      </code></td>
<td><code dir="ltr" translate="no">       int8[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indcollation      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indclass      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indoption      </code></td>
<td><code dir="ltr" translate="no">       int8[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       indexprs      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indpred      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_namespace    `

The [`  pg_namespace  `](https://www.postgresql.org/docs/current/catalog-pg-namespace.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       nspname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       nspowner      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_proc    `

The [`  pg_proc  `](https://www.postgresql.org/docs/current/catalog-pg-proc.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       proname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pronamespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       proowner      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       prolang      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       procost      </code></td>
<td><code dir="ltr" translate="no">       float8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       prorows      </code></td>
<td><code dir="ltr" translate="no">       float8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       provariadic      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       prokind      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       prosecdef      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proleakproof      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       proisstrict      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proretset      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       provolatile      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proparallel      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       pronargs      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pronargdefaults      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       prorettype      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proargtypes      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       proallargtypes      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proargmodes      </code></td>
<td><code dir="ltr" translate="no">       char[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       proargdefaults      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       protrftypes      </code></td>
<td><code dir="ltr" translate="no">       oid[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       prosrc      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       probin      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       prosqlbody      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       proconfig      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### `     pg_sequence    `

The [`  pg_sequence  `](https://www.postgresql.org/docs/current/catalog-pg-sequence.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       seqrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       seqtypid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       seqstart      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       seqincrement      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       seqmax      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       seqmin      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td>seqcache</td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       seqcycle      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
</tbody>
</table>

### `     pg_type    `

The [`  pg_type  `](https://www.postgresql.org/docs/current/catalog-pg-type.html) table has content.

The following table shows whether columns have content.

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Has content</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typnamespace      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typowner      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typlen      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typbyval      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typtype      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typcategory      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typispreferred      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typisdefined      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typdelim      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typrelid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typelem      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typarray      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typalign      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typstorage      </code></td>
<td><code dir="ltr" translate="no">       char      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typnotnull      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typbasetype      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typtypmod      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typndims      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typcollation      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       typdeafultbin      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       typdefault      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

## What's next

  - [System views](/spanner/docs/reference/postgresql/pg-system-catalog-views)
