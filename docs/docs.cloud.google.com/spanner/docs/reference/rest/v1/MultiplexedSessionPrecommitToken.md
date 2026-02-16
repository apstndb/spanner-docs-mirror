  - [JSON representation](#SCHEMA_REPRESENTATION)

When a read-write transaction is executed on a multiplexed session, this precommit token is sent back to the client as a part of the `  Transaction  ` message in the `  sessions.beginTransaction  ` response and also as a part of the `  ResultSet  ` and `  PartialResultSet  ` responses.

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
  &quot;precommitToken&quot;: string,
  &quot;seqNum&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  precommitToken  `

`  string ( bytes format)  `

Opaque precommit token.

A base64-encoded string.

`  seqNum  `

`  integer  `

An incrementing seq number is generated on every precommit token that is returned. Clients should remember the precommit token with the highest sequence number from the current transaction attempt.
