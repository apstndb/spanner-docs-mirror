  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by `  backups.copy  ` .

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
  &quot;name&quot;: string,
  &quot;sourceBackup&quot;: string,
  &quot;progress&quot;: {
    object (OperationProgress)
  },
  &quot;cancelTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

The name of the backup being created through the copy operation. Values are of the form `  projects/<project>/instances/<instance>/backups/<backup>  ` .

`  sourceBackup  `

`  string  `

The name of the source backup that is being copied. Values are of the form `  projects/<project>/instances/<instance>/backups/<backup>  ` .

`  progress  `

`  object ( OperationProgress  ` )

The progress of the `  backups.copy  ` operation.

`  cancelTime  `

`  string ( Timestamp  ` format)

The time at which cancellation of backups.copy operation was received. `  Operations.CancelOperation  ` starts asynchronous cancellation on a long-running operation. The server makes a best effort to cancel the operation, but success is not guaranteed. Clients can use `  Operations.GetOperation  ` or other methods to check whether the cancellation succeeded or whether the operation completed despite cancellation. On successful cancellation, the operation is not deleted; instead, it becomes an operation with an `  Operation.error  ` value with a `  google.rpc.Status.code  ` of 1, corresponding to `  Code.CANCELLED  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .
