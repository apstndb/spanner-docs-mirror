---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryOptions
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryOptions
title: QueryOptions
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryOptions#SCHEMA_REPRESENTATION)

Query optimizer configuration.

**JSON representation**

```
{
  "optimizerVersion": string,
  "optimizerStatisticsPackage": string
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>optimizerVersion</code></td>
<td><p><code>string</code></p>
<p>An option to control the selection of optimizer version.</p>
<p>This parameter allows individual queries to pick different query optimizer versions.</p>
<p>Specifying <code>latest</code> as a value instructs Cloud Spanner to use the latest supported query optimizer version. If not specified, Cloud Spanner uses the optimizer version set at the database level options. Any other positive integer (from the list of supported optimizer versions) overrides the default optimizer version for query execution.</p>
<p>The list of supported optimizer versions can be queried from <code>SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS</code> .</p>
<p>Executing a SQL statement with an invalid optimizer version fails with an <code>INVALID_ARGUMENT</code> error.</p>
<p>See <a href="https://cloud.google.com/spanner/docs/query-optimizer/manage-query-optimizer">https://cloud.google.com/spanner/docs/query-optimizer/manage-query-optimizer</a> for more information on managing the query optimizer.</p>
<p>The <code>optimizerVersion</code> statement hint has precedence over this setting.</p></td>
</tr>
<tr class="even">
<td><code>optimizerStatisticsPackage</code></td>
<td><p><code>string</code></p>
<p>An option to control the selection of optimizer statistics package.</p>
<p>This parameter allows individual queries to use a different query optimizer statistics package.</p>
<p>Specifying <code>latest</code> as a value instructs Cloud Spanner to use the latest generated statistics package. If not specified, Cloud Spanner uses the statistics package set at the database level options, or the latest package if the database option isn't set.</p>
<p>The statistics package requested by the query has to be exempt from garbage collection. This can be achieved with the following DDL statement:</p>
<pre class="sql"><code>ALTER STATISTICS &lt;package_name&gt; SET OPTIONS (allow_gc=false)</code></pre>
<p>The list of available statistics packages can be queried from <code>INFORMATION_SCHEMA.SPANNER_STATISTICS</code> .</p>
<p>Executing a SQL statement with an invalid optimizer statistics package or with a statistics package that allows garbage collection fails with an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
</tbody>
</table>
