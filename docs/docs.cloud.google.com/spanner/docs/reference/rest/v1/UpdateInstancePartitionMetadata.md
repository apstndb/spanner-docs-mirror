  - [JSON representation](#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by `  instancePartitions.patch  ` .

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
  &quot;instancePartition&quot;: {
    object (InstancePartition)
  },
  &quot;startTime&quot;: string,
  &quot;cancelTime&quot;: string,
  &quot;endTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  instancePartition  `

`  object ( InstancePartition  ` )

The desired end state of the update.

`  startTime  `

`  string ( Timestamp  ` format)

The time at which `  instancePartitions.patch  ` request was received.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  cancelTime  `

`  string ( Timestamp  ` format)

The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  endTime  `

`  string ( Timestamp  ` format)

The time at which this operation failed or was completed successfully.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .
