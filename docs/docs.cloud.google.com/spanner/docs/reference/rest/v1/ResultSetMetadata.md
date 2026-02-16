  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata about a `  ResultSet  ` or `  PartialResultSet  ` .

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;rowType&quot;: {
    object (StructType)
  },
  &quot;transaction&quot;: {
    object (Transaction)
  },
  &quot;undeclaredParameters&quot;: {
    object (StructType)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  rowType  `

`  object ( StructType  ` )

Indicates the field names and types for the rows in the result set. For example, a SQL query like `  "SELECT UserId, UserName FROM Users"  ` could return a `  rowType  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```

`  transaction  `

`  object ( Transaction  ` )

If the read or SQL query began a transaction as a side-effect, the information about the new transaction is yielded here.

`  undeclaredParameters  `

`  object ( StructType  ` )

A SQL query can be parameterized. In PLAN mode, these parameters can be undeclared. This indicates the field names and types for those undeclared parameters in the SQL query. For example, a SQL query like `  "SELECT * FROM Users where UserId = @userId and UserName = @userName "  ` could return a `  undeclaredParameters  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```
