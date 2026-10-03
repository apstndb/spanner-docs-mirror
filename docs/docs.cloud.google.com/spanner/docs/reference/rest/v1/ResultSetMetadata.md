---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata
title: ResultSetMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata#SCHEMA_REPRESENTATION)

Metadata about a [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) or [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet) .

**JSON representation**

```
{
  "rowType": {
    object (StructType)
  },
  "transaction": {
    object (Transaction)
  },
  "undeclaredParameters": {
    object (StructType)
  }
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
<td><code>rowType</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType"><code>StructType</code></a><code> )</code></p>
<p>Indicates the field names and types for the rows in the result set. For example, a SQL query like <code>"SELECT UserId, UserName FROM Users"</code> could return a <code>rowType</code> value like:</p>
<pre data-fenced=""><code>&quot;fields&quot;: [
  { &quot;name&quot;: &quot;UserId&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;INT64&quot; } },
  { &quot;name&quot;: &quot;UserName&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;STRING&quot; } },
]</code></pre></td>
</tr>
<tr class="even">
<td><code>transaction</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction"><code>Transaction</code></a><code> )</code></p>
<p>If the read or SQL query began a transaction as a side-effect, the information about the new transaction is yielded here.</p></td>
</tr>
<tr class="odd">
<td><code>undeclaredParameters</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType"><code>StructType</code></a><code> )</code></p>
<p>A SQL query can be parameterized. In PLAN mode, these parameters can be undeclared. This indicates the field names and types for those undeclared parameters in the SQL query. For example, a SQL query like <code>"SELECT * FROM Users where UserId = @userId and UserName = @userName "</code> could return a <code>undeclaredParameters</code> value like:</p>
<pre data-fenced=""><code>&quot;fields&quot;: [
  { &quot;name&quot;: &quot;UserId&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;INT64&quot; } },
  { &quot;name&quot;: &quot;UserName&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;STRING&quot; } },
]</code></pre></td>
</tr>
</tbody>
</table>
