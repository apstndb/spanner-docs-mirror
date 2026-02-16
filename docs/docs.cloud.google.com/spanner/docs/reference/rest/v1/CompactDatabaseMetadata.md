  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the long-running operation returned by `  CALL compact_all()  ` , which can be executed using `  ExecuteSql  ` or `  sessions.executeStreamingSql  ` APIs.

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
  &quot;progress&quot;: {
    object (OperationProgress)
  },
  &quot;cancelTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  database  `

`  string  `

Output only. The database being compacted.

`  progress  `

`  object ( OperationProgress  ` )

Output only. The progress of the compaction operation.

`  cancelTime  `

`  string ( Timestamp  ` format)

Output only. The time at which cancellation of this operation was received. `  Operations.CancelOperation  ` starts asynchronous cancellation on a long-running operation. The server makes a best effort to cancel the operation, but success is not guaranteed. Clients can use `  Operations.GetOperation  ` or other methods to check whether the cancellation succeeded or whether the operation completed despite cancellation. On successful cancellation, the operation is not deleted; instead, it becomes an operation with an Operation.error value with a `  google.rpc.Status.code  ` of 1, corresponding to `  Code.CANCELLED  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .
