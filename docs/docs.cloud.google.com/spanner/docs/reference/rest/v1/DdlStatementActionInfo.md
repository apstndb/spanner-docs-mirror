  - [JSON representation](#SCHEMA_REPRESENTATION)

Action information extracted from a DDL statement. This proto is used to display the brief info of the DDL statement for the operation `  databases.updateDdl  ` .

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
  &quot;action&quot;: string,
  &quot;entityType&quot;: string,
  &quot;entityNames&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  action  `

`  string  `

The action for the DDL statement, for example, CREATE, ALTER, DROP, GRANT, etc. This field is a non-empty string.

`  entityType  `

`  string  `

The entity type for the DDL statement, for example, TABLE, INDEX, VIEW, etc. This field can be empty string for some DDL statement, for example, for statement "ANALYZE", `  entityType  ` = "".

`  entityNames[]  `

`  string  `

The entity names being operated on the DDL statement. For example, 1. For statement "CREATE TABLE t1(...)", `  entityNames  ` = \["t1"\]. 2. For statement "GRANT ROLE r1, r2 ...", `  entityNames  ` = \["r1", "r2"\]. 3. For statement "ANALYZE", `  entityNames  ` = \[\].
