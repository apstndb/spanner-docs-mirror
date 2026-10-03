---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml
title: 'Method: projects.instances.databases.sessions.executeBatchDml'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.ExecuteBatchDmlResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.aspect)
- [Statement](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#Statement)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#Statement.SCHEMA_REPRESENTATION)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#try-it)

Executes a batch of SQL DML statements. This method allows many statements to be run with lower latency than submitting them sequentially with [`sessions.executeSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#google.spanner.v1.Spanner.ExecuteSql) .

Statements are executed in sequential order. A request can succeed even if a statement fails. The [`ExecuteBatchDmlResponse.status`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#body.ExecuteBatchDmlResponse.FIELDS.status) field in the response provides information about the statement that failed. Clients must inspect this field to determine whether an error occurred.

Execution stops after the first failed statement; the remaining statements are not executed.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:executeBatchDml`

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
<p>Required. The session in which the DML statements should be performed.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.write</code></li>
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
  "statements": [
    {
      object (Statement)
    }
  ],
  "seqno": string,
  "requestOptions": {
    object (RequestOptions)
  },
  "lastStatements": boolean
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `transaction`    | `object ( `[`TransactionSelector`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionSelector)` )` Required. The transaction to use. Must be a read-write transaction. To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.                                                                                                                                                                                                                  |
| `statements[]`   | `object ( `[`Statement`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#Statement)` )` Required. The list of statements to execute in this batch. Statements are executed serially, such that the effects of statement `i` are visible to statement `i+1` . Each statement must be a DML statement. Execution stops at the first failed statement; the remaining statements are not executed. Callers must provide at least one statement.                                                            |
| `seqno`          | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` Required. A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds. The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction might be aborted. Replays of previously handled requests yield the same response as the first execution. |
| `requestOptions` | `object ( `[`RequestOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/RequestOptions)` )` Common options for this request.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `lastStatements` | `boolean` Optional. If set to `true` , this request marks the end of the transaction. After these statements execute, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected. Setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of statements shouldn't be assumed until a subsequent `sessions.commit` call completes successfully.                  |

### Response body

The response for [`sessions.executeBatchDml`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#google.spanner.v1.Spanner.ExecuteBatchDml) . Contains a list of [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) messages, one for each DML statement that has successfully executed, in the same order as the statements in the request. If a statement fails, the status in the response body identifies the cause of the failure.

To check for DML statements that failed, use the following approach:

1.  Check the status in the response message. The [`google.rpc.Code`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Code) enum value `OK` indicates that all statements were executed successfully.
2.  If the status was not `OK` , check the number of result sets in the response. If the response contains `N` [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) messages, then statement `N+1` in the request failed.

Example 1:

- Request: 5 DML statements, all executed successfully.
- Response: 5 [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) messages, with the status `OK` .

Example 2:

- Request: 5 DML statements. The third statement has a syntax error.
- Response: 2 [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) messages, and a syntax error ( `INVALID_ARGUMENT` ) status. The number of [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) messages indicates that the third statement failed, and the fourth and fifth statements were not executed.

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "resultSets": [
    {
      object (ResultSet)
    }
  ],
  "status": {
    object (Status)
  },
  "precommitToken": {
    object (MultiplexedSessionPrecommitToken)
  }
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `resultSets[]`   | `object ( `[`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet)` )` One [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) for each statement in the request that ran successfully, in the same order as the statements in the request. Each [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) does not contain any rows. The [`ResultSetStats`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats) in each [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) contain the number of rows modified by the statement. Only the first [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) in the response contains valid [`ResultSetMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata) . |
| `status`         | `object ( `[`Status`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Status)` )` If all DML statements are executed successfully, the status is `OK` . Otherwise, the error status of the first failed statement.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `precommitToken` | `object ( `[`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MultiplexedSessionPrecommitToken)` )` Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt should be passed to the [`sessions.commit`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#google.spanner.v1.Spanner.Commit) request for this transaction.                                                                                                                                                                                                                                                                                                                                                                   |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## Statement

A single DML statement.

**JSON representation**

```
{
  "sql": string,
  "params": {
    object
  },
  "paramTypes": {
    string: {
      object (Type)
    },
    ...
  }
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sql`        | `string` Required. The DML string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `params`     | `object ( `[`Struct`](https://protobuf.dev/reference/protobuf/google.protobuf/#struct)` format)` Parameter names and values that bind to placeholders in the DML string. A parameter placeholder consists of the `@` character followed by the parameter name (for example, `@firstName` ). Parameter names can contain letters, numbers, and underscores. Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example: `"WHERE id > @msg_id AND id < @msg_id + 100"` It's an error to execute a SQL statement with unbound parameters.                                                                                                                          |
| `paramTypes` | `map (key: string, value: object ( `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type)` ))` It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `BYTES` and values of type `STRING` both appear in [`params`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml#Statement.FIELDS.params) as JSON strings. In these cases, `paramTypes` can be used to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type) for more information about SQL types. |
