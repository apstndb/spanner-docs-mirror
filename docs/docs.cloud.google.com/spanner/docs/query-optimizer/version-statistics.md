Spanner provides a built-in `  SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS  ` table to keep track of query optimizer versions. You can retrieve this data using SQL queries.

## `     SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS    ` table schema

<table>
<thead>
<tr class="header">
<th>Column name</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       VERSION      </code></td>
<td><code dir="ltr" translate="no">       INT64      </code></td>
<td>The optimizer version.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       RELEASE_DATE      </code></td>
<td><code dir="ltr" translate="no">       DATE      </code></td>
<td>The release date of the optimizer version.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       IS_DEFAULT      </code></td>
<td><code dir="ltr" translate="no">       BOOL      </code></td>
<td>Whether the version is the default version.</td>
</tr>
</tbody>
</table>

### List all supported optimizer versions

``` text
SELECT * FROM SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS
```

An example result:

<table>
<thead>
<tr class="header">
<th>VERSION</th>
<th>RELEASE_DATE</th>
<th>IS_DEFAULT</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>2019-06-18</td>
<td>false</td>
</tr>
<tr class="even">
<td>2</td>
<td>2020-03-01</td>
<td>false</td>
</tr>
<tr class="odd">
<td>3</td>
<td>2021-08-01</td>
<td>true</td>
</tr>
</tbody>
</table>

## What's next

  - To learn more about the query optimizer, see [Query optimizer overview](/spanner/docs/query-optimizer/overview) .
  - To learn more about how the query optimizer has evolved, see [Query optimizer versions](/spanner/docs/query-optimizer/versions) .
