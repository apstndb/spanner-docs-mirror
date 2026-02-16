The information schema is a built-in schema that's common to every PostgreSQL database. You can run SQL queries against tables in the `  information_schema  ` to fetch schema metadata for a database.

For example, the following query fetches the names of all user-defined tables in a database:

``` text
  SELECT
    table_schema,
    table_name
  FROM
    information_schema.tables
  WHERE
    table_schema NOT IN ('pg_catalog', 'information_schema', 'SPANNER_SYS')
    AND table_type = 'BASE TABLE'
```

## Usage

  - `  information_schema  ` tables are available only through SQL interfaces, for example:
    
      - The `  executeQuery  ` API
      - The `  gcloud spanner databases execute-sql  ` command
      - The **Query** page of a database in the Google Cloud console.
    
    Other single read methods don't support `  information_schema  ` .

<!-- end list -->

  - Queries against the `  information_schema  ` can use strong, bounded staleness, or exact staleness [timestamp bounds](/spanner/docs/timestamp-bounds) .
  - If you are using a GoogleSQL-dialect database, see [Information schema for GoogleSQL-dialect databases](/spanner/docs/information-schema) .

## Differences from `     information_schema    ` for PostgreSQL

The tables in the `  information_schema  ` for PostgreSQL-dialect databases include columns from the tables in the `  information_schema  ` for open source PostgreSQL and in some cases also include columns from Spanner. In these tables, the open source PostgreSQL columns come first and in the same order as they do for a open source PostgreSQL database, and any distinct columns for Spanner are appended afterwards. Queries written for the open source PostgreSQL version of `  information_schema  ` should work without modification when using PostgreSQL-dialect databases in Google Cloud CLI.

**Note:** Queries against the `  information_schema  ` for PostgreSQL-dialect databases that use `  select *  ` and reference columns by offset might not work the same as they do against open source PostgreSQL databases. The GoogleSQL-specific columns can shift in position if PostgreSQL adds new columns to an `  information_schema  ` table.

Other notable differences in the `  information_schema  ` for PostgreSQL-dialect databases are:

  - Some of the table columns for open source PostgreSQL are available, but not populated in PostgreSQL-dialect databases.
  - PostgreSQL-dialect databases use `  public  ` for the default schema name.
  - Automatically generated constraint names use a different format than open source PostgreSQL databases.
  - Tables related to open source PostgreSQL features that are not supported in PostgreSQL-dialect databases are not available.
  - Some tables that are available with Spanner but not open source PostgreSQL, such as `  database_options  ` , `  index_columns  ` , `  indexes  ` , and `  spanner_statistics  ` are available.

## Row filtering in `     information_schema    ` tables and views

Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` [system role](/spanner/docs/fgac-system-roles) (or to members of that role) can see all rows in all `  information_schema  ` tables and views. For other principals, Spanner filters rows based on the current database role. The table and view descriptions in the following sections indicate how Spanner filters rows for each table and view.

## Tables in `     information_schema    ` for PostgreSQL-dialect databases

The tables and views in the `  information_schema  ` are compatible with the tables and views in the `  information_schema  ` of open source PostgreSQL.

The following sections describe the tables and views in the `  information_schema  ` for PostgreSQL-dialect databases.

### `     applicable_roles    `

This row-filtered view lists all role memberships that are explicitly granted to all database roles. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only the role memberships that are granted to the current database role or to a role of which the current database role is a member.

Because all database roles are members of the [public role](/spanner/docs/fgac-system-roles#public) , the results omit records for implicit membership in the public role.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which membership is granted.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       role_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the parent database role in which this membership is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     change_stream_columns    `

This row-filtered view contains information about table columns and the change streams that watch them. Each row describes one change stream and one column. If a change stream tracks an entire table, then the columns in that table don't show in this view.

Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only rows for change streams on which the `  SELECT  ` privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that this row refers to.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column that this row refers to.</td>
</tr>
</tbody>
</table>

### `     change_stream_options    `

This row-filtered view contains the configuration options for change streams. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only options for change streams on which the `  SELECT  ` privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream option.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the change stream option.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value of the change stream option.</td>
</tr>
</tbody>
</table>

### `     change_stream_privileges    `

This row-filtered view lists all fine-grained access control privileges granted on all change streams to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on change streams to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the change stream. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> only).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     change_stream_tables    `

This row-filtered view contains information about tables and the change streams that watch them. Each row describes one table and one change stream. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only rows for change streams on which the `  SELECT  ` privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

The data in `  change_stream_tables  ` does not include the implicit relationships between tables and change streams that track the entire database.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream that this row refers to.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that this row refers to.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       all_columns      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td><code dir="ltr" translate="no">       YES      </code> if this row's change stream tracks the entirety of the table this row refers to. Otherwise, <code dir="ltr" translate="no">       NO      </code> . In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
</tbody>
</table>

### `     change_streams    `

This row-filtered view lists all of a database's change streams, and notes which ones track the entire database versus specific tables or columns. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only change streams on which the `  SELECT  ` fine-grained access control privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of this change stream's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       all      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td><code dir="ltr" translate="no">       YES      </code> if this change stream tracks the entire database. <code dir="ltr" translate="no">       NO      </code> if this change stream tracks specific tables or columns. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
</tbody>
</table>

### `     check_constraints    `

The `  check_constraints  ` view contains one row for each check constraint defined by either the `  CHECK  ` or the `  NOT NULL  ` keyword.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint's schema. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint. If the name of the constraint is not explicitly specified in the schema, the auto-generated name is used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       check_clause      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The check constraint's expression.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_state      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The current state of the check constraint. The possible states are as follows:
<ul>
<li><code dir="ltr" translate="no">         VALIDATING        </code> : The PostgreSQL-dialect database is validating the existing data for an <code dir="ltr" translate="no">         ALTER CONSTRAINT        </code> or <code dir="ltr" translate="no">         ADD CONSTRAINT        </code> command.</li>
<li><code dir="ltr" translate="no">         COMMITTED        </code> : There is no active schema change for this constraint.</li>
</ul></td>
</tr>
</tbody>
</table>

### `     column_column_usage    `

This view lists all the generated columns that depend on another base column in the same table.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table. The name is <code dir="ltr" translate="no">       public      </code> for the default schema and non-empty for other schemas (for example, the <code dir="ltr" translate="no">       information_schema      </code> itself). This column is never null.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that contains the generated columns.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the base column that the generated column depends on.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       dependent_column      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the generated column.</td>
</tr>
</tbody>
</table>

### `     column_options    `

This view lists all the options defined for the referenced table columns of a foreign key constraint. The view contains only those columns in the reference table that the current user has access to (by way of being the owner or granted privileges).

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the foreign table. The name is <code dir="ltr" translate="no">       public      </code> for the default schema and non-empty for other schemas (for example, the <code dir="ltr" translate="no">       information_schema      </code> itself). This column is never null.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the foreign table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A SQL identifier that uniquely identifies the option. This identifier is the key of the <code dir="ltr" translate="no">       OPTIONS      </code> clause in DDL.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A SQL literal describing the value of this option. The value of this column is parsable as part of a query.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A data type name that is the type of this option value.</td>
</tr>
</tbody>
</table>

### `     column_privileges    `

This row-filtered view lists all fine-grained access control privileges granted on all columns to any database role, including `  public  ` . Principals that have IAM database-level permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on columns to the current database role, to roles of which the current database role is a member, or to `  public  ` .

The view includes the `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` privileges that the column inherits from the table or view that contains the column.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table or view. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table or view that contains the column.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> , <code dir="ltr" translate="no">       INSERT      </code> , or <code dir="ltr" translate="no">       UPDATE      </code> ).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     columns    `

This row-filtered view provides information about all table columns and view columns in the database. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only columns that have any fine-grained access control privileges granted on them (or the `  SELECT  ` , `  INSERT  ` or `  UPDATE  ` privileges granted on their containing tables) to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table. The name is <code dir="ltr" translate="no">       public      </code> for the default schema and non-empty for other schemas (for example, the <code dir="ltr" translate="no">       information_schema      </code> itself). This column is never null.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ordinal_position      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The ordinal position of the column in the table, starting with a value of 1</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_default      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string representation of the open source PostgreSQL expression of the default value of the column, for example, <code dir="ltr" translate="no">       '9'::bigint      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_nullable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string that indicates whether the column is nullable. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       data_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the column. The value is one of the following:
<ul>
<li>For built-in types, the name of the data type.</li>
<li>For arrays, the value ARRAY.</li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_maximum_length      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The declared maximum length for character and bit string data types. If a maximum length was not specified, then the value is <code dir="ltr" translate="no">       NULL      </code> . If the data type of the column is not a character or bit string, then the value is <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_octet_length      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_precision      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The precision of the numeric data type of the current column. For <code dir="ltr" translate="no">       double precision      </code> , the value is 53. For <code dir="ltr" translate="no">       bigint      </code> , the value is 64. For all other data types, the value is <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       numeric_precision_radix      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The base (unit) of the precision for numeric types. Only two values are supported:
<ul>
<li>2 for <code dir="ltr" translate="no">         double precision        </code> <code dir="ltr" translate="no">         float8        </code> , and <code dir="ltr" translate="no">         bigint        </code></li>
<li>10 for <code dir="ltr" translate="no">         numeric        </code></li>
</ul>
For all other data types the value is <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_scale      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>Contains the scale of the numeric column type, which is the number of precision base units after the radix point. For <code dir="ltr" translate="no">       bigint      </code> , the value is 0. For all other data types, the value is <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       datetime_precision      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interval_precision      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_set_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_set_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_set_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collation_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collation_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collation_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       domain_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       domain_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       domain_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       udt_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       udt_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       udt_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       scope_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       scope_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       scope_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       maximum_cardinality      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       dtd_identifier      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_self_referencing      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_identity      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       identity_generation      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       identity_start      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       identity_increment      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       identity_maximum      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       identity_minimum      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       identity_cycle      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_generated      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string that indicates whether the column is generated. The string is either <code dir="ltr" translate="no">       ALWAYS      </code> for a generated column or <code dir="ltr" translate="no">       NEVER      </code> for a non-generated column.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       generation_expression      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string representing the SQL expression of a generated column, or <code dir="ltr" translate="no">       NULL      </code> if the column is not a generated column.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_updatable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string holding the DDL-compatible type of the column.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_stored      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string that indicates whether the generated column is stored. The string is always <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> for generated columns, and <code dir="ltr" translate="no">       NULL      </code> for non-generated columns.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_state      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The current state of the column. A new stored generated column added to an existing table may go through multiple user-observable states before it is fully usable. Possible values are:
<ul>
<li><code dir="ltr" translate="no">         NO_WRITE        </code> : No read or write is allowed to the columns. A stored generated column in this state does not cause any client effect.</li>
<li><code dir="ltr" translate="no">         WRITE_ONLY        </code> : The column is being backfilled. No read is allowed.</li>
<li><code dir="ltr" translate="no">         COMMITTED        </code> : The column is fully usable.</li>
<li><code dir="ltr" translate="no">         NULL        </code> : Used for columns in system schemas.</li>
</ul></td>
</tr>
</tbody>
</table>

### `     constraint_column_usage    `

This view contains one row about each column used by a constraint.

  - For `  PRIMARY KEY  ` and `  CHECK  ` constraints defined by the `  NOT NULL  ` keyword, the view contains those columns.
  - For `  CHECK  ` constraints created with the `  CHECK  ` keyword, the view includes the columns used by the check constraint expression.
  - For foreign key constraints, the view contains the columns of the referenced table.
  - For `  UNIQUE  ` constraints, the view contains the columns from `  KEY_COLUMN_USAGE  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table that contains the column that is used by the constraint.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that contains the column that is used by the constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column that is used by the constraint.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint's schema.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint.</td>
</tr>
</tbody>
</table>

### `     constraint_table_usage    `

This view contains one row for each table used by a constraint. For `  FOREIGN KEY  ` constraints, the table information is for the tables in the `  REFERENCES  ` clause. For a unique or primary key constraint, this view identifies the table the constraint belongs to. Check constraints and not-null constraints are not included in this view.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constrained table's schema.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that is used by some constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint.</td>
</tr>
</tbody>
</table>

### `     database_options    `

This table lists the options that are set on the database.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       catalog_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       schema_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema. The default value is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database option. This is the value of <code dir="ltr" translate="no">       key      </code> in the <code dir="ltr" translate="no">       OPTIONS      </code> clause in DDL.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the database option.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value of the database option.</td>
</tr>
</tbody>
</table>

### `     enabled_roles    `

This row-filtered view lists the defined database roles. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all database roles. All other principals can see only database roles to which they have been granted access either directly or through inheritance. All system roles excluding `  public  ` also appear in this view.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       role_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the role.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner_is_system      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td><code dir="ltr" translate="no">       YES      </code> if the role is a system role. Otherwise, <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     index_columns    `

This view lists the columns in an index.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the index. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table associated with the index.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       index_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the index. Tables that have a <code dir="ltr" translate="no">       PRIMARY KEY      </code> specification have a pseudo-index entry generated with the name <code dir="ltr" translate="no">       PRIMARY_KEY      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       index_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of index. Possible values are <code dir="ltr" translate="no">       PRIMARY_KEY      </code> , <code dir="ltr" translate="no">       LOCAL      </code> , or <code dir="ltr" translate="no">       GLOBAL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ordinal_position      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The ordinal position of the column in the index (or primary key), starting with a value of 1. This value is <code dir="ltr" translate="no">       NULL      </code> for non-key columns (for example, columns specified in the <a href="/spanner/docs/secondary-indexes#storing_clause"><code dir="ltr" translate="no">        INCLUDE       </code> clause</a> of an index).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_ordering      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The sort order of the column. The value is <code dir="ltr" translate="no">       ASC      </code> or <code dir="ltr" translate="no">       DESC      </code> for key columns, and <code dir="ltr" translate="no">       NULL      </code> for non-key columns (for example, columns specified in the <code dir="ltr" translate="no">       STORING      </code> clause of an index).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_nullable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string that indicates whether the column is nullable. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string holding the DDL-compatible type of the column.</td>
</tr>
</tbody>
</table>

### `     indexes    `

This view lists the indexes in a schema.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       index_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the index. Tables created with a <code dir="ltr" translate="no">       PRIMARY KEY      </code> clause have a pseudo-index entry generated with the name <code dir="ltr" translate="no">       PRIMARY_KEY      </code> , which allows the fields of the primary key to be identified.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       index_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the index. The values include <code dir="ltr" translate="no">       PRIMARY_KEY      </code> , <code dir="ltr" translate="no">       LOCAL      </code> , or <code dir="ltr" translate="no">       GLOBAL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       parent_table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Secondary indexes can be interleaved in a parent table, as discussed in <a href="/spanner/docs/secondary-indexes#creating_a_secondary_index">Creating a secondary index</a> . This column holds the name of that parent table, or an empty string if the index is not interleaved.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_unique      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether the index keys must be unique. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_null_filtered      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether the index includes entries with <code dir="ltr" translate="no">       NULL      </code> values. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       index_state      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The current state of the index. Possible values and the states they represent are:
<ul>
<li><code dir="ltr" translate="no">         NULL        </code> : the index type is <code dir="ltr" translate="no">         PRIMARY_KEY        </code></li>
<li><code dir="ltr" translate="no">         PREPARE        </code> : creating empty tables for a new index</li>
<li><code dir="ltr" translate="no">         WRITE_ONLY        </code> : backfilling data for a new index</li>
<li><code dir="ltr" translate="no">         WRITE_ONLY_CLEANUP        </code> : cleaning up a new index</li>
<li><code dir="ltr" translate="no">         WRITE_ONLY_VALIDATE_UNIQUE        </code> : checking uniqueness of data in a new index</li>
<li><code dir="ltr" translate="no">         READ_WRITE        </code> : normal index operation</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner_is_managed      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether the index is managed by Spanner. For example, secondary backing indexes for foreign keys are managed by Spanner. The string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value, in accordance with the SQL standard.</td>
</tr>
</tbody>
</table>

### `     information_schema_catalog_name    `

This table contains one row and one column containing the database name.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       catalog_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
</tbody>
</table>

### `     key_column_usage    `

This view identifies all columns in the current database that are referenced by a unique, primary key, or foreign key constraint. For information about `  CHECK  ` constraint columns, see the `  check_constraints  ` view.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint's schema. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table that contains the constrained column. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table that contains the column that is restricted by this constraint.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column that is constrained.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ordinal_position      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>The ordinal position of the column within the constraint's key, starting with a value of <code dir="ltr" translate="no">       1      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       position_in_unique_constraint      </code></td>
<td><code dir="ltr" translate="no">       BIGINT      </code></td>
<td>For <code dir="ltr" translate="no">       FOREIGN KEY      </code> s, the ordinal position of the column within the unique constraint, starting with a value of <code dir="ltr" translate="no">       1      </code> . This column has a <code dir="ltr" translate="no">       NULL      </code> value for other constraint types.</td>
</tr>
</tbody>
</table>

### `     parameters    `

This row-filtered view defines the arguments for each user defined and change stream read function. Each row describes one argument for one user defined or change stream read function.

Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only parameters for user defined and change stream read functions on which the `  EXECUTE  ` fine-grained access control privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. Uniquely identifies the routine even if its name is overloaded.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ordinal_position      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>The ordinal position of the parameter in the argument list of the routine, starting with a value 1.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       parameter_mode      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_result      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       as_locator      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       parameter_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the parameter.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       data_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the parameter. The value is one of the following:<br />

<ul>
<li>For built-in types, the name of the data type.</li>
</ul>
<ul>
<li>For arrays, the value <code dir="ltr" translate="no">         ARRAY        </code> .</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_maximum_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_octet_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_set_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_set_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_set_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collation_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collation_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collation_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       numeric_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_precision_radix      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       numeric_scale      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       datetime_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interval_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       udt_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       udt_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       udt_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       scope_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       scope_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       scope_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       maximum_cardinality      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       dtd_identifier      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       parameter_default      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The detailed return type of the routine. Includes the subtype if an <code dir="ltr" translate="no">       ARRAY      </code> is returned.</td>
</tr>
</tbody>
</table>

### `     placements    `

This table lists the placements in the database.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       placement_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the placement.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_default      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>A string that indicates whether the column is nullable. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
</tbody>
</table>

### `     placement-options    `

For each placement, this table lists the options that are set on the placement in the `  OPTIONS  ` clause of the [`  CREATE PLACEMENT  `](/spanner/docs/reference/postgresql/data-definition-language#create-placement) statement.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       placement_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the placement.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the placement option. The valid values for <code dir="ltr" translate="no">       option_name      </code> include:
<ul>
<li><code dir="ltr" translate="no">         instance_partition        </code></li>
<li><code dir="ltr" translate="no">         default_leader        </code></li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the placement option.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value of the placement option. For <code dir="ltr" translate="no">       instance_partition      </code> , this is the name of the instance partition. For <code dir="ltr" translate="no">       default_leader      </code> , it's the name of the default leader region.</td>
</tr>
</tbody>
</table>

### `     locality-group-options    `

For each locality group, this table lists the name and options that are set on the locality group in the `  OPTIONS  ` clause of the [`  CREATE LOCALITY GROUP  `](/spanner/docs/reference/postgresql/data-definition-language#create-locality-group) statement.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       locality_group_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the locality group.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the locality group option. The valid options are:
<ul>
<li><code dir="ltr" translate="no">         storage        </code> : defines the storage type for the locality group.</li>
<li><code dir="ltr" translate="no">         ssd_to_hdd_spill_timespan        </code> : defines how long data is stored in SSD storage before it moves to HDD storage.</li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value of the locality group option. For <code dir="ltr" translate="no">       storage      </code> , this is either <code dir="ltr" translate="no">       ssd      </code> or <code dir="ltr" translate="no">       hdd      </code> . For <code dir="ltr" translate="no">       ssd_to_hdd_spill_timespan      </code> , this is the amount of time that data must be stored in SSD before it's moved to HDD storage. For example, <code dir="ltr" translate="no">       10d      </code> is 10 days. The minimum amount of time you can set is one hour.</td>
</tr>
</tbody>
</table>

### `     referential_constraints    `

This view contains one row about each `  FOREIGN KEY  ` constraint. You can see only those constraints for which you have write access to the referencing table. This view also identifies the `  PRIMARY KEY  ` and `  UNIQUE  ` constraints on the referenced tables that the foreign keys use for constraint enforcement and referential actions.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the foreign key constraint. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the foreign key constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       unique_constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       unique_constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the unique or primary key constraint that the foreign key constraint references.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       unique_constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the unique or primary key constraint that the foreign key constraint references.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       match_option      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The match method used by the foreign key constraint. The value is always <code dir="ltr" translate="no">       NONE      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       update_rule      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The update rule of the foreign key constraint. This value is always <code dir="ltr" translate="no">       NO ACTION      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       delete_rule      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The delete rule of the foreign key constraint. This value is either <code dir="ltr" translate="no">       CASCADE      </code> or <code dir="ltr" translate="no">       NO ACTION      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner_state      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The current state of the foreign key. Spanner does not begin enforcing the constraint until the foreign key's backing indexes are created and backfilled. Once the indexes are ready, Spanner begins enforcing the constraint for new transactions while it validates the existing data. The possible values and the states they represent are:
<ul>
<li><code dir="ltr" translate="no">         BACKFILLING_INDEXES        </code> : Indexes are being backfilled.</li>
<li><code dir="ltr" translate="no">         VALIDATING_DATA        </code> : Existing data and new writes are being validated.</li>
<li><code dir="ltr" translate="no">         WAITING_FOR_COMMIT        </code> : The foreign key bulk operations have completed successfully, or none were needed, but the foreign key is still pending.</li>
<li><code dir="ltr" translate="no">         COMMITTED        </code> : The schema change was committed.</li>
</ul></td>
</tr>
</tbody>
</table>

### `     role_change_stream_grants    `

This row-filtered view lists the `  SELECT  ` privileges granted on all change streams to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on change streams to the current database role and to roles of which the current database role is a member, not including `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       change_stream_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the change stream. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       change_stream_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the change stream.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> only).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     role_column_grants    `

This row-filtered view lists all fine-grained access control privileges granted on all columns to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on columns to the current database role and to roles of which the current database role is a member, not including `  public  ` .

The view includes the `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` privileges that the column inherits from the table or view that contains the column.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table or view. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table or view that contains the column.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the column.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> , <code dir="ltr" translate="no">       INSERT      </code> , or <code dir="ltr" translate="no">       UPDATE      </code> ).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     role_routine_grants    `

This row-filtered view lists the `  EXECUTE  ` privileges granted on all change stream read functions to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on change stream read functions to the current database role and to roles of which the current database role is a member, not including `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the role that the privilege was granted to.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. Uniquely identifies the routine even if its name is overloaded.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       routine_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. (Might be duplicated in case of overloading.)</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege granted. Always <code dir="ltr" translate="no">       EXECUTE      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     role_table_grants    `

This row-filtered view lists all fine-grained access control privileges granted on all tables and views to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on tables and views to the current database role and to roles of which the current database role is a member, not including `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table or view. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table or view.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> , <code dir="ltr" translate="no">       INSERT      </code> , <code dir="ltr" translate="no">       UPDATE      </code> , or <code dir="ltr" translate="no">       DELETE      </code> ).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       with_hierarchy      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
</tbody>
</table>

### `     routine_options    `

This row-filtered view contains one row for each option for each defined change stream read function.

Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only options for change stream read functions on which the `  EXECUTE  ` fine-grained access control privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. Uniquely identifies the routine even if its name is overloaded.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the option.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       option_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The data type of the option. The value is one of the following:<br />

<ul>
<li>For built-in types, the name of the data type.</li>
</ul>
<ul>
<li>For arrays, the value <code dir="ltr" translate="no">         ARRAY        </code> .</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       option_value      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value of the option.</td>
</tr>
</tbody>
</table>

### `     routine_privileges    `

This row-filtered view lists all fine-grained access control privileges granted on all change stream read functions to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on change stream read functions to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the role that the privilege was granted to.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. For PostgreSQL-dialect databases, the default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. Uniquely identifies the routine even if its name is overloaded.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       routine_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. The default is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. (Might be duplicated if overloaded.)</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege granted.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
</tbody>
</table>

### `     routines    `

This row-filtered view lists all of a database's user defined and change stream read functions. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only user defined and change stream read functions on which the `  EXECUTE  ` fine-grained access control privilege is granted to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. Uniquely identifies the routine even if its name is overloaded.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       routine_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine's schema.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the routine. (Might be duplicated in case of overloading.)</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       routine_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the routine ( <code dir="ltr" translate="no">       FUNCTION      </code> or <code dir="ltr" translate="no">       PROCEDURE      </code> ). Always <code dir="ltr" translate="no">       FUNCTION      </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       module_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       module_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       module_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       udt_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       udt_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       udt_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       data_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The return type of the routine. The value is one of the following:<br />

<ul>
<li>For built-in types, the name of the data type.</li>
</ul>
<ul>
<li>For arrays, the value <code dir="ltr" translate="no">         ARRAY        </code> .</li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_maximum_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_octet_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_set_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       character_set_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       character_set_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collation_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       collation_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       collation_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       numeric_precision_radix      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_scale      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       datetime_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       interval_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interval_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       type_udt_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       type_udt_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       type_udt_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       scope_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       scope_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       scope_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       maximum_cardinality      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       dtd_identifier      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       routine_body      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the routine body ( <code dir="ltr" translate="no">       SQL      </code> or <code dir="ltr" translate="no">       EXTERNAL      </code> ).</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       routine_definition      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The definition for the <code dir="ltr" translate="no">       routine_body      </code> SQL, empty otherwise.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       external_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       external_language      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       parameter_style      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_deterministic      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sql_data_access      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_null_call      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sql_path      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       schema_level_routine      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       max_dynamic_result_sets      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_user_defined_cast      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_implicitly_invocable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       security_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The security type of the routine. Only <code dir="ltr" translate="no">       INVOKER      </code> is supported.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_sql_specific_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       to_sql_specific_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       to_sql_specific_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       as_locator      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       created      </code></td>
<td><code dir="ltr" translate="no">       timestamp with time zone      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       last_altered      </code></td>
<td><code dir="ltr" translate="no">       timestamp with time zone      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       new_savepoint_level      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_udt_dependent      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_from_data_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_as_locator      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_char_max_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_char_octet_length      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_char_set_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_char_set_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_char_set_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_collation_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_collation_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_collation_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_numeric_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_numeric_precision_radix      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_numeric_scale      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_datetime_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_interval_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_interval_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_type_udt_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_type_udt_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_type_udt_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_scope_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_scope_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_scope_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       result_cast_maximum_cardinality      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       result_cast_dtd_identifier      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. The value is always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The detailed return type of the routine. Includes the subtype if an <code dir="ltr" translate="no">       ARRAY      </code> is returned.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       spanner_determinism      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The user specified determinism of the function ( <code dir="ltr" translate="no">       DETERMINISTIC      </code> , <code dir="ltr" translate="no">       NOT_DETERMINISTIC_STABLE      </code> , or <code dir="ltr" translate="no">       NOT_DETERMINISTIC_VOLATILE      </code> ). Further info can be found at the description for [provolatile](https://www.postgresql.org/docs/current/catalog-pg-proc.html#:~:text=provolatile).</td>
</tr>
</tbody>
</table>

### `     schemata    `

The `  information_schema.schemata  ` view contains one row for each schema in the current database. The schemas include the information schema and a default schema named `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       catalog_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       schema_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema. This is set to <code dir="ltr" translate="no">       public      </code> for the default schema and non-empty for named schemas.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       schema_owner      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the owner of the schema.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       default_character_set_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       default_character_set_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       default_character_set_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sql_path      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       effective_timestamp      </code></td>
<td><code dir="ltr" translate="no">       timestamp with timezone      </code></td>
<td>The timestamp at which all the data in this schema became effective. This is used only for the default schema.</td>
</tr>
</tbody>
</table>

### `     sequences    `

The `  information_schema.sequences  ` view contains the `  sequences  ` metadata.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       sequence_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sequence_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the sequence's schema. The default is <code dir="ltr" translate="no">       public      </code> for a PostgreSQL-dialect database.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sequence_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the sequence.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       data_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Sequence only supports <code dir="ltr" translate="no">       int8      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_precision      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       numeric_precision_radix      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       numeric_scale      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       start_value      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       minimum_value      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       maximum_value      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       increment      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Not used. The value is always `NULL`.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       cycle_option      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The only option that <code dir="ltr" translate="no">       sequence      </code> accepts is <code dir="ltr" translate="no">       no      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sequence_kind      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The kind of sequence. <code dir="ltr" translate="no">       bit_reversed_positive      </code> is the only acceptable value.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       counter_start_value      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>Starting value of the sequence counter.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       skip_range_min      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>The minimum value in the skipped range. This value is <code dir="ltr" translate="no">       NULL      </code> if not set.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       skip_range_max      </code></td>
<td><code dir="ltr" translate="no">       bigint      </code></td>
<td>The maximum value in the skipped range. This value is <code dir="ltr" translate="no">       NULL      </code> if not set.</td>
</tr>
</tbody>
</table>

### `     spanner_statistics    `

This table lists the available query optimizer statistics packages.

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 10%" />
<col style="width: 70%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       catalog_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       schema_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema. The default schema value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       package_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the statistics package.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       allow_gc      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether the statistics package is exempted from garbage collection. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value. This attribute must be set to <code dir="ltr" translate="no">       NO      </code> before you can reference the statistics package in a hint or through the client API.</td>
</tr>
</tbody>
</table>

### `     table_constraints    `

This view contains all constraints belonging to tables that the current user has access to (other than `  SELECT  ` ).

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       constraint_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the constraint.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of schema that contains the table associated with the constraint.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       constraint_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the constraint. Possible values are:
<ul>
<li><code dir="ltr" translate="no">         CHECK        </code></li>
<li><code dir="ltr" translate="no">         FOREIGN KEY        </code></li>
<li><code dir="ltr" translate="no">         PLACEMENT KEY        </code></li>
<li><code dir="ltr" translate="no">         PRIMARY KEY        </code></li>
<li><code dir="ltr" translate="no">         UNIQUE        </code></li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_deferrable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value is always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       initially_deferred      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The value is always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       enforced      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether the constraint is enforced. If a constraint is enforced, (after it reaches a certain state), it's validated both at write time and by a background integrity verifier. In accordance with the SQL standard, the string is either <code dir="ltr" translate="no">       YES      </code> or <code dir="ltr" translate="no">       NO      </code> , rather than a Boolean value.</td>
</tr>
</tbody>
</table>

### `     table_privileges    `

This row-filtered view lists all fine-grained access control privileges granted on all tables and views to any database role, including `  public  ` . Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all rows in this view. All other principals can see only privileges granted on tables and views to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Column name</strong></th>
<th><strong>Type</strong></th>
<th><strong>Description</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       grantor      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       grantee      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the database role to which this privilege is granted.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table or view. The default is <code dir="ltr" translate="no">       public      </code> for PostgreSQL-dialect databases.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table or view.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       privilege_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The type of the privilege ( <code dir="ltr" translate="no">       SELECT      </code> , <code dir="ltr" translate="no">       INSERT      </code> , <code dir="ltr" translate="no">       UPDATE      </code> , or <code dir="ltr" translate="no">       DELETE      </code> ).</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_grantable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NO      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       that have_hierarchy      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used. Always <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
</tbody>
</table>

### `     tables    `

This row-filtered view lists all the tables and view that are in the current database. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all tables and views. All other principals can see only tables that meet either of the following requirements:

  - The `  SELECT  ` , `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` fine-grained access control privileges are granted on the table to the current database role, to roles of which the current database role is a member, or to `  public  ` .
  - The `  SELECT  ` , `  INSERT  ` , or `  UPDATE  ` privileges are granted on any table column to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema that contains the table or view.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the table, view, or synonym.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The table type. Possible values include 'BASE TABLE', 'VIEW', or 'SYNONYM'.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       self_referencing_column_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       reference_generation      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       user_defined_type_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       user_defined_type_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       user_defined_type_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_insertable_into      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_typed      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       commit_action      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       parent_table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the parent table if this table is interleaved, or <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       on_delete_action      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>This is set to <code dir="ltr" translate="no">       CASCADE      </code> or <code dir="ltr" translate="no">       NO ACTION      </code> for interleaved tables, and <code dir="ltr" translate="no">       NULL      </code> otherwise. See <a href="/spanner/docs/reference/standard-sql/data-definition-language#table_statements">TABLE statements</a> for more information.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       spanner_state      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The current creation state of the table.<br />
A table can go through multiple states during creation, if bulk operations are involved, for example, when the table is created with a foreign key that requires backfilling of its referenced index. Possible states are:
<ul>
<li><code dir="ltr" translate="no">         ADDING_FOREIGN_KEY        </code> : Adding the table's foreign keys</li>
<li><code dir="ltr" translate="no">         WAITING_FOR_COMMIT        </code> : Finalizing the schema change</li>
<li><code dir="ltr" translate="no">         COMMITTED        </code> : The schema change to create the table has been committed. You cannot write to the table until the change is committed.</li>
<li><code dir="ltr" translate="no">         NULL        </code> : Tables or views that are not base tables.</li>
</ul></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       interleave_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Whether there exists a parent-child relationship between this table and the table it is interleaved in. Possible values are:
<ul>
<li><code dir="ltr" translate="no">         IN        </code> : An <code dir="ltr" translate="no">         INTERLEAVE IN        </code> table that has no parent-child relationship. A row in this table can exist regardless of the existence of its parent table row.</li>
<li><code dir="ltr" translate="no">         IN PARENT        </code> : An <code dir="ltr" translate="no">         INTERLEAVE IN PARENT        </code> table that has a parent-child relationship. A row in this table requires the existence of its parent table row.</li>
</ul></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       row_deletion_policy_expression      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>An string that contains the expression text that defines the <code dir="ltr" translate="no">       ROW       DELETION POLICY      </code> .</td>
</tr>
</tbody>
</table>

### `     table_synonyms    `

This table lists synonym information for the table.

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       CATALOG      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Name of the catalog containing the table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SCHEMA      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Name of the schema containing the table.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       TABLE_NAME      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Name of the table.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SYNONYM_CATALOG      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The name of the catalog for the synonym.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       SYNONYM_SCHEMA      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The name of the schema for the synonym.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       SYNONYM_TABLE_NAME      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>The name of the table for the synonym.</td>
</tr>
</tbody>
</table>

### `     views    `

This row-filtered view lists all views in the current database. Principals that have database-level IAM permissions and principals who have been granted access to the `  spanner_info_reader  ` system role or to members of that role can see all views. All other principals can see only views that have the `  SELECT  ` fine-grained access control privilege granted on them to the current database role, to roles of which the current database role is a member, or to `  public  ` .

<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 10%" />
<col style="width: 65%" />
</colgroup>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_catalog      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The database name.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       table_schema      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the schema. The default value is <code dir="ltr" translate="no">       public      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       table_name      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The name of the view.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       view_definition      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The SQL text of the query that defines the view.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       check_option      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_updatable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_insertable_into      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_trigger_updatable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_trigger_deletable      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_trigger_insertable_into      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>Not used.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       security_type      </code></td>
<td><code dir="ltr" translate="no">       character varying      </code></td>
<td>The security type of the view. Either <code dir="ltr" translate="no">       INVOKER      </code> or <code dir="ltr" translate="no">       DEFINER      </code> .
<p>For more information, see <a href="/spanner/docs/views">About views</a> .</p></td>
</tr>
</tbody>
</table>

## Examples

Return information about each table in the user's schema:

``` text
SELECT
  t.table_schema,
  t.table_catalog,
  t.table_name,
  t.parent_table_name
FROM
  information_schema.tables AS t
WHERE
  t.table_schema NOT IN ('pg_catalog', 'information_schema', 'SPANNER_SYS')
  AND t.table_type = 'BASE TABLE'
ORDER BY
  t.table_catalog,
  t.table_schema,
  t.table_name
```

Return the name of all tables and views in the `  information_schema  ` for PostgreSQL-dialect databases:

``` text
SELECT table_name
FROM information_schema.tables
WHERE table_schema = "information_schema"
```

Return information about columns in the user table `  my_table  ` in the default schema:

``` text
SELECT
  t.ordinal_position,
  t.column_name,
  t.data_type,
  t.spanner_type,
  t.is_nullable
FROM
  information_schema.columns AS t
WHERE
  t.table_schema = 'public'
  AND
  t.table_name = 'my_table'
ORDER BY
  t.ordinal_position
```

Return information about each index in the default schema in the current database: \`\`\`postgresql SELECT t.table\_name, t.index\_name, t.parent\_table\_name FROM information\_schema.indexes AS t WHERE t.table\_schema = 'public' AND t.index\_type \!= 'PRIMARY\_KEY' ORDER BY t.table\_schema, t.table\_name, t.index\_name

Return columns that use non-default options in the default schema:

``` text
SELECT
  t.table_name,
  t.column_name,
  t.option_type,
  t.option_value,
  t.option_name
FROM
  information_schema.column_options AS t
WHERE
  t.table_schema = 'public'
ORDER BY
  t.table_schema,
  t.table_name,
  t.column_name,
  t.option_name
```

Return the current optimizer-related database options:

``` text
SELECT
  s.option_name,
  s.option_value
FROM
  information_schema.database_options s
WHERE
  s.schema_name='public'
  AND s.option_name IN ('optimizer_version',
    'optimizer_statistics_package')
```

Return all available statistics packages:

```` postgresql
SELECT *
FROM information_schema.spanner_statistics;
``` ## What's
next {: #whats-next}

+   Learn about available [Introspection tools](/spanner/docs/introspection) to
help you investigate database issues. 
  

  
    
  
````
