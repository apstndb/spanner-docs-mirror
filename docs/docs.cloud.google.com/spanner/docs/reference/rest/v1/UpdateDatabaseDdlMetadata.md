  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by `  databases.updateDdl  ` .

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
  &quot;database&quot;: string,
  &quot;statements&quot;: [
    string
  ],
  &quot;commitTimestamps&quot;: [
    string
  ],
  &quot;throttled&quot;: boolean,
  &quot;progress&quot;: [
    {
      object (OperationProgress)
    }
  ],
  &quot;actions&quot;: [
    {
      object (DdlStatementActionInfo)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  database  `

`  string  `

The database being modified.

`  statements[]  `

`  string  `

For an update this list contains all the statements. For an individual statement, this list contains only that statement.

`  commitTimestamps[]  `

`  string ( Timestamp  ` format)

Reports the commit timestamps of all statements that have succeeded so far, where `  commitTimestamps[i]  ` is the commit timestamp for the statement `  statements[i]  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  throttled  `

`  boolean  `

Output only. When true, indicates that the operation is throttled, for example, due to resource constraints. When resources become available the operation will resume and this field will be false again.

`  progress[]  `

`  object ( OperationProgress  ` )

The progress of the `  databases.updateDdl  ` operations. All DDL statements will have continuously updating progress, and `  progress[i]  ` is the operation progress for `  statements[i]  ` . Also, `  progress[i]  ` will have start time and end time populated with commit timestamp of operation, as well as a progress of 100% once the operation has completed.

`  actions[]  `

`  object ( DdlStatementActionInfo  ` )

The brief action info for the DDL statements. `  actions[i]  ` is the brief info for `  statements[i]  ` .
