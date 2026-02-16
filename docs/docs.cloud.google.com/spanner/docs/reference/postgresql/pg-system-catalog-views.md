This page lists the PostgreSQL system catalog views that Spanner supports.

### pg\_available\_extension\_versions

The [`  pg_available_extension_versions  `](https://www.postgresql.org/docs/current/view-pg-available-extension-versions.html) view has no content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       version      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       installed      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       superuser      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       trusted      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       relocatable      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       schema      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       requires      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       comment      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_available\_extensions

The [`  pg_available_extensions  `](https://www.postgresql.org/docs/current/view-pg-available-extensions.html) view has no content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       default_version      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       installed_version      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       comment      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_config

The [`  pg_config  `](https://www.postgresql.org/docs/current/view-pg-config.html)

view has no content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       setting      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_cursors

The [`  pg_cursors  `](https://www.postgresql.org/docs/current/view-pg-cursors.html)

view has no content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       statement      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_holdable      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       is_binary      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       is_scrollable      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       creation_time      </code></td>
<td><code dir="ltr" translate="no">       timestamptz      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_file\_settings

The [`  pg_file_settings  `](https://www.postgresql.org/docs/current/view-pg-file-settings.html) view has no content.

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
<td><code dir="ltr" translate="no">       sourcefile      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sourceline      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       seqno      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       setting      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       applied      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       error      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_hba\_file\_rules

The [`  pg_hba_file_rules  `](https://www.postgresql.org/docs/current/view-pg-hba-file-rules.html) view has no content.

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
<td><code dir="ltr" translate="no">       rule_number      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       file_name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       line_number      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       type      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       database      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       user_name      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       address      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       netmask      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       auth_method      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       options      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       error      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_indexes

The [`  pg_indexes  `](https://www.postgresql.org/docs/current/view-pg-indexes.html)

view has content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indexname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablespace      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       indexdef      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_matviews

The [`  pg_matviews  `](https://www.postgresql.org/docs/current/view-pg-matviews.html)

view has no content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       matviewname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       matviewowner      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablespace      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       hasindexes      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ispopulated      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       definition      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_policies

The [`  pg_policies  `](https://www.postgresql.org/docs/current/view-pg-policies.html)

view has no content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       policyname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       permissive      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       roles      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       cmd      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       qual      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       with_check      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_prepared\_xacts

The [`  pg_prepared_xacts  `](https://www.postgresql.org/docs/current/view-pg-prepared-xacts.html) view has no content.

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
<td><code dir="ltr" translate="no">       transaction      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       gid      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       prepared      </code></td>
<td><code dir="ltr" translate="no">       timestamptz      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       owner      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       database      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_publication\_tables

The [`  pg_publication_tables  `](https://www.postgresql.org/docs/current/view-pg-publication-tables.html) view has no content.

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
<td><code dir="ltr" translate="no">       pubname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       tablename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       attnames      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rowfilter      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_roles

The [`  pg_roles  `](https://www.postgresql.org/docs/current/view-pg-roles.html) view has no content.

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
<td><code dir="ltr" translate="no">       rolname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolsuper      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rolinherit      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolcreaterole      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rolcreatedb      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolcanlogin      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rolreplication      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolconnlimit      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rolpassword      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolvaliduntil      </code></td>
<td><code dir="ltr" translate="no">       timestamptz      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rolbypassrls bool      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rolconfig      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       oid      </code></td>
<td><code dir="ltr" translate="no">       oid      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_rules

The [`  pg_rules  `](https://www.postgresql.org/docs/current/view-pg-rules.html) view has no content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       rulename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       definition      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_sequences

The [`  pg_sequences  `](https://www.postgresql.org/docs/current/view-pg-sequences.html)

view has content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sequencename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sequenceowner      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       start_value      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       min_value      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       max_value      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       increment_by      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       cycle      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       cache_size      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       last_value      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_settings

The [`  pg_settings  `](https://www.postgresql.org/docs/current/view-pg-settings.html)

view has content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       setting      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       unit      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       category      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       short_desc      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       extra_desc      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       context      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       vartype      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       source      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       min_val      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       max_val      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       enumvals      </code></td>
<td><code dir="ltr" translate="no">       text[]      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       boot_val      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       reset_val      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       sourcefile      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       sourceline      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       pending_restart      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
</tbody>
</table>

### pg\_shmem\_allocations

The [`  pg_shmem_allocations  `](https://www.postgresql.org/docs/current/view-pg-shmem-allocations.html) view has no content.

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
<td><code dir="ltr" translate="no">       name      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       off      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       size      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       allocated_size      </code></td>
<td><code dir="ltr" translate="no">       int8      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_tables

The [`  pg_tables  `](https://www.postgresql.org/docs/current/view-pg-tables.html)

view has content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablename      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       tableowner      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       tablespace      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       hasindexes      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       hasrules      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       hastriggers      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       rowsecurity      </code></td>
<td><code dir="ltr" translate="no">       bool      </code></td>
<td>N</td>
</tr>
</tbody>
</table>

### pg\_views

The [`  pg_views  `](https://www.postgresql.org/docs/current/view-pg-views.html) view has content.

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
<td><code dir="ltr" translate="no">       schemaname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       viewname      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       viewowner      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>N</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       definition      </code></td>
<td><code dir="ltr" translate="no">       text      </code></td>
<td>Y</td>
</tr>
</tbody>
</table>

## What's next

  - [System catalogs](/spanner/docs/reference/postgresql/pg-system-catalog-tables)
