  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Like `  sessions.executeSql  ` , except returns the result set as a stream. Unlike `  sessions.executeSql  ` , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.

The query string can be SQL or [Graph Query Language (GQL)](https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeStreamingSql  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the SQL query should be performed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.select  `

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
  &quot;sql&quot;: string,
  &quot;params&quot;: {
    object
  },
  &quot;paramTypes&quot;: {
    string: {
      object (Type)
    },
    ...
  },
  &quot;resumeToken&quot;: string,
  &quot;queryMode&quot;: enum (QueryMode),
  &quot;partitionToken&quot;: string,
  &quot;seqno&quot;: string,
  &quot;queryOptions&quot;: {
    object (QueryOptions)
  },
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;directedReadOptions&quot;: {
    object (DirectedReadOptions)
  },
  &quot;dataBoostEnabled&quot;: boolean,
  &quot;lastStatement&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transaction  `

`  object ( TransactionSelector  ` )

The transaction to use.

For queries, if none is provided, the default is a temporary read-only transaction with strong concurrency.

Standard DML statements require a read-write transaction. To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.

Partitioned DML requires an existing Partitioned DML transaction ID.

`  sql  `

`  string  `

Required. The SQL string.

`  params  `

`  object ( Struct  ` format)

Parameter names and values that bind to placeholders in the SQL string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names must conform to the naming requirements of identifiers as specified at <https://cloud.google.com/spanner/docs/lexical#identifiers> .

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  paramTypes  `

`  map (key: string, value: object ( Type  ` ))

It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, you can use `  paramTypes  ` to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of `  Type  ` for more information about SQL types.

`  resumeToken  `

`  string ( bytes format)  `

If this request is resuming a previously interrupted SQL statement execution, `  resumeToken  ` should be copied from the last `  PartialResultSet  ` yielded before the interruption. Doing this enables the new SQL statement execution to resume where the last one left off. The rest of the request parameters must exactly match the request that yielded this token.

A base64-encoded string.

`  queryMode  `

`  enum ( QueryMode  ` )

Used to control the amount of debugging information returned in `  ResultSetStats  ` . If `  partitionToken  ` is set, `  queryMode  ` can only be set to `  QueryMode.NORMAL  ` .

`  partitionToken  `

`  string ( bytes format)  `

If present, results are restricted to the specified partition previously created using `  sessions.partitionQuery  ` . There must be an exact match for the values of fields common to this message and the `  PartitionQueryRequest  ` message used to create this `  partitionToken  ` .

A base64-encoded string.

`  seqno  `

`  string ( int64 format)  `

A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.

The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction can be aborted. Replays of previously handled requests yield the same response as the first execution.

Required for DML statements. Ignored for queries.

`  queryOptions  `

`  object ( QueryOptions  ` )

Query optimizer configuration to use for the given query.

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request.

`  directedReadOptions  `

`  object ( DirectedReadOptions  ` )

Directed read options for this request.

`  dataBoostEnabled  `

`  boolean  `

If this is for a partitioned query and this field is set to `  true  ` , the request is executed with Spanner Data Boost independent compute resources.

If the field is set to `  true  ` but the request doesn't set `  partitionToken  ` , the API returns an `  INVALID_ARGUMENT  ` error.

`  lastStatement  `

`  boolean  `

Optional. If set to `  true  ` , this statement marks the end of the transaction. After this statement executes, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.

For DML statements, setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of a DML statement shouldn't be assumed until a subsequent `  sessions.commit  ` call completes successfully.

### Response body

If successful, the response body contains a stream of `  PartialResultSet  ` instances.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
