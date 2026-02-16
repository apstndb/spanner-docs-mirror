## Tool: `       commit      `

Commit a transaction in a given session. \* If commit is finalizing the result of a DML statement then commit request should include latest precommit\_token returned by execute\_sql tool. \* If response to commit includes another precommit\_token then issue another commit call to finalize the transaction with the latest precommit\_token.

The following sample demonstrate how to use `  curl  ` to invoke the `  commit  ` MCP tool.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;commit&quot;,
    &quot;arguments&quot;: {
      // provide these details according to the tool&#39;s MCP specification
    }
  },
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;id&quot;: 1
}&#39;
                </code></pre></td>
</tr>
</tbody>
</table>

## Input Schema

The request for `  Commit  ` .

### CommitRequest

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
  &quot;session&quot;: string,
  &quot;transactionId&quot;: string,
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  session  `

`  string  `

Required. The session in which the transaction to be committed is running.

`  transactionId  `

`  string ( bytes format)  `

Required. The transaction in which to commit. Commit a previously-started transaction.

A base64-encoded string.

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Required. You must include the precommit token with the highest sequence number received in this transaction attempt. Failing to do so results in a `  FailedPrecondition  ` error.

### MultiplexedSessionPrecommitToken

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

## Output Schema

The response for `  Commit  ` .

### CommitResponse

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
  &quot;commitTimestamp&quot;: string,
  &quot;commitStats&quot;: {
    object (CommitStats)
  },
  &quot;snapshotTimestamp&quot;: string,

  // Union field MultiplexedSessionRetry can be only one of the following:
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
  // End of list of possible types for union field MultiplexedSessionRetry.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  commitTimestamp  `

`  string ( Timestamp  ` format)

The Cloud Spanner timestamp at which the transaction committed.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  commitStats  `

`  object ( CommitStats  ` )

The statistics about this `  Commit  ` . Not returned by default. For more information, see `  CommitRequest.return_commit_stats  ` .

`  snapshotTimestamp  `

`  string ( Timestamp  ` format)

If `  TransactionOptions.isolation_level  ` is set to `  IsolationLevel.REPEATABLE_READ  ` , then the snapshot timestamp is the timestamp at which all reads in the transaction ran. This timestamp is never returned.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

Union field `  MultiplexedSessionRetry  ` . You must examine and retry the commit if the following is populated. `  MultiplexedSessionRetry  ` can be only one of the following:

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

If specified, transaction has not committed yet. You must retry the commit with the new precommit token.

### Timestamp

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
  &quot;seconds&quot;: string,
  &quot;nanos&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  seconds  `

`  string ( int64 format)  `

Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).

`  nanos  `

`  integer  `

Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive.

### CommitStats

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
  &quot;mutationCount&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  mutationCount  `

`  string ( int64 format)  `

The total number of mutations for the transaction. Knowing the `  mutation_count  ` value can help you maximize the number of mutations in a transaction and minimize the number of API round trips. You can also monitor this value to prevent transactions from exceeding the system [limit](https://cloud.google.com/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . If the number of mutations exceeds the limit, the server returns [INVALID\_ARGUMENT](https://cloud.google.com/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) .

### MultiplexedSessionPrecommitToken

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

### Tool Annotations

Destructive Hint: ✅ | Idempotent Hint: ❌ | Read Only Hint: ❌ | Open World Hint: ❌
