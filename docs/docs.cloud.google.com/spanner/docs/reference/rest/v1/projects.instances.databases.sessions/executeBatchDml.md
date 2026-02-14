  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ExecuteBatchDmlResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Statement](#Statement)
      - [JSON representation](#Statement.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Executes a batch of SQL DML statements. This method allows many statements to be run with lower latency than submitting them sequentially with `  sessions.executeSql  ` .

Statements are executed in sequential order. A request can succeed even if a statement fails. The `  ExecuteBatchDmlResponse.status  ` field in the response provides information about the statement that failed. Clients must inspect this field to determine whether an error occurred.

Execution stops after the first failed statement; the remaining statements are not executed.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeBatchDml  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the DML statements should be performed.

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
  &quot;transaction&quot;: {
    object (TransactionSelector)
  },
  &quot;statements&quot;: [
    {
      object (Statement)
    }
  ],
  &quot;seqno&quot;: string,
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;lastStatements&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transaction  `

`  object ( TransactionSelector  ` )

Required. The transaction to use. Must be a read-write transaction.

To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.

`  statements[]  `

`  object ( Statement  ` )

Required. The list of statements to execute in this batch. Statements are executed serially, such that the effects of statement `  i  ` are visible to statement `  i+1  ` . Each statement must be a DML statement. Execution stops at the first failed statement; the remaining statements are not executed.

Callers must provide at least one statement.

`  seqno  `

`  string ( int64 format)  `

Required. A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.

The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction might be aborted. Replays of previously handled requests yield the same response as the first execution.

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request.

`  lastStatements  `

`  boolean  `

Optional. If set to `  true  ` , this request marks the end of the transaction. After these statements execute, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.

Setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of statements shouldn't be assumed until a subsequent `  sessions.commit  ` call completes successfully.

### Response body

The response for `  sessions.executeBatchDml  ` . Contains a list of `  ResultSet  ` messages, one for each DML statement that has successfully executed, in the same order as the statements in the request. If a statement fails, the status in the response body identifies the cause of the failure.

To check for DML statements that failed, use the following approach:

1.  Check the status in the response message. The `  google.rpc.Code  ` enum value `  OK  ` indicates that all statements were executed successfully.
2.  If the status was not `  OK  ` , check the number of result sets in the response. If the response contains `  N  ` `  ResultSet  ` messages, then statement `  N+1  ` in the request failed.

Example 1:

  - Request: 5 DML statements, all executed successfully.
  - Response: 5 `  ResultSet  ` messages, with the status `  OK  ` .

Example 2:

  - Request: 5 DML statements. The third statement has a syntax error.
  - Response: 2 `  ResultSet  ` messages, and a syntax error ( `  INVALID_ARGUMENT  ` ) status. The number of `  ResultSet  ` messages indicates that the third statement failed, and the fourth and fifth statements were not executed.

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
  &quot;resultSets&quot;: [
    {
      object (ResultSet)
    }
  ],
  &quot;status&quot;: {
    object (Status)
  },
  &quot;precommitToken&quot;: {
    object (MultiplexedSessionPrecommitToken)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  resultSets[]  `

`  object ( ResultSet  ` )

One `  ResultSet  ` for each statement in the request that ran successfully, in the same order as the statements in the request. Each `  ResultSet  ` does not contain any rows. The `  ResultSetStats  ` in each `  ResultSet  ` contain the number of rows modified by the statement.

Only the first `  ResultSet  ` in the response contains valid `  ResultSetMetadata  ` .

`  status  `

`  object ( Status  ` )

If all DML statements are executed successfully, the status is `  OK  ` . Otherwise, the error status of the first failed statement.

`  precommitToken  `

`  object ( MultiplexedSessionPrecommitToken  ` )

Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt should be passed to the `  sessions.commit  ` request for this transaction.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

## Statement

A single DML statement.

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
  &quot;sql&quot;: string,
  &quot;params&quot;: {
    object
  },
  &quot;paramTypes&quot;: {
    string: {
      object (Type)
    },
    ...
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  sql  `

`  string  `

Required. The DML string.

`  params  `

`  object ( Struct  ` format)

Parameter names and values that bind to placeholders in the DML string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names can contain letters, numbers, and underscores.

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  paramTypes  `

`  map (key: string, value: object ( Type  ` ))

It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, `  paramTypes  ` can be used to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of `  Type  ` for more information about SQL types.
