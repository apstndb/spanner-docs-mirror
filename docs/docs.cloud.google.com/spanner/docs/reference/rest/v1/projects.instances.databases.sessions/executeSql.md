---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql
title: 'Method: projects.instances.databases.sessions.executeSql'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it)

Executes an SQL statement, returning all results in a single reply. This method can't be used to return a result set larger than 10 MiB; if the query yields more data than that, the query fails with a `FAILED_PRECONDITION` error.

Operations inside read-write transactions might return `ABORTED` . If this occurs, the application should restart the transaction from the beginning. See [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction) for more details.

Larger result sets can be fetched in streaming fashion by calling [`sessions.executeStreamingSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql#google.spanner.v1.Spanner.ExecuteStreamingSql) instead.

The query string can be SQL or [Graph Query Language (GQL)](https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) .

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeSql`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the SQL query should be performed.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.select</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "transaction": {
    object (TransactionSelector)
  },
  "sql": string,
  "params": {
    object
  },
  "paramTypes": {
    string: {
      object (Type)
    },
    ...
  },
  "resumeToken": string,
  "queryMode": enum (QueryMode),
  "partitionToken": string,
  "seqno": string,
  "queryOptions": {
    object (QueryOptions)
  },
  "requestOptions": {
    object (RequestOptions)
  },
  "directedReadOptions": {
    object (DirectedReadOptions)
  },
  "dataBoostEnabled": boolean,
  "lastStatement": boolean
}
```

| Fields                |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|-----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `transaction`         | `object ( `[`TransactionSelector`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionSelector)` )` The transaction to use. For queries, if none is provided, the default is a temporary read-only transaction with strong concurrency. Standard DML statements require a read-write transaction. To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction. Partitioned DML requires an existing Partitioned DML transaction ID.                                                                                                                                                                                                          |
| `sql`                 | `string` Required. The SQL string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `params`              | `object ( `[`Struct`](https://protobuf.dev/reference/protobuf/google.protobuf/#struct)` format)` Parameter names and values that bind to placeholders in the SQL string. A parameter placeholder consists of the `@` character followed by the parameter name (for example, `@firstName` ). Parameter names must conform to the naming requirements of identifiers as specified at <https://cloud.google.com/spanner/docs/lexical#identifiers> . Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example: `"WHERE id > @msg_id AND id < @msg_id + 100"` It's an error to execute a SQL statement with unbound parameters.                                                               |
| `paramTypes`          | `map (key: string, value: object ( `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type)` ))` It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `BYTES` and values of type `STRING` both appear in [`params`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body.FIELDS.params) as JSON strings. In these cases, you can use `paramTypes` to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) for more information about SQL types.                         |
| `resumeToken`         | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` If this request is resuming a previously interrupted SQL statement execution, `resumeToken` should be copied from the last [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet) yielded before the interruption. Doing this enables the new SQL statement execution to resume where the last one left off. The rest of the request parameters must exactly match the request that yielded this token. A base64-encoded string.                                                                                                                                                                                             |
| `queryMode`           | `enum ( `[`QueryMode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryMode)` )` Used to control the amount of debugging information returned in [`ResultSetStats`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats) . If [`partitionToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body.FIELDS.partition_token) is set, [`queryMode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#body.request_body.FIELDS.query_mode) can only be set to [`QueryMode.NORMAL`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryMode#ENUM_VALUES.NORMAL) . |
| `partitionToken`      | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` If present, results are restricted to the specified partition previously created using `sessions.partitionQuery` . There must be an exact match for the values of fields common to this message and the `PartitionQueryRequest` message used to create this `partitionToken` . A base64-encoded string.                                                                                                                                                                                                                                                                                                                                                                   |
| `seqno`               | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds. The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction can be aborted. Replays of previously handled requests yield the same response as the first execution. Required for DML statements. Ignored for queries.                                                                                                                                                  |
| `queryOptions`        | `object ( `[`QueryOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/QueryOptions)` )` Query optimizer configuration to use for the given query.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `requestOptions`      | `object ( `[`RequestOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RequestOptions)` )` Common options for this request.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `directedReadOptions` | `object ( `[`DirectedReadOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/DirectedReadOptions)` )` Directed read options for this request.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `dataBoostEnabled`    | `boolean` If this is for a partitioned query and this field is set to `true` , the request is executed with Spanner Data Boost independent compute resources. If the field is set to `true` but the request doesn't set `partitionToken` , the API returns an `INVALID_ARGUMENT` error.                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `lastStatement`       | `boolean` Optional. If set to `true` , this statement marks the end of the transaction. After this statement executes, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected. For DML statements, setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of a DML statement shouldn't be assumed until a subsequent `sessions.commit` call completes successfully.                                                                                                                                                                               |

### Response body

If successful, the response body contains an instance of [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
