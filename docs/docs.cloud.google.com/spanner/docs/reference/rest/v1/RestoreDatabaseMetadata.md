  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the long-running operation returned by `  databases.restore  ` .

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
  &quot;sourceType&quot;: enum (RestoreSourceType),
  &quot;progress&quot;: {
    object (OperationProgress)
  },
  &quot;cancelTime&quot;: string,
  &quot;optimizeDatabaseOperationName&quot;: string,

  // Union field source_info can be only one of the following:
  &quot;backupInfo&quot;: {
    object (BackupInfo)
  }
  // End of list of possible types for union field source_info.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Name of the database being created and restored to.

`  sourceType  `

`  enum ( RestoreSourceType  ` )

The type of the restore source.

`  progress  `

`  object ( OperationProgress  ` )

The progress of the `  databases.restore  ` operation.

`  cancelTime  `

`  string ( Timestamp  ` format)

The time at which cancellation of this operation was received. `  Operations.CancelOperation  ` starts asynchronous cancellation on a long-running operation. The server makes a best effort to cancel the operation, but success is not guaranteed. Clients can use `  Operations.GetOperation  ` or other methods to check whether the cancellation succeeded or whether the operation completed despite cancellation. On successful cancellation, the operation is not deleted; instead, it becomes an operation with an Operation.error value with a `  google.rpc.Status.code  ` of 1, corresponding to `  Code.CANCELLED  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  optimizeDatabaseOperationName  `

`  string  `

If exists, the name of the long-running operation that will be used to track the post-restore optimization process to optimize the performance of the restored database, and remove the dependency on the restore source. The name is of the form `  projects/<project>/instances/<instance>/databases/<database>/operations/<operation>  ` where the is the name of database being created and restored to. The metadata type of the long-running operation is `  OptimizeRestoredDatabaseMetadata  ` . This long-running operation will be automatically created by the system after the databases.restore long-running operation completes successfully. This operation will not be created if the restore was not successful.

Union field `  source_info  ` . Information about the source used to restore the database, as specified by `  source  ` in `  RestoreDatabaseRequest  ` . `  source_info  ` can be only one of the following:

`  backupInfo  `

`  object ( BackupInfo  ` )

Information about the backup used to restore the database.
