  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [ChangeQuorumRequest](#ChangeQuorumRequest)
      - [JSON representation](#ChangeQuorumRequest.SCHEMA_REPRESENTATION)

Metadata type for the long-running operation returned by `  databases.changequorum  ` .

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
    object (ChangeQuorumRequest)
  },
  &quot;startTime&quot;: string,
  &quot;endTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  request  `

`  object ( ChangeQuorumRequest  ` )

The request for `  databases.changequorum  ` .

`  startTime  `

`  string ( Timestamp  ` format)

Time the request was received.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  endTime  `

`  string ( Timestamp  ` format)

If set, the time at which this operation failed or was completed successfully.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

## ChangeQuorumRequest

The request for `  databases.changequorum  ` .

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
  &quot;quorumType&quot;: {
    object (QuorumType)
  },
  &quot;etag&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Required. Name of the database in which to apply `  databases.changequorum  ` . Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

`  quorumType  `

`  object ( QuorumType  ` )

Required. The type of this quorum.

`  etag  `

`  string  `

Optional. The etag is the hash of the `  QuorumInfo  ` . The `  databases.changequorum  ` operation is only performed if the etag matches that of the `  QuorumInfo  ` in the current database resource. Otherwise the API returns an `  ABORTED  ` error.

The etag is used for optimistic concurrency control as a way to help prevent simultaneous change quorum requests that could create a race condition.
