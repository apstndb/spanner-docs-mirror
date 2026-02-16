  - [JSON representation](#SCHEMA_REPRESENTATION)

A transaction.

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
  &quot;id&quot;: string,
  &quot;readTimestamp&quot;: string,
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  id  `

`  string ( bytes format)  `

`  id  ` may be used to identify the transaction in subsequent `  sessions.read  ` , `  ExecuteSql  ` , `  Commit  ` , or `  sessions.rollback  ` calls.

Single-use read-only transactions do not have IDs, because single-use transactions do not support multiple requests.

A base64-encoded string.

`  readTimestamp  `

`  string ( Timestamp  ` format)

For snapshot read-only transactions, the read timestamp chosen for the transaction. Not returned by default: see `  TransactionOptions.ReadOnly.return_read_timestamp  ` .

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

A precommit token is included in the response of a sessions.beginTransaction request if the read-write transaction is on a multiplexed session and a mutationKey was specified in the `  sessions.beginTransaction  ` . The precommit token with the highest sequence number from this transaction attempt should be passed to the `  Commit  ` request for this transaction.
