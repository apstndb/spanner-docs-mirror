---
name: documents/docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1
uri: https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1
title: Package google.spanner.v1
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Index

- [`Spanner`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner) (interface)
- [`BatchCreateSessionsRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchCreateSessionsRequest) (message)
- [`BatchCreateSessionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchCreateSessionsResponse) (message)
- [`BatchWriteRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteRequest) (message)
- [`BatchWriteRequest.MutationGroup`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteRequest.MutationGroup) (message)
- [`BatchWriteResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteResponse) (message)
- [`BeginTransactionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BeginTransactionRequest) (message)
- [`CommitRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitRequest) (message)
- [`CommitResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitResponse) (message)
- [`CommitResponse.CommitStats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitResponse.CommitStats) (message)
- [`CreateSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CreateSessionRequest) (message)
- [`DeleteSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DeleteSessionRequest) (message)
- [`DirectedReadOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions) (message)
- [`DirectedReadOptions.ExcludeReplicas`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ExcludeReplicas) (message)
- [`DirectedReadOptions.IncludeReplicas`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.IncludeReplicas) (message)
- [`DirectedReadOptions.ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ReplicaSelection) (message)
- [`DirectedReadOptions.ReplicaSelection.Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ReplicaSelection.Type) (enum)
- [`ExecuteBatchDmlRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlRequest) (message)
- [`ExecuteBatchDmlRequest.Statement`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlRequest.Statement) (message)
- [`ExecuteBatchDmlResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlResponse) (message)
- [`ExecuteSqlRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest) (message)
- [`ExecuteSqlRequest.QueryMode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryMode) (enum)
- [`ExecuteSqlRequest.QueryOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryOptions) (message)
- [`GetSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.GetSessionRequest) (message)
- [`KeyRange`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeyRange) (message)
- [`KeySet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeySet) (message)
- [`ListSessionsRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsRequest) (message)
- [`ListSessionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsResponse) (message)
- [`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken) (message)
- [`Mutation`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation) (message)
- [`Mutation.Delete`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Delete) (message)
- [`Mutation.Write`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write) (message)
- [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet) (message)
- [`Partition`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Partition) (message)
- [`PartitionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionOptions) (message)
- [`PartitionQueryRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionQueryRequest) (message)
- [`PartitionReadRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest) (message)
- [`PartitionResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionResponse) (message)
- [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) (message)
- [`PlanNode.ChildLink`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.ChildLink) (message)
- [`PlanNode.Kind`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.Kind) (enum)
- [`PlanNode.ShortRepresentation`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.ShortRepresentation) (message)
- [`QueryAdvisorResult`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryAdvisorResult) (message)
- [`QueryAdvisorResult.IndexAdvice`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryAdvisorResult.IndexAdvice) (message)
- [`QueryPlan`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryPlan) (message)
- [`ReadRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest) (message)
- [`RequestOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions) (message)
- [`RequestOptions.Priority`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions.Priority) (enum)
- [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) (message)
- [`ResultSetMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata) (message)
- [`ResultSetStats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetStats) (message)
- [`RollbackRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RollbackRequest) (message)
- [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) (message)
- [`StructType`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType) (message)
- [`StructType.Field`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType.Field) (message)
- [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) (message)
- [`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions) (message)
- [`TransactionOptions.IsolationLevel`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel) (enum)
- [`TransactionOptions.PartitionedDml`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.PartitionedDml) (message)
- [`TransactionOptions.ReadOnly`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadOnly) (message)
- [`TransactionOptions.ReadWrite`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadWrite) (message)
- [`TransactionOptions.ReadWrite.ReadLockMode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadWrite.ReadLockMode) (enum)
- [`TransactionSelector`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector) (message)
- [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) (message)
- [`TypeAnnotationCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeAnnotationCode) (enum)
- [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) (enum)

## Spanner

Cloud Spanner API

The Cloud Spanner API can be used to manage sessions and execute transactions on data stored in Cloud Spanner databases.

**BatchCreateSessions**

`rpc BatchCreateSessions( `[`BatchCreateSessionsRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchCreateSessionsRequest)` ) returns ( `[`BatchCreateSessionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchCreateSessionsResponse)` )`

Creates multiple new sessions.

This API can be used to initialize a session cache on the clients. See <https://goo.gl/TgSFN2> for best practices on session cache management.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**BatchWrite**

`rpc BatchWrite( `[`BatchWriteRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteRequest)` ) returns ( `[`BatchWriteResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteResponse)` )`

Batches the supplied mutation groups in a collection of efficient transactions. All mutations in a group are committed atomically. However, mutations across groups can be committed non-atomically in an unspecified order and thus, they must be independent of each other. Partial failure is possible, that is, some groups might have been committed successfully, while some might have failed. The results of individual batches are streamed into the response as the batches are applied.

`BatchWrite` requests are not replay protected, meaning that each mutation group can be applied more than once. Replays of non-idempotent mutations can have undesirable effects. For example, replays of an insert mutation can produce an already exists error or if you use generated or commit timestamp-based keys, it can result in additional rows being added to the mutation's table. We recommend structuring your mutation groups to be idempotent to avoid this issue.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**BeginTransaction**

`rpc BeginTransaction( `[`BeginTransactionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BeginTransactionRequest)` ) returns ( `[`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction)` )`

Begins a new transaction. This step can often be skipped: [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) , [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) and [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) can begin a new transaction as a side-effect.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**Commit**

`rpc Commit( `[`CommitRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitRequest)` ) returns ( `[`CommitResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitResponse)` )`

Commits a transaction. The request includes the mutations to be applied to rows in the database.

`Commit` might return an `ABORTED` error. This can occur at any time; commonly, the cause is conflicts with concurrent transactions. However, it can also happen for a variety of other reasons. If `Commit` returns `ABORTED` , the caller should retry the transaction from the beginning, reusing the same session.

On very rare occasions, `Commit` might return `UNKNOWN` . This can happen, for example, if the client job experiences a 1+ hour networking failure. At that point, Cloud Spanner has lost track of the transaction outcome and we recommend that you perform another read from the database to see the state of things as they are now.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**CreateSession**

`rpc CreateSession( `[`CreateSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CreateSessionRequest)` ) returns ( `[`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session)` )`

Creates a new session. A session can be used to perform transactions that read and/or modify data in a Cloud Spanner database. Sessions are meant to be reused for many consecutive transactions.

Sessions can only execute one transaction at a time. To execute multiple concurrent read-write/write-only transactions, create multiple sessions. Note that standalone reads and queries use a transaction internally, and count toward the one transaction limit.

Active sessions use additional server resources, so it's a good idea to delete idle and unneeded sessions. Aside from explicit deletes, Cloud Spanner can delete sessions when no operations are sent for more than an hour. If a session is deleted, requests to it return `NOT_FOUND` .

Idle sessions can be kept alive by sending a trivial SQL query periodically, for example, `"SELECT 1"` .

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**DeleteSession**

`rpc DeleteSession( `[`DeleteSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DeleteSessionRequest)` ) returns ( `[`Empty`](https://protobuf.dev/reference/protobuf/google.protobuf/#empty)` )`

Ends a session, releasing server resources associated with it. This asynchronously triggers the cancellation of any operations that are running with this session.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**ExecuteBatchDml**

`rpc ExecuteBatchDml( `[`ExecuteBatchDmlRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlRequest)` ) returns ( `[`ExecuteBatchDmlResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlResponse)` )`

Executes a batch of SQL DML statements. This method allows many statements to be run with lower latency than submitting them sequentially with [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) .

Statements are executed in sequential order. A request can succeed even if a statement fails. The [`ExecuteBatchDmlResponse.status`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlResponse.FIELDS.google.rpc.Status.google.spanner.v1.ExecuteBatchDmlResponse.status) field in the response provides information about the statement that failed. Clients must inspect this field to determine whether an error occurred.

Execution stops after the first failed statement; the remaining statements are not executed.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**ExecuteSql**

`rpc ExecuteSql( `[`ExecuteSqlRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest)` ) returns ( `[`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet)` )`

Executes an SQL statement, returning all results in a single reply. This method can't be used to return a result set larger than 10 MiB; if the query yields more data than that, the query fails with a `FAILED_PRECONDITION` error.

Operations inside read-write transactions might return `ABORTED` . If this occurs, the application should restart the transaction from the beginning. See [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) for more details.

Larger result sets can be fetched in streaming fashion by calling [`ExecuteStreamingSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) instead.

The query string can be SQL or [Graph Query Language (GQL)](https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) .

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**ExecuteStreamingSql**

`rpc ExecuteStreamingSql( `[`ExecuteSqlRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest)` ) returns ( `[`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet)` )`

Like [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) , except returns the result set as a stream. Unlike [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.

The query string can be SQL or [Graph Query Language (GQL)](https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro) .

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**GetSession**

`rpc GetSession( `[`GetSessionRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.GetSessionRequest)` ) returns ( `[`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session)` )`

Gets a session. Returns `NOT_FOUND` if the session doesn't exist. This is mainly useful for determining whether a session is still alive.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**ListSessions**

`rpc ListSessions( `[`ListSessionsRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsRequest)` ) returns ( `[`ListSessionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsResponse)` )`

Lists all sessions in a given database.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**PartitionQuery**

`rpc PartitionQuery( `[`PartitionQueryRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionQueryRequest)` ) returns ( `[`PartitionResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionResponse)` )`

Creates a set of partition tokens that can be used to execute a query operation in parallel. Each of the returned partition tokens can be used by [`ExecuteStreamingSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) to specify a subset of the query result to read. The same session and read-only transaction must be used by the `PartitionQueryRequest` used to create the partition tokens and the `ExecuteSqlRequests` that use the partition tokens.

Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the query, and the whole operation must be restarted from the beginning.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**PartitionRead**

`rpc PartitionRead( `[`PartitionReadRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest)` ) returns ( `[`PartitionResponse`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionResponse)` )`

Creates a set of partition tokens that can be used to execute a read operation in parallel. Each of the returned partition tokens can be used by [`StreamingRead`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.StreamingRead) to specify a subset of the read result to read. The same session and read-only transaction must be used by the `PartitionReadRequest` used to create the partition tokens and the `ReadRequests` that use the partition tokens. There are no ordering guarantees on rows returned among the returned partition tokens, or even within each individual `StreamingRead` call issued with a `partition_token` .

Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the read, and the whole operation must be restarted from the beginning.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**Read**

`rpc Read( `[`ReadRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest)` ) returns ( `[`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet)` )`

Reads rows from the database using key lookups and scans, as a simple key/value style alternative to [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) . This method can't be used to return a result set larger than 10 MiB; if the read matches more data than that, the read fails with a `FAILED_PRECONDITION` error.

Reads inside read-write transactions might return `ABORTED` . If this occurs, the application should restart the transaction from the beginning. See [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) for more details.

Larger result sets can be yielded in streaming fashion by calling [`StreamingRead`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.StreamingRead) instead.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**Rollback**

`rpc Rollback( `[`RollbackRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RollbackRequest)` ) returns ( `[`Empty`](https://protobuf.dev/reference/protobuf/google.protobuf/#empty)` )`

Rolls back a transaction, releasing any locks it holds. It's a good idea to call this for any transaction that includes one or more [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) or [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) requests and ultimately decides not to commit.

`Rollback` returns `OK` if it successfully aborts the transaction, the transaction was already aborted, or the transaction isn't found. `Rollback` never returns `ABORTED` .

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

**StreamingRead**

`rpc StreamingRead( `[`ReadRequest`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest)` ) returns ( `[`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet)` )`

Like [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) , except returns the result set as a stream. Unlike [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.

Authorization scopes  
Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## BatchCreateSessionsRequest

The request for [`BatchCreateSessions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BatchCreateSessions) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>database</code></td>
<td><p><code>string</code></p>
<p>Required. The database in which the new sessions are created.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.sessions.create</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>session_template</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session"><code>Session</code></a></p>
<p>Parameters to apply to each created session.</p></td>
</tr>
<tr class="odd">
<td><code>session_count</code></td>
<td><p><code>int32</code></p>
<p>Required. The number of sessions to be created in this batch call. At least one session is created. The API can return fewer than the requested number of sessions. If a specific number of sessions are desired, the client can make additional calls to <code>BatchCreateSessions</code> (adjusting <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchCreateSessionsRequest.FIELDS.int32.google.spanner.v1.BatchCreateSessionsRequest.session_count"><code>session_count</code></a> as necessary).</p></td>
</tr>
</tbody>
</table>

## BatchCreateSessionsResponse

The response for [`BatchCreateSessions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BatchCreateSessions) .

| Fields      |                                                                                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `session[]` | [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) The freshly created sessions. |

## BatchWriteRequest

The request for [`BatchWrite`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BatchWrite) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the batch request is to be run.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.write</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request.</p></td>
</tr>
<tr class="odd">
<td><code>mutation_groups[]</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BatchWriteRequest.MutationGroup"><code>MutationGroup</code></a></p>
<p>Required. The groups of mutations to be applied.</p></td>
</tr>
<tr class="even">
<td><code>exclude_txn_from_change_streams</code></td>
<td><p><code>bool</code></p>
<p>Optional. If you don't set the <code>exclude_txn_from_change_streams</code> option or if it's set to <code>false</code> , then any change streams monitoring columns modified by transactions will capture the updates made within that transaction.</p></td>
</tr>
</tbody>
</table>

## MutationGroup

A group of mutations to be committed together. Related mutations should be placed in a group. For example, two mutations inserting rows with the same primary key prefix in both parent and child tables are related.

| Fields        |                                                                                                                                                            |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mutations[]` | [`Mutation`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation) Required. The mutations in this group. |

## BatchWriteResponse

The result of applying a batch of mutations.

| Fields             |                                                                                                                                                                                                                                                                                                                                                                                                                             |
|--------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `indexes[]`        | `int32` The mutation groups applied in this batch. The values index into the `mutation_groups` field in the corresponding `BatchWriteRequest` .                                                                                                                                                                                                                                                                             |
| `status`           | [`Status`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.rpc#google.rpc.Status) An `OK` status indicates success. Any other status indicates a failure.                                                                                                                                                                                                                                                   |
| `commit_timestamp` | [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) The commit timestamp of the transaction that applied this batch. Present if status is OK and the mutation groups were applied, absent otherwise. For mutation groups with conditions, a status=OK and missing commit_timestamp means that the mutation groups were not applied due to the condition not being satisfied after evaluation. |

## BeginTransactionRequest

The request for [`BeginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BeginTransaction) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the transaction runs.</p>
<p>Authorization requires one or more of the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permissions on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.beginReadOnlyTransaction</code></li>
<li><code>spanner.databases.beginOrRollbackReadWriteTransaction</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions"><code>TransactionOptions</code></a></p>
<p>Required. Options for the new transaction.</p></td>
</tr>
<tr class="odd">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request. Priority is ignored for this request. Setting the priority in this <code>request_options</code> struct doesn't do anything. To set the priority for a transaction, set it on the reads and writes that are part of this transaction instead.</p></td>
</tr>
<tr class="even">
<td><code>mutation_key</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation"><code>Mutation</code></a></p>
<p>Optional. Required for read-write transactions on a multiplexed session that commit mutations but don't perform any reads or queries. You must randomly select one of the mutations from the mutation set and send it as a part of this request.</p></td>
</tr>
</tbody>
</table>

## CommitRequest

The request for [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the transaction to be committed is running.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.write</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>mutations[]</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation"><code>Mutation</code></a></p>
<p>The mutations to be executed when this transaction commits. All mutations are applied atomically, in the order they appear in this list.</p></td>
</tr>
<tr class="odd">
<td><code>return_commit_stats</code></td>
<td><p><code>bool</code></p>
<p>If <code>true</code> , then statistics related to the transaction is included in the <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitResponse.FIELDS.google.spanner.v1.CommitResponse.CommitStats.google.spanner.v1.CommitResponse.commit_stats"><code>CommitResponse</code></a> . Default value is <code>false</code> .</p></td>
</tr>
<tr class="even">
<td><code>max_commit_delay</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#duration"><code>Duration</code></a></p>
<p>Optional. The amount of latency this request is configured to incur in order to improve throughput. If this field isn't set, Spanner assumes requests are relatively latency sensitive and automatically determines an appropriate delay time. You can specify a commit delay value between 0 and 500 ms.</p></td>
</tr>
<tr class="odd">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request.</p></td>
</tr>
<tr class="even">
<td><code>precommit_token</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken"><code>MultiplexedSessionPrecommitToken</code></a></p>
<p>Optional. If the read-write transaction was executed on a multiplexed session, then you must include the precommit token with the highest sequence number received in this transaction attempt. Failing to do so results in a <code>FailedPrecondition</code> error.</p></td>
</tr>
<tr class="odd">
<td>Union field <code>transaction</code> . Required. The transaction in which to commit. <code>transaction</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>transaction_id</code></td>
<td><p><code>bytes</code></p>
<p>Commit a previously-started transaction.</p></td>
</tr>
<tr class="odd">
<td><code>single_use_transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions"><code>TransactionOptions</code></a></p>
<p>Execute mutations in a temporary transaction. Note that unlike commit of a previously-started transaction, commit with a temporary transaction is non-idempotent. That is, if the <code>CommitRequest</code> is sent to Cloud Spanner more than once (for instance, due to retries in the application, or in the transport library), it's possible that the mutations are executed more than once. If this is undesirable, use <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BeginTransaction"><code>BeginTransaction</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit"><code>Commit</code></a> instead.</p></td>
</tr>
</tbody>
</table>

## CommitResponse

The response for [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) .

| Fields                                                                                                                                                                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `commit_timestamp`                                                                                                                                                       | [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) The Cloud Spanner timestamp at which the transaction committed.                                                                                                                                                                                                                                                                                                    |
| `commit_stats`                                                                                                                                                           | [`CommitStats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitResponse.CommitStats) The statistics about this `Commit` . Not returned by default. For more information, see [`CommitRequest.return_commit_stats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.CommitRequest.FIELDS.bool.google.spanner.v1.CommitRequest.return_commit_stats) . |
| Union field `MultiplexedSessionRetry` . You must examine and retry the commit if the following is populated. `MultiplexedSessionRetry` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `precommit_token`                                                                                                                                                        | [`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken) If specified, transaction has not committed yet. You must retry the commit with the new precommit token.                                                                                                                                                                         |

## CommitStats

Additional statistics about a commit.

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `mutation_count` | `int64` The total number of mutations for the transaction. Knowing the `mutation_count` value can help you maximize the number of mutations in a transaction and minimize the number of API round trips. You can also monitor this value to prevent transactions from exceeding the system [limit](https://cloud.google.com/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . If the number of mutations exceeds the limit, the server returns [INVALID_ARGUMENT](https://cloud.google.com/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) . |

## CreateSessionRequest

The request for [`CreateSession`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.CreateSession) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>database</code></td>
<td><p><code>string</code></p>
<p>Required. The database in which the new session is created.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.sessions.create</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>session</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session"><code>Session</code></a></p>
<p>Required. The session to create.</p></td>
</tr>
</tbody>
</table>

## DeleteSessionRequest

The request for [`DeleteSession`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.DeleteSession) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the session to delete.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.sessions.delete</code></li>
</ul></td>
</tr>
</tbody>
</table>

## DirectedReadOptions

The `DirectedReadOptions` can be used to indicate which replicas or regions should be used for non-transactional reads or queries.

`DirectedReadOptions` can only be specified for a read-only transaction, otherwise the API returns an `INVALID_ARGUMENT` error.

| Fields                                                                                                                                                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `replicas` . Required. At most one of either `include_replicas` or `exclude_replicas` should be present in the message. `replicas` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `include_replicas`                                                                                                                                                               | [`IncludeReplicas`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.IncludeReplicas) `Include_replicas` indicates the order of replicas (as they appear in this list) to process the request. If `auto_failover_disabled` is set to `true` and all replicas are exhausted without finding a healthy replica, Spanner waits for a replica in the list to become available, requests might fail due to `DEADLINE_EXCEEDED` errors. |
| `exclude_replicas`                                                                                                                                                               | [`ExcludeReplicas`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ExcludeReplicas) `Exclude_replicas` indicates that specified replicas should be excluded from serving requests. Spanner doesn't route requests to the replicas in this list.                                                                                                                                                                                 |

## ExcludeReplicas

An ExcludeReplicas contains a repeated set of ReplicaSelection that should be excluded from serving requests.

| Fields                 |                                                                                                                                                                                             |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replica_selections[]` | [`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ReplicaSelection) The directed read replica selector. |

## IncludeReplicas

An `IncludeReplicas` contains a repeated set of `ReplicaSelection` which indicates the order in which replicas should be considered.

| Fields                   |                                                                                                                                                                                                   |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replica_selections[]`   | [`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ReplicaSelection) The directed read replica selector.       |
| `auto_failover_disabled` | `bool` If `true` , Spanner doesn't route requests to a replica outside the \< `include_replicas` list when all of the specified replicas are unavailable or unhealthy. Default value is `false` . |

## ReplicaSelection

The directed read replica selector. Callers must provide one or more of the following fields for replica selection:

- `location` - The location must be one of the regions within the multi-region configuration of your database.
- `type` - The type of the replica.

Some examples of using replica_selectors are:

- `location:us-east1` --\> The "us-east1" replica(s) of any available type is used to process the request.
- `type:READ_ONLY` --\> The "READ_ONLY" type replica(s) in the nearest available location are used to process the request.
- `location:us-east1 type:READ_ONLY` --\> The "READ_ONLY" type replica(s) in location "us-east1" is used to process the request.

| Fields     |                                                                                                                                                                       |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `location` | `string` The location or region of the serving requests, for example, "us-east1".                                                                                     |
| `type`     | [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions.ReplicaSelection.Type) The type of replica. |

## Type

Indicates the type of replica.

| Enums              |                                                     |
|--------------------|-----------------------------------------------------|
| `TYPE_UNSPECIFIED` | Not specified.                                      |
| `READ_WRITE`       | Read-write replicas support both reads and writes.  |
| `READ_ONLY`        | Read-only replicas only support reads (not writes). |

## ExecuteBatchDmlRequest

The request for [`ExecuteBatchDml`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteBatchDml) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
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
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector"><code>TransactionSelector</code></a></p>
<p>Required. The transaction to use. Must be a read-write transaction.</p>
<p>To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.</p></td>
</tr>
<tr class="odd">
<td><code>statements[]</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlRequest.Statement"><code>Statement</code></a></p>
<p>Required. The list of statements to execute in this batch. Statements are executed serially, such that the effects of statement <code>i</code> are visible to statement <code>i+1</code> . Each statement must be a DML statement. Execution stops at the first failed statement; the remaining statements are not executed.</p>
<p>Callers must provide at least one statement.</p></td>
</tr>
<tr class="even">
<td><code>seqno</code></td>
<td><p><code>int64</code></p>
<p>Required. A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.</p>
<p>The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction might be aborted. Replays of previously handled requests yield the same response as the first execution.</p></td>
</tr>
<tr class="odd">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request.</p></td>
</tr>
<tr class="even">
<td><code>last_statements</code></td>
<td><p><code>bool</code></p>
<p>Optional. If set to <code>true</code> , this request marks the end of the transaction. After these statements execute, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.</p>
<p>Setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of statements shouldn't be assumed until a subsequent <code>Commit</code> call completes successfully.</p></td>
</tr>
</tbody>
</table>

## Statement

A single DML statement.

| Fields        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|---------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sql`         | `string` Required. The DML string.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `params`      | [`Struct`](https://protobuf.dev/reference/protobuf/google.protobuf/#struct) Parameter names and values that bind to placeholders in the DML string. A parameter placeholder consists of the `@` character followed by the parameter name (for example, `@firstName` ). Parameter names can contain letters, numbers, and underscores. Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example: `"WHERE id > @msg_id AND id < @msg_id + 100"` It's an error to execute a SQL statement with unbound parameters.                                                                                                                                                                                                                                                                    |
| `param_types` | `map<string, `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type)` >` It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `BYTES` and values of type `STRING` both appear in [`params`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteBatchDmlRequest.Statement.FIELDS.google.protobuf.Struct.google.spanner.v1.ExecuteBatchDmlRequest.Statement.params) as JSON strings. In these cases, `param_types` can be used to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) for more information about SQL types. |

## ExecuteBatchDmlResponse

The response for [`ExecuteBatchDml`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteBatchDml) . Contains a list of [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) messages, one for each DML statement that has successfully executed, in the same order as the statements in the request. If a statement fails, the status in the response body identifies the cause of the failure.

To check for DML statements that failed, use the following approach:

1.  Check the status in the response message. The [`google.rpc.Code`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.rpc#google.rpc.Code) enum value `OK` indicates that all statements were executed successfully.
2.  If the status was not `OK` , check the number of result sets in the response. If the response contains `N` [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) messages, then statement `N+1` in the request failed.

Example 1:

- Request: 5 DML statements, all executed successfully.
- Response: 5 [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) messages, with the status `OK` .

Example 2:

- Request: 5 DML statements. The third statement has a syntax error.
- Response: 2 [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) messages, and a syntax error ( `INVALID_ARGUMENT` ) status. The number of [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) messages indicates that the third statement failed, and the fourth and fifth statements were not executed.

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `result_sets[]`   | [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) One [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) for each statement in the request that ran successfully, in the same order as the statements in the request. Each [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) does not contain any rows. The [`ResultSetStats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetStats) in each [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) contain the number of rows modified by the statement. Only the first [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) in the response contains valid [`ResultSetMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata) . |
| `status`          | [`Status`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.rpc#google.rpc.Status) If all DML statements are executed successfully, the status is `OK` . Otherwise, the error status of the first failed statement.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `precommit_token` | [`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken) Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt should be passed to the [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) request for this transaction.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

## ExecuteSqlRequest

The request for [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) and [`ExecuteStreamingSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
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
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector"><code>TransactionSelector</code></a></p>
<p>The transaction to use.</p>
<p>For queries, if none is provided, the default is a temporary read-only transaction with strong concurrency.</p>
<p>Standard DML statements require a read-write transaction. To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.</p>
<p>Partitioned DML requires an existing Partitioned DML transaction ID.</p></td>
</tr>
<tr class="odd">
<td><code>sql</code></td>
<td><p><code>string</code></p>
<p>Required. The SQL string.</p></td>
</tr>
<tr class="even">
<td><code>params</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a></p>
<p>Parameter names and values that bind to placeholders in the SQL string.</p>
<p>A parameter placeholder consists of the <code>@</code> character followed by the parameter name (for example, <code>@firstName</code> ). Parameter names must conform to the naming requirements of identifiers as specified at <a href="https://cloud.google.com/spanner/docs/lexical#identifiers">https://cloud.google.com/spanner/docs/lexical#identifiers</a> .</p>
<p>Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:</p>
<p><code>"WHERE id &gt; @msg_id AND id &lt; @msg_id + 100"</code></p>
<p>It's an error to execute a SQL statement with unbound parameters.</p></td>
</tr>
<tr class="odd">
<td><code>param_types</code></td>
<td><p><code>map&lt;string, </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type"><code>Type</code></a><code> &gt;</code></p>
<p>It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type <code>BYTES</code> and values of type <code>STRING</code> both appear in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.protobuf.Struct.google.spanner.v1.ExecuteSqlRequest.params"><code>params</code></a> as JSON strings.</p>
<p>In these cases, you can use <code>param_types</code> to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type"><code>Type</code></a> for more information about SQL types.</p></td>
</tr>
<tr class="even">
<td><code>resume_token</code></td>
<td><p><code>bytes</code></p>
<p>If this request is resuming a previously interrupted SQL statement execution, <code>resume_token</code> should be copied from the last <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet"><code>PartialResultSet</code></a> yielded before the interruption. Doing this enables the new SQL statement execution to resume where the last one left off. The rest of the request parameters must exactly match the request that yielded this token.</p></td>
</tr>
<tr class="odd">
<td><code>query_mode</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryMode"><code>QueryMode</code></a></p>
<p>Used to control the amount of debugging information returned in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetStats"><code>ResultSetStats</code></a> . If <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.bytes.google.spanner.v1.ExecuteSqlRequest.partition_token"><code>partition_token</code></a> is set, <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.spanner.v1.ExecuteSqlRequest.QueryMode.google.spanner.v1.ExecuteSqlRequest.query_mode"><code>query_mode</code></a> can only be set to <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryMode.ENUM_VALUES.google.spanner.v1.ExecuteSqlRequest.QueryMode.NORMAL"><code>QueryMode.NORMAL</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>partition_token</code></td>
<td><p><code>bytes</code></p>
<p>If present, results are restricted to the specified partition previously created using <code>PartitionQuery</code> . There must be an exact match for the values of fields common to this message and the <code>PartitionQueryRequest</code> message used to create this <code>partition_token</code> .</p></td>
</tr>
<tr class="odd">
<td><code>seqno</code></td>
<td><p><code>int64</code></p>
<p>A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.</p>
<p>The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction can be aborted. Replays of previously handled requests yield the same response as the first execution.</p>
<p>Required for DML statements. Ignored for queries.</p></td>
</tr>
<tr class="even">
<td><code>query_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryOptions"><code>QueryOptions</code></a></p>
<p>Query optimizer configuration to use for the given query.</p></td>
</tr>
<tr class="odd">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request.</p></td>
</tr>
<tr class="even">
<td><code>directed_read_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions"><code>DirectedReadOptions</code></a></p>
<p>Directed read options for this request.</p></td>
</tr>
<tr class="odd">
<td><code>data_boost_enabled</code></td>
<td><p><code>bool</code></p>
<p>If this is for a partitioned query and this field is set to <code>true</code> , the request is executed with Spanner Data Boost independent compute resources.</p>
<p>If the field is set to <code>true</code> but the request doesn't set <code>partition_token</code> , the API returns an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
<tr class="even">
<td><code>last_statement</code></td>
<td><p><code>bool</code></p>
<p>Optional. If set to <code>true</code> , this statement marks the end of the transaction. After this statement executes, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.</p>
<p>For DML statements, setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of a DML statement shouldn't be assumed until a subsequent <code>Commit</code> call completes successfully.</p></td>
</tr>
</tbody>
</table>

## QueryMode

Mode in which the statement must be processed.

| Enums                 |                                                                                                                                                                                                                                                        |
|-----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `NORMAL`              | The default mode. Only the statement results are returned.                                                                                                                                                                                             |
| `PLAN`                | This mode returns only the query plan, without any results or execution statistics information.                                                                                                                                                        |
| `PROFILE`             | This mode returns the query plan, overall execution statistics, operator level execution statistics along with the results. This has a performance overhead compared to the other modes. It isn't recommended to use this mode for production traffic. |
| `WITH_STATS`          | This mode returns the overall (but not operator-level) execution statistics along with the results.                                                                                                                                                    |
| `WITH_PLAN_AND_STATS` | This mode returns the query plan, overall (but not operator-level) execution statistics along with the results.                                                                                                                                        |

## QueryOptions

Query optimizer configuration.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>optimizer_version</code></td>
<td><p><code>string</code></p>
<p>An option to control the selection of optimizer version.</p>
<p>This parameter allows individual queries to pick different query optimizer versions.</p>
<p>Specifying <code>latest</code> as a value instructs Cloud Spanner to use the latest supported query optimizer version. If not specified, Cloud Spanner uses the optimizer version set at the database level options. Any other positive integer (from the list of supported optimizer versions) overrides the default optimizer version for query execution.</p>
<p>The list of supported optimizer versions can be queried from <code>SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS</code> .</p>
<p>Executing a SQL statement with an invalid optimizer version fails with an <code>INVALID_ARGUMENT</code> error.</p>
<p>See <a href="https://cloud.google.com/spanner/docs/query-optimizer/manage-query-optimizer">https://cloud.google.com/spanner/docs/query-optimizer/manage-query-optimizer</a> for more information on managing the query optimizer.</p>
<p>The <code>optimizer_version</code> statement hint has precedence over this setting.</p></td>
</tr>
<tr class="even">
<td><code>optimizer_statistics_package</code></td>
<td><p><code>string</code></p>
<p>An option to control the selection of optimizer statistics package.</p>
<p>This parameter allows individual queries to use a different query optimizer statistics package.</p>
<p>Specifying <code>latest</code> as a value instructs Cloud Spanner to use the latest generated statistics package. If not specified, Cloud Spanner uses the statistics package set at the database level options, or the latest package if the database option isn't set.</p>
<p>The statistics package requested by the query has to be exempt from garbage collection. This can be achieved with the following DDL statement:</p>
<pre class="sql"><code>ALTER STATISTICS &lt;package_name&gt; SET OPTIONS (allow_gc=false)</code></pre>
<p>The list of available statistics packages can be queried from <code>INFORMATION_SCHEMA.SPANNER_STATISTICS</code> .</p>
<p>Executing a SQL statement with an invalid optimizer statistics package or with a statistics package that allows garbage collection fails with an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
</tbody>
</table>

## GetSessionRequest

The request for [`GetSession`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.GetSession) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the session to retrieve.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.sessions.get</code></li>
</ul></td>
</tr>
</tbody>
</table>

## KeyRange

KeyRange represents a range of rows in a table or index.

A range has a start key and an end key. These keys can be open or closed, indicating if the range includes rows with that key.

Keys are represented by lists, where the ith value in the list corresponds to the ith component of the table or index primary key. Individual values are encoded as described [`here`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) .

For example, consider the following table definition:

```
CREATE TABLE UserEvents (
  UserName STRING(MAX),
  EventDate STRING(10)
) PRIMARY KEY(UserName, EventDate);
```

The following keys name rows in this table:

```
["Bob", "2014-09-23"]
["Alfred", "2015-06-12"]
```

Since the `UserEvents` table's `PRIMARY KEY` clause names two columns, each `UserEvents` key has two elements; the first is the `UserName` , and the second is the `EventDate` .

Key ranges with multiple components are interpreted lexicographically by component using the table or index key's declared sort order. For example, the following range returns all events for user `"Bob"` that occurred in the year 2015:

```
"start_closed": ["Bob", "2015-01-01"]
"end_closed": ["Bob", "2015-12-31"]
```

Start and end keys can omit trailing key components. This affects the inclusion and exclusion of rows that exactly match the provided key components: if the key is closed, then rows that exactly match the provided components are included; if the key is open, then rows that exactly match are not included.

For example, the following range includes all events for `"Bob"` that occurred during and after the year 2000:

```
"start_closed": ["Bob", "2000-01-01"]
"end_closed": ["Bob"]
```

The next example retrieves all events for `"Bob"` :

```
"start_closed": ["Bob"]
"end_closed": ["Bob"]
```

To retrieve events before the year 2000:

```
"start_closed": ["Bob"]
"end_open": ["Bob", "2000-01-01"]
```

The following range includes all rows in the table:

```
"start_closed": []
"end_closed": []
```

This range returns all users whose `UserName` begins with any character from A to C:

```
"start_closed": ["A"]
"end_open": ["D"]
```

This range returns all users whose `UserName` begins with B:

```
"start_closed": ["B"]
"end_open": ["C"]
```

Key ranges honor column sort order. For example, suppose a table is defined as follows:

```
CREATE TABLE DescendingSortedTable {
  Key INT64,
  ...
) PRIMARY KEY(Key DESC);
```

The following range retrieves all rows with key values between 1 and 100 inclusive:

```
"start_closed": ["100"]
"end_closed": ["1"]
```

Note that 100 is passed as the start, and 1 is passed as the end, because `Key` is a descending column in the schema.

| Fields                                                                                                                                             |                                                                                                                                                                                                                        |
|----------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `start_key_type` . The start key must be provided. It can be either closed or open. `start_key_type` can be only one of the following: |                                                                                                                                                                                                                        |
| `start_closed`                                                                                                                                     | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) If the start is closed, then the range includes all rows whose first `len(start_closed)` key columns exactly match `start_closed` . |
| `start_open`                                                                                                                                       | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) If the start is open, then the range excludes rows whose first `len(start_open)` key columns exactly match `start_open` .           |
| Union field `end_key_type` . The end key must be provided. It can be either closed or open. `end_key_type` can be only one of the following:       |                                                                                                                                                                                                                        |
| `end_closed`                                                                                                                                       | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) If the end is closed, then the range includes all rows whose first `len(end_closed)` key columns exactly match `end_closed` .       |
| `end_open`                                                                                                                                         | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) If the end is open, then the range excludes rows whose first `len(end_open)` key columns exactly match `end_open` .                 |

## KeySet

`KeySet` defines a collection of Cloud Spanner keys and/or key ranges. All the keys are expected to be in the same table or index. The keys need not be sorted in any particular way.

If the same key is specified multiple times in the set (for example if two ranges, two keys, or a key and a range overlap), Cloud Spanner behaves as if the key were only specified once.

| Fields     |                                                                                                                                                                                                                                                                                                                                                                                                                        |
|------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `keys[]`   | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) A list of specific keys. Entries in `keys` should have exactly as many elements as there are columns in the primary or index key with which this `KeySet` is used. Individual key values are encoded as described [`here`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) . |
| `ranges[]` | [`KeyRange`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeyRange) A list of key ranges. See [`KeyRange`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeyRange) for more information about key range specifications.                                                                                                 |
| `all`      | `bool` For convenience `all` can be set to `true` to indicate that this `KeySet` matches all keys in the table or index. Note that any keys specified in `keys` or `ranges` are only yielded once.                                                                                                                                                                                                                     |

## ListSessionsRequest

The request for [`ListSessions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ListSessions) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>database</code></td>
<td><p><code>string</code></p>
<p>Required. The database in which to list sessions.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>page_size</code></td>
<td><p><code>int32</code></p>
<p>Number of sessions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="odd">
<td><code>page_token</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>page_token</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsResponse.FIELDS.string.google.spanner.v1.ListSessionsResponse.next_page_token"><code>next_page_token</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ListSessionsResponse"><code>ListSessionsResponse</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:</p>
<ul>
<li><code>labels.key</code> where key is the name of a label</li>
</ul>
<p>Some examples of using filters are:</p>
<ul>
<li><code>labels.env:*</code> --&gt; The session has the label "env".</li>
<li><code>labels.env:dev</code> --&gt; The session has the label "env" and the value of the label contains the string "dev".</li>
</ul></td>
</tr>
</tbody>
</table>

## ListSessionsResponse

The response for [`ListSessions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ListSessions) .

| Fields            |                                                                                                                                                                                                                                         |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sessions[]`      | [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Session) The list of requested sessions.                                                                                       |
| `next_page_token` | `string` `next_page_token` can be sent in a subsequent [`ListSessions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ListSessions) call to fetch more of the matching sessions. |

## MultiplexedSessionPrecommitToken

When a read-write transaction is executed on a multiplexed session, this precommit token is sent back to the client as a part of the [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) message in the [`BeginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BeginTransactionRequest) response and also as a part of the [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) and [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet) responses.

| Fields            |                                                                                                                                                                                                               |
|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `precommit_token` | `bytes` Opaque precommit token.                                                                                                                                                                               |
| `seq_num`         | `int32` An incrementing seq number is generated on every precommit token that is returned. Clients should remember the precommit token with the highest sequence number from the current transaction attempt. |

## Mutation

A modification to one or more Cloud Spanner rows. Mutations can be applied to a Cloud Spanner database by sending them in a [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) call.

| Fields                                                                                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `operation` . Required. The operation to perform. `operation` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `insert`                                                                                                    | [`Write`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write) Insert new rows in a table. If any of the rows already exist, the write or transaction fails with error `ALREADY_EXISTS` .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `update`                                                                                                    | [`Write`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write) Update existing rows in a table. If any of the rows does not already exist, the transaction fails with error `NOT_FOUND` .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `insert_or_update`                                                                                          | [`Write`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write) Like [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert) , except that if the row already exists, then its column values are overwritten with the ones provided. Any column values not explicitly written are preserved. When using [`insert_or_update`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert_or_update) , just as when using [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert) , all `NOT NULL` columns in the table must be given a value. This holds true even when the row already exists and will therefore actually be updated. |
| `replace`                                                                                                   | [`Write`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write) Like [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert) , except that if the row already exists, it is deleted, and the column values provided are inserted instead. Unlike [`insert_or_update`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert_or_update) , this means any values not explicitly written become `NULL` . In an interleaved table, if you create the child table with the `ON DELETE CASCADE` annotation, then replacing a parent row also deletes the child rows. Otherwise, you must delete the child rows before you replace the parent row.                                                                                                                          |
| `delete`                                                                                                    | [`Delete`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Delete) Delete rows from a table. Succeeds whether or not the named rows were present.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

## Delete

Arguments to [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Delete.google.spanner.v1.Mutation.delete) operations.

| Fields    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`   | `string` Required. The table whose rows will be deleted.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `key_set` | [`KeySet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeySet) Required. The primary keys of the rows within [`table`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Delete.FIELDS.string.google.spanner.v1.Mutation.Delete.table) to delete. The primary keys must be specified in the order in which they appear in the `PRIMARY KEY()` clause of the table's equivalent DDL statement (the DDL statement used to create the table). Delete is idempotent. The transaction will succeed even if some or all rows do not exist. |

## Write

Arguments to [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert) , [`update`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.update) , [`insert_or_update`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.insert_or_update) , and [`replace`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.FIELDS.google.spanner.v1.Mutation.Write.google.spanner.v1.Mutation.replace) operations.

| Fields      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`     | `string` Required. The table whose rows will be written.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `columns[]` | `string` The names of the columns in [`table`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write.FIELDS.string.google.spanner.v1.Mutation.Write.table) to be written. The list of columns must contain enough columns to allow Cloud Spanner to derive values for all primary key columns in the row(s) to be modified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `values[]`  | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) The values to be written. `values` can contain more than one list of values. If it does, then multiple rows are written, one for each entry in `values` . Each list in `values` must have exactly as many entries as there are entries in [`columns`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write.FIELDS.repeated.string.google.spanner.v1.Mutation.Write.columns) above. Sending multiple lists is equivalent to sending multiple `Mutation` s, each containing one `values` entry and repeating [`table`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write.FIELDS.string.google.spanner.v1.Mutation.Write.table) and [`columns`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Mutation.Write.FIELDS.repeated.string.google.spanner.v1.Mutation.Write.columns) . Individual values in each list are encoded as described [`here`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) . |

## PartialResultSet

Partial results from a streaming read or SQL query. Streaming reads and SQL queries better tolerate large result sets, large rows, and large values, but are a little trickier to consume.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>metadata</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata"><code>ResultSetMetadata</code></a></p>
<p>Metadata about the result set, such as row type information. Only present in the first response.</p></td>
</tr>
<tr class="even">
<td><code>values[]</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#value"><code>Value</code></a></p>
<p>A streamed result set consists of a stream of values, which might be split into many <code>PartialResultSet</code> messages to accommodate large rows and/or large values. Every N complete values defines a row, where N is equal to the number of entries in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType.FIELDS.repeated.google.spanner.v1.StructType.Field.google.spanner.v1.StructType.fields"><code>metadata.row_type.fields</code></a> .</p>
<p>Most values are encoded based on type as described <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode"><code>here</code></a> .</p>
<p>It's possible that the last value in values is "chunked", meaning that the rest of the value is sent in subsequent <code>PartialResultSet</code> (s). This is denoted by the <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet.FIELDS.bool.google.spanner.v1.PartialResultSet.chunked_value"><code>chunked_value</code></a> field. Two or more chunked values can be merged to form a complete value as follows:</p>
<ul>
<li><code>bool/number/null</code> : can't be chunked</li>
<li><code>string</code> : concatenate the strings</li>
<li><code>list</code> : concatenate the lists. If the last element in a list is a <code>string</code> , <code>list</code> , or <code>object</code> , merge it with the first element in the next list by applying these rules recursively.</li>
<li><code>object</code> : concatenate the (field name, field value) pairs. If a field name is duplicated, then apply these rules recursively to merge the field values.</li>
</ul>
<p>Some examples of merging:</p>
<pre data-fenced=""><code>Strings are concatenated.
&quot;foo&quot;, &quot;bar&quot; =&gt; &quot;foobar&quot;

Lists of non-strings are concatenated.
[2, 3], [4] =&gt; [2, 3, 4]

Lists are concatenated, but the last and first elements are merged
because they are strings.
[&quot;a&quot;, &quot;b&quot;], [&quot;c&quot;, &quot;d&quot;] =&gt; [&quot;a&quot;, &quot;bc&quot;, &quot;d&quot;]

Lists are concatenated, but the last and first elements are merged
because they are lists. Recursively, the last and first elements
of the inner lists are merged because they are strings.
[&quot;a&quot;, [&quot;b&quot;, &quot;c&quot;]], [[&quot;d&quot;], &quot;e&quot;] =&gt; [&quot;a&quot;, [&quot;b&quot;, &quot;cd&quot;], &quot;e&quot;]

Non-overlapping object fields are combined.
{&quot;a&quot;: &quot;1&quot;}, {&quot;b&quot;: &quot;2&quot;} =&gt; {&quot;a&quot;: &quot;1&quot;, &quot;b&quot;: 2&quot;}

Overlapping object fields are merged.
{&quot;a&quot;: &quot;1&quot;}, {&quot;a&quot;: &quot;2&quot;} =&gt; {&quot;a&quot;: &quot;12&quot;}

Examples of merging objects containing lists of strings.
{&quot;a&quot;: [&quot;1&quot;]}, {&quot;a&quot;: [&quot;2&quot;]} =&gt; {&quot;a&quot;: [&quot;12&quot;]}</code></pre>
<p>For a more complete example, suppose a streaming SQL query is yielding a result set whose rows contain a single string field. The following <code>PartialResultSet</code> s might be yielded:</p>
<pre data-fenced=""><code>{
  &quot;metadata&quot;: { ... }
  &quot;values&quot;: [&quot;Hello&quot;, &quot;W&quot;]
  &quot;chunked_value&quot;: true
  &quot;resume_token&quot;: &quot;Af65...&quot;
}
{
  &quot;values&quot;: [&quot;orl&quot;]
  &quot;chunked_value&quot;: true
}
{
  &quot;values&quot;: [&quot;d&quot;]
  &quot;resume_token&quot;: &quot;Zx1B...&quot;
}</code></pre>
<p>This sequence of <code>PartialResultSet</code> s encodes two rows, one containing the field value <code>"Hello"</code> , and a second containing the field value <code>"World" = "W" + "orl" + "d"</code> .</p>
<p>Not all <code>PartialResultSet</code> s contain a <code>resume_token</code> . Execution can only be resumed from a previously yielded <code>resume_token</code> . For the above sequence of <code>PartialResultSet</code> s, resuming the query with <code>"resume_token": "Af65..."</code> yields results from the <code>PartialResultSet</code> with value "orl".</p></td>
</tr>
<tr class="odd">
<td><code>chunked_value</code></td>
<td><p><code>bool</code></p>
<p>If true, then the final value in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet.FIELDS.repeated.google.protobuf.Value.google.spanner.v1.PartialResultSet.values"><code>values</code></a> is chunked, and must be combined with more values from subsequent <code>PartialResultSet</code> s to obtain a complete field value.</p></td>
</tr>
<tr class="even">
<td><code>resume_token</code></td>
<td><p><code>bytes</code></p>
<p>Streaming calls might be interrupted for a variety of reasons, such as TCP connection loss. If this occurs, the stream of results can be resumed by re-sending the original request and including <code>resume_token</code> . Note that executing any other transaction in the same session invalidates the token.</p></td>
</tr>
<tr class="odd">
<td><code>stats</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetStats"><code>ResultSetStats</code></a></p>
<p>Query plan and execution statistics for the statement that produced this streaming result set. These can be requested by setting <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.spanner.v1.ExecuteSqlRequest.QueryMode.google.spanner.v1.ExecuteSqlRequest.query_mode"><code>ExecuteSqlRequest.query_mode</code></a> and are sent only once with the last response in the stream. This field is also present in the last response for DML statements.</p></td>
</tr>
<tr class="even">
<td><code>precommit_token</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken"><code>MultiplexedSessionPrecommitToken</code></a></p>
<p>Optional. A precommit token is included if the read-write transaction has multiplexed sessions enabled. Pass the precommit token with the highest sequence number from this transaction attempt to the <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit"><code>Commit</code></a> request for this transaction.</p></td>
</tr>
<tr class="odd">
<td><code>last</code></td>
<td><p><code>bool</code></p>
<p>Optional. Indicates whether this is the last <code>PartialResultSet</code> in the stream. The server might optionally set this field. Clients shouldn't rely on this field being set in all cases.</p></td>
</tr>
</tbody>
</table>

## Partition

Information returned for each partition returned in a PartitionResponse.

| Fields            |                                                                                                                                                                                      |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partition_token` | `bytes` This token can be passed to `Read` , `StreamingRead` , `ExecuteSql` , or `ExecuteStreamingSql` requests to restrict the results to those identified by this partition token. |

## PartitionOptions

Options for a `PartitionQueryRequest` and `PartitionReadRequest` .

| Fields                 |                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partition_size_bytes` | `int64` **Note:** This hint is currently ignored by `PartitionQuery` and `PartitionRead` requests. The desired data size for each partition generated. The default for this option is currently 1 GiB. This is only a hint. The actual size of each partition can be smaller or larger than this size request.                                                                                                                             |
| `max_partitions`       | `int64` **Note:** This hint is currently ignored by `PartitionQuery` and `PartitionRead` requests. The desired maximum number of partitions to return. For example, this might be set to the number of workers available. The default for this option is currently 10,000. The maximum value is currently 200,000. This is only a hint. The actual number of partitions returned can be smaller or larger than this maximum count request. |

## PartitionQueryRequest

The request for [`PartitionQuery`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.PartitionQuery)

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session used to create the partitions.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.partitionQuery</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector"><code>TransactionSelector</code></a></p>
<p>Read-only snapshot transactions are supported, read and write and single-use transactions are not.</p></td>
</tr>
<tr class="odd">
<td><code>sql</code></td>
<td><p><code>string</code></p>
<p>Required. The query request to generate partitions for. The request fails if the query isn't root partitionable. For a query to be root partitionable, it needs to satisfy a few conditions. For example, if the query execution plan contains a distributed union operator, then it must be the first operator in the plan. For more information about other conditions, see <a href="https://cloud.google.com/spanner/docs/reads#read_data_in_parallel">Read data in parallel</a> .</p>
<p>The query request must not contain DML commands, such as <code>INSERT</code> , <code>UPDATE</code> , or <code>DELETE</code> . Use <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql"><code>ExecuteStreamingSql</code></a> with a <code>PartitionedDml</code> transaction for large, partition-friendly DML operations.</p></td>
</tr>
<tr class="even">
<td><code>params</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a></p>
<p>Optional. Parameter names and values that bind to placeholders in the SQL string.</p>
<p>A parameter placeholder consists of the <code>@</code> character followed by the parameter name (for example, <code>@firstName</code> ). Parameter names can contain letters, numbers, and underscores.</p>
<p>Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:</p>
<p><code>"WHERE id &gt; @msg_id AND id &lt; @msg_id + 100"</code></p>
<p>It's an error to execute a SQL statement with unbound parameters.</p></td>
</tr>
<tr class="odd">
<td><code>param_types</code></td>
<td><p><code>map&lt;string, </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type"><code>Type</code></a><code> &gt;</code></p>
<p>Optional. It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type <code>BYTES</code> and values of type <code>STRING</code> both appear in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionQueryRequest.FIELDS.google.protobuf.Struct.google.spanner.v1.PartitionQueryRequest.params"><code>params</code></a> as JSON strings.</p>
<p>In these cases, <code>param_types</code> can be used to specify the exact SQL type for some or all of the SQL query parameters. See the definition of <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type"><code>Type</code></a> for more information about SQL types.</p></td>
</tr>
<tr class="even">
<td><code>partition_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionOptions"><code>PartitionOptions</code></a></p>
<p>Additional options that affect how many partitions are created.</p></td>
</tr>
</tbody>
</table>

## PartitionReadRequest

The request for [`PartitionRead`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.PartitionRead)

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session used to create the partitions.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.partitionRead</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector"><code>TransactionSelector</code></a></p>
<p>Read only snapshot transactions are supported, read/write and single use transactions are not.</p></td>
</tr>
<tr class="odd">
<td><code>table</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the table in the database to be read.</p></td>
</tr>
<tr class="even">
<td><code>index</code></td>
<td><p><code>string</code></p>
<p>If non-empty, the name of an index on <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.table"><code>table</code></a> . This index is used instead of the table primary key when interpreting <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.PartitionReadRequest.key_set"><code>key_set</code></a> and sorting result rows. See <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.PartitionReadRequest.key_set"><code>key_set</code></a> for further information.</p></td>
</tr>
<tr class="odd">
<td><code>columns[]</code></td>
<td><p><code>string</code></p>
<p>The columns of <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.table"><code>table</code></a> to be returned for each row matching this request.</p></td>
</tr>
<tr class="even">
<td><code>key_set</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeySet"><code>KeySet</code></a></p>
<p>Required. <code>key_set</code> identifies the rows to be yielded. <code>key_set</code> names the primary keys of the rows in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.table"><code>table</code></a> to be yielded, unless <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.index"><code>index</code></a> is present. If <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.index"><code>index</code></a> is present, then <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.PartitionReadRequest.key_set"><code>key_set</code></a> instead names index keys in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionReadRequest.FIELDS.string.google.spanner.v1.PartitionReadRequest.index"><code>index</code></a> .</p>
<p>It isn't an error for the <code>key_set</code> to name rows that don't exist in the database. Read yields nothing for nonexistent rows.</p></td>
</tr>
<tr class="odd">
<td><code>partition_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartitionOptions"><code>PartitionOptions</code></a></p>
<p>Additional options that affect how many partitions are created.</p></td>
</tr>
</tbody>
</table>

## PartitionResponse

The response for [`PartitionQuery`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.PartitionQuery) or [`PartitionRead`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.PartitionRead)

| Fields         |                                                                                                                                                                |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partitions[]` | [`Partition`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Partition) Partitions created by this request.      |
| `transaction`  | [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) Transaction created by this request. |

## PlanNode

Node information for nodes appearing in a [`QueryPlan.plan_nodes`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryPlan.FIELDS.repeated.google.spanner.v1.PlanNode.google.spanner.v1.QueryPlan.plan_nodes) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>index</code></td>
<td><p><code>int32</code></p>
<p>The <code>PlanNode</code> 's index in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryPlan.FIELDS.repeated.google.spanner.v1.PlanNode.google.spanner.v1.QueryPlan.plan_nodes"><code>node list</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>kind</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.Kind"><code>Kind</code></a></p>
<p>Used to determine the type of node. May be needed for visualizing different kinds of nodes differently. For example, If the node is a <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.Kind.ENUM_VALUES.google.spanner.v1.PlanNode.Kind.SCALAR"><code>SCALAR</code></a> node, it will have a condensed representation which can be used to directly embed a description of the node in its parent.</p></td>
</tr>
<tr class="odd">
<td><code>display_name</code></td>
<td><p><code>string</code></p>
<p>The display name for the node.</p></td>
</tr>
<tr class="even">
<td><code>child_links[]</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.ChildLink"><code>ChildLink</code></a></p>
<p>List of child node <code>index</code> es and their relationship to this parent.</p></td>
</tr>
<tr class="odd">
<td><code>short_representation</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.ShortRepresentation"><code>ShortRepresentation</code></a></p>
<p>Condensed representation for <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.Kind.ENUM_VALUES.google.spanner.v1.PlanNode.Kind.SCALAR"><code>SCALAR</code></a> nodes.</p></td>
</tr>
<tr class="even">
<td><code>metadata</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a></p>
<p>Attributes relevant to the node contained in a group of key-value pairs. For example, a Parameter Reference node could have the following information in its metadata:</p>
<pre data-fenced=""><code>{
  &quot;parameter_reference&quot;: &quot;param1&quot;,
  &quot;parameter_type&quot;: &quot;array&quot;
}</code></pre></td>
</tr>
<tr class="odd">
<td><code>execution_stats</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a></p>
<p>The execution statistics associated with the node, contained in a group of key-value pairs. Only present if the plan was returned as a result of a profile query. For example, number of executions, number of rows/time per execution etc.</p></td>
</tr>
</tbody>
</table>

## ChildLink

Metadata associated with a parent-child relationship appearing in a [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) .

| Fields        |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|---------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `child_index` | `int32` The node to which the link points.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `type`        | `string` The type of the link. For example, in Hash Joins this could be used to distinguish between the build child and the probe child, or in the case of the child being an output variable, to represent the tag associated with the output variable.                                                                                                                                                                                                                                                                                                                                                                              |
| `variable`    | `string` Only present if the child node is [`SCALAR`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode.Kind.ENUM_VALUES.google.spanner.v1.PlanNode.Kind.SCALAR) and corresponds to an output variable of the parent node. The field carries the name of the output variable. For example, a `TableScan` operator that reads rows from a table will have child links to the `SCALAR` nodes representing the output variables created for each column that is read by the operator. The corresponding `variable` fields will be set to the variable names assigned to the columns. |

## Kind

The kind of [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) . Distinguishes between the two different kinds of nodes that can appear in a query plan.

| Enums              |                                                                                                                                                                                                                                    |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `KIND_UNSPECIFIED` | Not specified.                                                                                                                                                                                                                     |
| `RELATIONAL`       | Denotes a Relational operator node in the expression tree. Relational operators represent iterative processing of rows during query execution. For example, a `TableScan` operation that reads rows from a table.                  |
| `SCALAR`           | Denotes a Scalar node in the expression tree. Scalar nodes represent non-iterable entities in the query plan. For example, constants or arithmetic operators appearing inside predicate expressions or references to column names. |

## ShortRepresentation

Condensed representation of a node and its subtree. Only present for `SCALAR` [`PlanNode(s)`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) .

| Fields        |                                                                                                                                                                                                                                                                                                                      |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `description` | `string` A string representation of the expression subtree rooted at this node.                                                                                                                                                                                                                                      |
| `subqueries`  | `map<string, int32>` A mapping of (subquery variable name) -\> (subquery node id) for cases where the `description` string of this node references a `SCALAR` subquery contained in the expression subtree rooted at this node. The referenced `SCALAR` subquery may not necessarily be a direct child of this node. |

## QueryAdvisorResult

Output of query advisor analysis.

| Fields           |                                                                                                                                                                                                                                                                                                                                                   |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `index_advice[]` | [`IndexAdvice`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryAdvisorResult.IndexAdvice) Optional. Index Recommendation for a query. This is an optional field and the recommendation will only be available when the recommendation guarantees significant improvement in query performance. |

## IndexAdvice

Recommendation to add new indexes to run queries more efficiently.

| Fields               |                                                                                                                                                                                            |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ddl[]`              | `string` Optional. DDL statements to add new indexes that will improve the query.                                                                                                          |
| `improvement_factor` | `double` Optional. Estimated latency improvement factor. For example if the query currently takes 500 ms to run and the estimated latency with new indexes is 100 ms this field will be 5. |

## QueryPlan

Contains an ordered list of nodes appearing in the query plan.

| Fields         |                                                                                                                                                                                                                                                                                                                                                                                                |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `plan_nodes[]` | [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) The nodes in the query plan. Plan nodes are returned in pre-order starting with the plan root. Each [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PlanNode) 's `id` corresponds to its index in `plan_nodes` . |
| `query_advice` | [`QueryAdvisorResult`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryAdvisorResult) Optional. The advise/recommendations for a query. Currently this field will be serving index recommendations for a query.                                                                                                                              |

## ReadRequest

The request for [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) and [`StreamingRead`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.StreamingRead) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the read should be performed.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.read</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionSelector"><code>TransactionSelector</code></a></p>
<p>The transaction to use. If none is provided, the default is a temporary read-only transaction with strong concurrency.</p></td>
</tr>
<tr class="odd">
<td><code>table</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the table in the database to be read.</p></td>
</tr>
<tr class="even">
<td><code>index</code></td>
<td><p><code>string</code></p>
<p>If non-empty, the name of an index on <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.table"><code>table</code></a> . This index is used instead of the table primary key when interpreting <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.ReadRequest.key_set"><code>key_set</code></a> and sorting result rows. See <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.ReadRequest.key_set"><code>key_set</code></a> for further information.</p></td>
</tr>
<tr class="odd">
<td><code>columns[]</code></td>
<td><p><code>string</code></p>
<p>Required. The columns of <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.table"><code>table</code></a> to be returned for each row matching this request.</p></td>
</tr>
<tr class="even">
<td><code>key_set</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.KeySet"><code>KeySet</code></a></p>
<p>Required. <code>key_set</code> identifies the rows to be yielded. <code>key_set</code> names the primary keys of the rows in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.table"><code>table</code></a> to be yielded, unless <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.index"><code>index</code></a> is present. If <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.index"><code>index</code></a> is present, then <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.google.spanner.v1.KeySet.google.spanner.v1.ReadRequest.key_set"><code>key_set</code></a> instead names index keys in <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.index"><code>index</code></a> .</p>
<p>If the <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.bytes.google.spanner.v1.ReadRequest.partition_token"><code>partition_token</code></a> field is empty, rows are yielded in table primary key order (if <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.index"><code>index</code></a> is empty) or index key order (if <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.string.google.spanner.v1.ReadRequest.index"><code>index</code></a> is non-empty). If the <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ReadRequest.FIELDS.bytes.google.spanner.v1.ReadRequest.partition_token"><code>partition_token</code></a> field isn't empty, rows are yielded in an unspecified order.</p>
<p>It isn't an error for the <code>key_set</code> to name rows that don't exist in the database. Read yields nothing for nonexistent rows.</p></td>
</tr>
<tr class="odd">
<td><code>limit</code></td>
<td><p><code>int64</code></p>
<p>If greater than zero, only the first <code>limit</code> rows are yielded. If <code>limit</code> is zero, the default is no limit. A limit can't be specified if <code>partition_token</code> is set.</p></td>
</tr>
<tr class="even">
<td><code>resume_token</code></td>
<td><p><code>bytes</code></p>
<p>If this request is resuming a previously interrupted read, <code>resume_token</code> should be copied from the last <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet"><code>PartialResultSet</code></a> yielded before the interruption. Doing this enables the new read to resume where the last read left off. The rest of the request parameters must exactly match the request that yielded this token.</p></td>
</tr>
<tr class="odd">
<td><code>partition_token</code></td>
<td><p><code>bytes</code></p>
<p>If present, results are restricted to the specified partition previously created using <code>PartitionRead</code> . There must be an exact match for the values of fields common to this message and the PartitionReadRequest message used to create this partition_token.</p></td>
</tr>
<tr class="even">
<td><code>request_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions"><code>RequestOptions</code></a></p>
<p>Common options for this request.</p></td>
</tr>
<tr class="odd">
<td><code>directed_read_options</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.DirectedReadOptions"><code>DirectedReadOptions</code></a></p>
<p>Directed read options for this request.</p></td>
</tr>
<tr class="even">
<td><code>data_boost_enabled</code></td>
<td><p><code>bool</code></p>
<p>If this is for a partitioned read and this field is set to <code>true</code> , the request is executed with Spanner Data Boost independent compute resources.</p>
<p>If the field is set to <code>true</code> but the request doesn't set <code>partition_token</code> , the API returns an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
</tbody>
</table>

## RequestOptions

Common request options for various APIs.

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `priority`        | [`Priority`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions.Priority) Priority for the request.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `request_tag`     | `string` A per-request tag which can be applied to queries or reads, used for statistics collection. Both `request_tag` and `transaction_tag` can be specified for a read or query that belongs to a transaction. This field is ignored for requests where it's not applicable (for example, `CommitRequest` ). Legal characters for `request_tag` values are all printable characters (ASCII 32 - 126) and the length of a request_tag is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string.                                                                                                                                                                                                                                                               |
| `transaction_tag` | `string` A tag used for statistics collection about this transaction. Both `request_tag` and `transaction_tag` can be specified for a read or query that belongs to a transaction. To enable tagging on a transaction, `transaction_tag` must be set to the same value for all requests belonging to the same transaction, including [`BeginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BeginTransaction) . If this request doesn't belong to any transaction, `transaction_tag` is ignored. Legal characters for `transaction_tag` values are all printable characters (ASCII 32 - 126) and the length of a `transaction_tag` is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string. |

## Priority

The relative priority for requests. Note that priority isn't applicable for [`BeginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BeginTransaction) .

The priority acts as a hint to the Cloud Spanner scheduler and doesn't guarantee priority or order of execution. For example:

- Some parts of a write operation always execute at `PRIORITY_HIGH` , regardless of the specified priority. This can cause you to see an increase in high priority workload even when executing a low priority request. This can also potentially cause a priority inversion where a lower priority request is fulfilled ahead of a higher priority request.
- If a transaction contains multiple operations with different priorities, Cloud Spanner doesn't guarantee to process the higher priority operations first. There might be other constraints to satisfy, such as the order of operations.

| Enums                  |                                                           |
|------------------------|-----------------------------------------------------------|
| `PRIORITY_UNSPECIFIED` | `PRIORITY_UNSPECIFIED` is equivalent to `PRIORITY_HIGH` . |
| `PRIORITY_LOW`         | This specifies that the request is low priority.          |
| `PRIORITY_MEDIUM`      | This specifies that the request is medium priority.       |
| `PRIORITY_HIGH`        | This specifies that the request is high priority.         |

## ResultSet

Results from [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) or [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) .

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `metadata`        | [`ResultSetMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata) Metadata about the result set, such as row type information.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `rows[]`          | [`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value) Each element in `rows` is a row whose format is defined by [`metadata.row_type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata.FIELDS.google.spanner.v1.StructType.google.spanner.v1.ResultSetMetadata.row_type) . The ith element in each row matches the ith field in [`metadata.row_type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata.FIELDS.google.spanner.v1.StructType.google.spanner.v1.ResultSetMetadata.row_type) . Elements are encoded based on type as described [`here`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `stats`           | [`ResultSetStats`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetStats) Query plan and execution statistics for the SQL statement that produced this result set. These can be requested by setting [`ExecuteSqlRequest.query_mode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.spanner.v1.ExecuteSqlRequest.QueryMode.google.spanner.v1.ExecuteSqlRequest.query_mode) . DML statements always produce stats containing the number of rows modified, unless executed using the [`ExecuteSqlRequest.QueryMode.PLAN`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.QueryMode.ENUM_VALUES.google.spanner.v1.ExecuteSqlRequest.QueryMode.PLAN) [`ExecuteSqlRequest.query_mode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.spanner.v1.ExecuteSqlRequest.QueryMode.google.spanner.v1.ExecuteSqlRequest.query_mode) . Other fields might or might not be populated, based on the [`ExecuteSqlRequest.query_mode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ExecuteSqlRequest.FIELDS.google.spanner.v1.ExecuteSqlRequest.QueryMode.google.spanner.v1.ExecuteSqlRequest.query_mode) . |
| `precommit_token` | [`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken) Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt to the [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) request for this transaction.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

## ResultSetMetadata

Metadata about a [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) or [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>row_type</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType"><code>StructType</code></a></p>
<p>Indicates the field names and types for the rows in the result set. For example, a SQL query like <code>"SELECT UserId, UserName FROM Users"</code> could return a <code>row_type</code> value like:</p>
<pre data-fenced=""><code>&quot;fields&quot;: [
  { &quot;name&quot;: &quot;UserId&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;INT64&quot; } },
  { &quot;name&quot;: &quot;UserName&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;STRING&quot; } },
]</code></pre></td>
</tr>
<tr class="even">
<td><code>transaction</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction"><code>Transaction</code></a></p>
<p>If the read or SQL query began a transaction as a side-effect, the information about the new transaction is yielded here.</p></td>
</tr>
<tr class="odd">
<td><code>undeclared_parameters</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType"><code>StructType</code></a></p>
<p>A SQL query can be parameterized. In PLAN mode, these parameters can be undeclared. This indicates the field names and types for those undeclared parameters in the SQL query. For example, a SQL query like <code>"SELECT * FROM Users where UserId = @userId and UserName = @userName "</code> could return a <code>undeclared_parameters</code> value like:</p>
<pre data-fenced=""><code>&quot;fields&quot;: [
  { &quot;name&quot;: &quot;UserId&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;INT64&quot; } },
  { &quot;name&quot;: &quot;UserName&quot;, &quot;type&quot;: { &quot;code&quot;: &quot;STRING&quot; } },
]</code></pre></td>
</tr>
</tbody>
</table>

## ResultSetStats

Additional statistics about a [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSet) or [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.PartialResultSet) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>query_plan</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryPlan"><code>QueryPlan</code></a></p>
<p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.QueryPlan"><code>QueryPlan</code></a> for the query associated with this result.</p></td>
</tr>
<tr class="even">
<td><code>query_stats</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a></p>
<p>Aggregated statistics from the execution of the query. Only present when the query is profiled. For example, a query could return the statistics as follows:</p>
<pre data-fenced=""><code>{
  &quot;rows_returned&quot;: &quot;3&quot;,
  &quot;elapsed_time&quot;: &quot;1.22 secs&quot;,
  &quot;cpu_time&quot;: &quot;1.19 secs&quot;
}</code></pre></td>
</tr>
<tr class="odd">
<td>Union field <code>row_count</code> . The number of rows modified by the DML statement. <code>row_count</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>row_count_exact</code></td>
<td><p><code>int64</code></p>
<p>Standard DML returns an exact count of rows that were modified.</p></td>
</tr>
<tr class="odd">
<td><code>row_count_lower_bound</code></td>
<td><p><code>int64</code></p>
<p>Partitioned DML doesn't offer exactly-once semantics, so it returns a lower bound of the rows modified.</p></td>
</tr>
</tbody>
</table>

## RollbackRequest

The request for [`Rollback`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Rollback) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>session</code></td>
<td><p><code>string</code></p>
<p>Required. The session in which the transaction to roll back is running.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>session</code> :</p>
<ul>
<li><code>spanner.databases.beginOrRollbackReadWriteTransaction</code></li>
</ul></td>
</tr>
<tr class="even">
<td><code>transaction_id</code></td>
<td><p><code>bytes</code></p>
<p>Required. The transaction to roll back.</p></td>
</tr>
</tbody>
</table>

## Session

A session in the Cloud Spanner API.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Output only. The name of the session. This is always system-assigned.</p></td>
</tr>
<tr class="even">
<td><code>labels</code></td>
<td><p><code>map&lt;string, string&gt;</code></p>
<p>The labels for the session.</p>
<ul>
<li>Label keys must be between 1 and 63 characters long and must conform to the following regular expression: <code>[a-z]([-a-z0-9]*[a-z0-9])?</code> .</li>
<li>Label values must be between 0 and 63 characters long and must conform to the regular expression <code>([a-z]([-a-z0-9]*[a-z0-9])?)?</code> .</li>
<li>No more than 64 labels can be associated with a given session.</li>
</ul>
<p>See <a href="https://goo.gl/xmQnxf">https://goo.gl/xmQnxf</a> for more information on and examples of labels.</p></td>
</tr>
<tr class="odd">
<td><code>create_time</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a></p>
<p>Output only. The timestamp when the session is created.</p></td>
</tr>
<tr class="even">
<td><code>approximate_last_use_time</code></td>
<td><p><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a></p>
<p>Output only. The approximate timestamp when the session is last used. It's typically earlier than the actual last use time.</p></td>
</tr>
<tr class="odd">
<td><code>creator_role</code></td>
<td><p><code>string</code></p>
<p>The database role which created this session.</p></td>
</tr>
<tr class="even">
<td><code>multiplexed</code></td>
<td><p><code>bool</code></p>
<p>Optional. If <code>true</code> , specifies a multiplexed session. Use a multiplexed session for multiple, concurrent operations including any combination of read-only and read-write transactions. Use <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.CreateSession"><code>sessions.create</code></a> to create multiplexed sessions. Don't use <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.BatchCreateSessions"><code>BatchCreateSessions</code></a> to create a multiplexed session. You can't delete or list multiplexed sessions.</p></td>
</tr>
</tbody>
</table>

## StructType

`StructType` defines the fields of a [`STRUCT`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.STRUCT) type.

| Fields     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `fields[]` | [`Field`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType.Field) The list of fields that make up this struct. Order is significant, because values of this struct type are represented as lists, where the order of field values matches the order of fields in the [`StructType`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType) . In turn, the order of fields matches the order of columns in a read request, or the order of fields in the `SELECT` clause of a query. |

## Field

Message representing a single field of a struct.

| Fields |                                                                                                                                                                                                                                                                                                                                                                                                               |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name` | `string` The name of the field. For reads, this is the column name. For SQL queries, it is the column alias (e.g., `"Word"` in the query `"SELECT 'hello' AS Word"` ), or the column name (e.g., `"ColName"` in the query `"SELECT ColName FROM Table"` ). Some columns might have an empty name (e.g., `"SELECT UPPER(ColName)"` ). Note that a query result can contain multiple fields with the same name. |
| `type` | [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) The type of the field.                                                                                                                                                                                                                                                                            |

## Transaction

A transaction.

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `id`              | `bytes` `id` may be used to identify the transaction in subsequent [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) , [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) , [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) , or [`Rollback`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Rollback) calls. Single-use read-only transactions do not have IDs, because single-use transactions do not support multiple requests.                                                 |
| `read_timestamp`  | [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) For snapshot read-only transactions, the read timestamp chosen for the transaction. Not returned by default: see [`TransactionOptions.ReadOnly.return_read_timestamp`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadOnly.FIELDS.bool.google.spanner.v1.TransactionOptions.ReadOnly.return_read_timestamp) . A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `"2014-10-02T15:01:23.045123456Z"` .                                                                                                                                                                           |
| `precommit_token` | [`MultiplexedSessionPrecommitToken`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.MultiplexedSessionPrecommitToken) A precommit token is included in the response of a BeginTransaction request if the read-write transaction is on a multiplexed session and a mutation_key was specified in the [`BeginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.BeginTransactionRequest) . The precommit token with the highest sequence number from this transaction attempt should be passed to the [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Commit) request for this transaction. |

## TransactionOptions

Options to use for transactions.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>exclude_txn_from_change_streams</code></td>
<td><p><code>bool</code></p>
<p>When <code>exclude_txn_from_change_streams</code> is set to <code>true</code> , it prevents read or write transactions from being tracked in change streams.</p>
<ul>
<li><p>If the DDL option <code>allow_txn_exclusion</code> is set to <code>true</code> , then the updates made within this transaction aren't recorded in the change stream.</p></li>
<li><p>If you don't set the DDL option <code>allow_txn_exclusion</code> or if it's set to <code>false</code> , then the updates made within this transaction are recorded in the change stream.</p></li>
</ul>
<p>When <code>exclude_txn_from_change_streams</code> is set to <code>false</code> or not set, modifications from this transaction are recorded in all change streams that are tracking columns modified by these transactions.</p>
<p>The <code>exclude_txn_from_change_streams</code> option can only be specified for read-write or partitioned DML transactions, otherwise the API returns an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
<tr class="even">
<td><code>isolation_level</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel"><code>IsolationLevel</code></a></p>
<p>Isolation level for the transaction.</p></td>
</tr>
<tr class="odd">
<td>Union field <code>mode</code> . Required. The type of transaction. <code>mode</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>read_write</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadWrite"><code>ReadWrite</code></a></p>
<p>Transaction may write.</p>
<p>Authorization to begin a read-write transaction requires <code>spanner.databases.beginOrRollbackReadWriteTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
<tr class="odd">
<td><code>partitioned_dml</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.PartitionedDml"><code>PartitionedDml</code></a></p>
<p>Partitioned DML transaction.</p>
<p>Authorization to begin a Partitioned DML transaction requires <code>spanner.databases.beginPartitionedDmlTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
<tr class="even">
<td><code>read_only</code></td>
<td><p><a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadOnly"><code>ReadOnly</code></a></p>
<p>Transaction does not write.</p>
<p>Authorization to begin a read-only transaction requires <code>spanner.databases.beginReadOnlyTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
</tbody>
</table>

## IsolationLevel

`IsolationLevel` is used when setting the [isolation level](https://cloud.google.com/spanner/docs/isolation-levels) for a transaction.

| Enums                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ISOLATION_LEVEL_UNSPECIFIED` | Default value. If the value is not specified, the `SERIALIZABLE` isolation level is used.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `SERIALIZABLE`                | All transactions appear as if they executed in a serial order, even if some of the reads, writes, and other operations of distinct transactions actually occurred in parallel. Spanner assigns commit timestamps that reflect the order of committed transactions to implement this property. Spanner offers a stronger guarantee than serializability called external consistency. For more information, see [TrueTime and external consistency](https://cloud.google.com/spanner/docs/true-time-external-consistency#serializability) .                                                      |
| `REPEATABLE_READ`             | All reads performed during the transaction observe a consistent snapshot of the database, and the transaction is only successfully committed in the absence of conflicts between its updates and any concurrent updates that have occurred since that snapshot. Consequently, in contrast to `SERIALIZABLE` transactions, only write-write conflicts are detected in snapshot transactions. This isolation level does not support read-only and partitioned DML transactions. When `REPEATABLE_READ` is specified on a read-write transaction, the locking semantics default to `OPTIMISTIC` . |

## PartitionedDml

This type has no fields.

Message type to initiate a Partitioned DML transaction.

## ReadOnly

Message type to initiate a read-only transaction.

| Fields                                                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `return_read_timestamp`                                                                                                                        | `bool` If true, the Cloud Spanner-selected read timestamp is included in the [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) message that describes the transaction.                                                                                                                                                                                                                                                                                                                                                                                               |
| Union field `timestamp_bound` . How to choose the timestamp for the read-only transaction. `timestamp_bound` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `strong`                                                                                                                                       | `bool` Read at a timestamp where all previously committed transactions are visible.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `min_read_timestamp`                                                                                                                           | [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) Executes all reads at a timestamp \>= `min_read_timestamp` . This is useful for requesting fresher data than some previous read, or data that is fresh enough to observe the effects of some previously committed transaction whose timestamp is known. Note that this option can only be used in single-use transactions. A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `"2014-10-02T15:01:23.045123456Z"` .                                                                                                                |
| `max_staleness`                                                                                                                                | [`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration) Read data at a timestamp \>= `NOW - max_staleness` seconds. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps. Useful for reading the freshest data available at a nearby replica, while bounding the possible staleness if the local replica has fallen behind. Note that this option can only be used in single-use transactions. |
| `read_timestamp`                                                                                                                               | [`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp) Executes all reads at the given timestamp. Unlike other modes, reads at a specific timestamp are repeatable; the same read at the same timestamp always returns the same data. If the timestamp is in the future, the read is blocked until the specified timestamp, modulo the read's deadline. Useful for large scale consistent reads such as mapreduces, or for coordinating many reads against a consistent snapshot of the data. A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `"2014-10-02T15:01:23.045123456Z"` .    |
| `exact_staleness`                                                                                                                              | [`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration) Executes all reads at a timestamp that is `exact_staleness` old. The timestamp is chosen soon after the read is started. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps. Useful for reading at nearby replicas without the distributed timestamp negotiation overhead of `max_staleness` .                                       |

## ReadWrite

Message type to initiate a read-write transaction. Currently this transaction type has no options.

| Fields                                        |                                                                                                                                                                                                  |
|-----------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `read_lock_mode`                              | [`ReadLockMode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.ReadWrite.ReadLockMode) The read lock mode for the transaction. |
| `multiplexed_session_previous_transaction_id` | `bytes` Optional. Clients should pass the transaction ID of the previous transaction attempt that was aborted if this transaction is being executed on a multiplexed session.                    |

## ReadLockMode

`ReadLockMode` is used to set the read lock mode for read-write transactions.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Enums</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>READ_LOCK_MODE_UNSPECIFIED</code></td>
<td><p>Default value.</p>
<ul>
<li>If isolation level is <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.SERIALIZABLE"><code>SERIALIZABLE</code></a> , locking semantics default to <code>PESSIMISTIC</code> .</li>
<li>If isolation level is <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> , locking semantics default to <code>OPTIMISTIC</code> .</li>
<li>See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for more details.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>PESSIMISTIC</code></td>
<td><p>Pessimistic lock mode.</p>
<p>Lock acquisition behavior depends on the isolation level in use. In <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.SERIALIZABLE"><code>SERIALIZABLE</code></a> isolation, reads and writes acquire necessary locks during transaction statement execution. In <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> isolation, reads that explicitly request to be locked and writes acquire locks. See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for details on the types of locks acquired at each transaction step.</p></td>
</tr>
<tr class="odd">
<td><code>OPTIMISTIC</code></td>
<td><p>Optimistic lock mode.</p>
<p>Lock acquisition behavior depends on the isolation level in use. In both <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.SERIALIZABLE"><code>SERIALIZABLE</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions.IsolationLevel.ENUM_VALUES.google.spanner.v1.TransactionOptions.IsolationLevel.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> isolation, reads and writes do not acquire locks during transaction statement execution. See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for details on how the guarantees of each isolation level are provided at commit time.</p></td>
</tr>
</tbody>
</table>

## TransactionSelector

This message is used to select the transaction in which a [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) or [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteSql) call runs.

See [`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions) for more information about transactions.

| Fields                                                                                                                                                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `selector` . If no fields are set, the default is a single use transaction with strong concurrency. `selector` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `single_use`                                                                                                                                                 | [`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions) Execute the read or SQL query in a temporary transaction. This is the most efficient way to execute a transaction that consists of a single SQL query.                                                                                                                                                                                                                                                                                                                                                     |
| `id`                                                                                                                                                         | `bytes` Execute the read or SQL query in a previously-started transaction.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `begin`                                                                                                                                                      | [`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TransactionOptions) Begin a new transaction and execute this read or SQL query in it. The transaction ID of the new transaction is returned in [`ResultSetMetadata.transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.ResultSetMetadata.FIELDS.google.spanner.v1.Transaction.google.spanner.v1.ResultSetMetadata.transaction) , which is a [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Transaction) . |

## Type

`Type` indicates the type of a Cloud Spanner value, as might be stored in a table cell or returned from an SQL query.

| Fields               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `code`               | [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) Required. The [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) for this type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `array_element_type` | [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.TypeCode.google.spanner.v1.Type.code) == [`ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.ARRAY) , then `array_element_type` is the type of the array elements.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `struct_type`        | [`StructType`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType) If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.TypeCode.google.spanner.v1.Type.code) == [`STRUCT`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.STRUCT) , then `struct_type` provides type information for the struct's fields.                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `type_annotation`    | [`TypeAnnotationCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeAnnotationCode) The [`TypeAnnotationCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeAnnotationCode) that disambiguates SQL type that Spanner will use to represent values of this type during query processing. This is necessary for some type codes because a single [`TypeCode`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode) can be mapped to different SQL types depending on the SQL dialect. [`type_annotation`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.TypeAnnotationCode.google.spanner.v1.Type.type_annotation) typically is not needed to process the content of a value (it doesn't affect serialization) and clients can ignore it on the read path. |
| `proto_type_fqn`     | `string` If [`code`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.TypeCode.google.spanner.v1.Type.code) == [`PROTO`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.PROTO) or [`code`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.TypeCode.google.spanner.v1.Type.code) == [`ENUM`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.ENUM) , then `proto_type_fqn` is the fully qualified name of the proto type representing the proto/enum definition.                                                                                                                                                                                |

## TypeAnnotationCode

`TypeAnnotationCode` is used as a part of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) to disambiguate SQL types that should be used for a given Cloud Spanner value. Disambiguation is needed because the same Cloud Spanner type can be mapped to different SQL types depending on SQL dialect. TypeAnnotationCode doesn't affect the way value is serialized.

| Enums                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `TYPE_ANNOTATION_CODE_UNSPECIFIED` | Not specified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `PG_NUMERIC`                       | PostgreSQL compatible NUMERIC type. This annotation needs to be applied to [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) instances having [`NUMERIC`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.NUMERIC) type code to specify that values of this type should be treated as PostgreSQL NUMERIC values. Currently this annotation is always needed for [`NUMERIC`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.NUMERIC) when a client interacts with PostgreSQL-enabled Spanner databases. |
| `PG_JSONB`                         | PostgreSQL compatible JSONB type. This annotation needs to be applied to [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) instances having [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.JSON) type code to specify that values of this type should be treated as PostgreSQL JSONB values. Currently this annotation is always needed for [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.TypeCode.ENUM_VALUES.google.spanner.v1.TypeCode.JSON) when a client interacts with PostgreSQL-enabled Spanner databases.                 |
| `PG_OID`                           | PostgreSQL compatible OID type. This annotation can be used by a client interacting with PostgreSQL-enabled Spanner database to specify that a value should be treated using the semantics of the OID type.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

## TypeCode

`TypeCode` is used as part of [`Type`](https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type) to indicate the type of a Cloud Spanner value.

Each legal value of a type can be encoded to or decoded from a JSON value, using the encodings described below. All Cloud Spanner values can be `null` , regardless of type; `null` s are always encoded as a JSON `null` .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Enums</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>TYPE_CODE_UNSPECIFIED</code></td>
<td>Not specified.</td>
</tr>
<tr class="even">
<td><code>BOOL</code></td>
<td>Encoded as JSON <code>true</code> or <code>false</code> .</td>
</tr>
<tr class="odd">
<td><code>INT64</code></td>
<td>Encoded as <code>string</code> , in decimal format.</td>
</tr>
<tr class="even">
<td><code>FLOAT64</code></td>
<td>Encoded as <code>number</code> , or the strings <code>"NaN"</code> , <code>"Infinity"</code> , or <code>"-Infinity"</code> .</td>
</tr>
<tr class="odd">
<td><code>FLOAT32</code></td>
<td>Encoded as <code>number</code> , or the strings <code>"NaN"</code> , <code>"Infinity"</code> , or <code>"-Infinity"</code> .</td>
</tr>
<tr class="even">
<td><code>TIMESTAMP</code></td>
<td><p>Encoded as <code>string</code> in RFC 3339 timestamp format. The time zone must be present, and must be <code>"Z"</code> .</p>
<p>If the schema has the column option <code>allow_commit_timestamp=true</code> , the placeholder string <code>"spanner.commit_timestamp()"</code> can be used to instruct the system to insert the commit timestamp associated with the transaction commit.</p></td>
</tr>
<tr class="odd">
<td><code>DATE</code></td>
<td>Encoded as <code>string</code> in RFC 3339 date format.</td>
</tr>
<tr class="even">
<td><code>STRING</code></td>
<td>Encoded as <code>string</code> .</td>
</tr>
<tr class="odd">
<td><code>BYTES</code></td>
<td>Encoded as a base64-encoded <code>string</code> , as described in RFC 4648, section 4.</td>
</tr>
<tr class="even">
<td><code>ARRAY</code></td>
<td>Encoded as <code>list</code> , where the list elements are represented according to <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Type.FIELDS.google.spanner.v1.Type.google.spanner.v1.Type.array_element_type"><code>array_element_type</code></a> .</td>
</tr>
<tr class="odd">
<td><code>STRUCT</code></td>
<td>Encoded as <code>list</code> , where list element <code>i</code> is represented according to <a href="https://docs.cloud.google.com/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.StructType.FIELDS.repeated.google.spanner.v1.StructType.Field.google.spanner.v1.StructType.fields"><code>struct_type.fields[i]</code></a> .</td>
</tr>
<tr class="even">
<td><code>NUMERIC</code></td>
<td><p>Encoded as <code>string</code> , in decimal format or scientific notation format. Decimal format: <code>[+-]Digits[.[Digits]]</code> or <code>[+-][Digits].Digits</code></p>
<p>Scientific notation: <code>[+-]Digits[.[Digits]][ExponentIndicator[+-]Digits]</code> or <code>[+-][Digits].Digits[ExponentIndicator[+-]Digits]</code> (ExponentIndicator is <code>"e"</code> or <code>"E"</code> )</p></td>
</tr>
<tr class="odd">
<td><code>JSON</code></td>
<td><p>Encoded as a JSON-formatted <code>string</code> as described in RFC 7159. The following rules are applied when parsing JSON input:</p>
<ul>
<li>Whitespace characters are not preserved.</li>
<li>If a JSON object has duplicate keys, only the first key is preserved.</li>
<li>Members of a JSON object are not guaranteed to have their order preserved.</li>
<li>JSON array elements will have their order preserved.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>PROTO</code></td>
<td>Encoded as a base64-encoded <code>string</code> , as described in RFC 4648, section 4.</td>
</tr>
<tr class="odd">
<td><code>ENUM</code></td>
<td>Encoded as <code>string</code> , in decimal format.</td>
</tr>
<tr class="even">
<td><code>UUID</code></td>
<td>Encoded as <code>string</code> , in lower-case hexa-decimal format, as described in RFC 9562, section 4.</td>
</tr>
</tbody>
</table>
