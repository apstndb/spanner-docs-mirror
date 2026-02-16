  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [UpdateDatabaseRequest](#UpdateDatabaseRequest)
      - [JSON representation](#UpdateDatabaseRequest.SCHEMA_REPRESENTATION)

Metadata type for the operation returned by `  databases.patch  ` .

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
  &quot;request&quot;: {
    object (UpdateDatabaseRequest)
  },
  &quot;progress&quot;: {
    object (OperationProgress)
  },
  &quot;cancelTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  request  `

`  object ( UpdateDatabaseRequest  ` )

The request for `  databases.patch  ` .

`  progress  `

`  object ( OperationProgress  ` )

The progress of the `  databases.patch  ` operation.

`  cancelTime  `

`  string ( Timestamp  ` format)

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is best-effort).

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

## UpdateDatabaseRequest

The request for `  databases.patch  ` .

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
  &quot;database&quot;: {
    object (Database)
  },
  &quot;updateMask&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  database  `

`  object ( Database  ` )

Required. The database to update. The `  name  ` field of the database is of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

`  updateMask  `

`  string ( FieldMask  ` format)

Required. The list of fields to update. Currently, only `  enableDropProtection  ` field can be updated.

This is a comma-separated list of fully qualified names of fields. Example: `  "user.displayName,photo"  ` .
