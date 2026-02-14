  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
      - [JSON representation](#body.CommitResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [CommitStats](#CommitStats)
      - [JSON representation](#CommitStats.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Commits a transaction. The request includes the mutations to be applied to rows in the database.

`  sessions.commit  ` might return an `  ABORTED  ` error. This can occur at any time; commonly, the cause is conflicts with concurrent transactions. However, it can also happen for a variety of other reasons. If `  sessions.commit  ` returns `  ABORTED  ` , the caller should retry the transaction from the beginning, reusing the same session.

On very rare occasions, `  sessions.commit  ` might return `  UNKNOWN  ` . This can happen, for example, if the client job experiences a 1+ hour networking failure. At that point, Cloud Spanner has lost track of the transaction outcome and we recommend that you perform another read from the database to see the state of things as they are now.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:commit  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the transaction to be committed is running.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.write  `

### Request body

The request body contains data with the following structure:

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
  &quot;mutations&quot;: [
    {
      object (Mutation)
    }
  ],
  &quot;returnCommitStats&quot;: boolean,
  &quot;maxCommitDelay&quot;: string,
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  },

  // Union field transaction can be only one of the following:
  &quot;transactionId&quot;: string,
  &quot;singleUseTransaction&quot;: {
    object (TransactionOptions)
  }
  // End of list of possible types for union field transaction.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  mutations[]  `

`  object ( Mutation  ` )

The mutations to be executed when this transaction commits. All mutations are applied atomically, in the order they appear in this list.

`  returnCommitStats  `

`  boolean  `

If `  true  ` , then statistics related to the transaction is included in the `  CommitResponse  ` . Default value is `  false  ` .

`  maxCommitDelay  `

`  string ( Duration  ` format)

Optional. The amount of latency this request is configured to incur in order to improve throughput. If this field isn't set, Spanner assumes requests are relatively latency sensitive and automatically determines an appropriate delay time. You can specify a commit delay value between 0 and 500 ms.

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request.

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Optional. If the read-write transaction was executed on a multiplexed session, then you must include the precommit token with the highest sequence number received in this transaction attempt. Failing to do so results in a `  FailedPrecondition  ` error.

Union field `  transaction  ` . Required. The transaction in which to commit. `  transaction  ` can be only one of the following:

`  transactionId  `

`  string ( bytes format)  `

sessions.commit a previously-started transaction.

A base64-encoded string.

`  singleUseTransaction  `

`  object ( TransactionOptions  ` )

Execute mutations in a temporary transaction. Note that unlike commit of a previously-started transaction, commit with a temporary transaction is non-idempotent. That is, if the `  CommitRequest  ` is sent to Cloud Spanner more than once (for instance, due to retries in the application, or in the transport library), it's possible that the mutations are executed more than once. If this is undesirable, use `  sessions.beginTransaction  ` and `  sessions.commit  ` instead.

### Response body

The response for `  sessions.commit  ` .

If successful, the response body contains data with the following structure:

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

The statistics about this `  sessions.commit  ` . Not returned by default. For more information, see `  CommitRequest.return_commit_stats  ` .

Union field `  MultiplexedSessionRetry  ` . You must examine and retry the commit if the following is populated. `  MultiplexedSessionRetry  ` can be only one of the following:

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

If specified, transaction has not committed yet. You must retry the commit with the new precommit token.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## CommitStats

Additional statistics about a commit.

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

The total number of mutations for the transaction. Knowing the `  mutationCount  ` value can help you maximize the number of mutations in a transaction and minimize the number of API round trips. You can also monitor this value to prevent transactions from exceeding the system [limit](https://cloud.google.com/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . If the number of mutations exceeds the limit, the server returns [INVALID\_ARGUMENT](https://cloud.google.com/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) .
