## Index

  - `  Spanner  ` (interface)
  - `  BatchCreateSessionsRequest  ` (message)
  - `  BatchCreateSessionsResponse  ` (message)
  - `  BatchWriteRequest  ` (message)
  - `  BatchWriteRequest.MutationGroup  ` (message)
  - `  BatchWriteResponse  ` (message)
  - `  BeginTransactionRequest  ` (message)
  - `  CommitRequest  ` (message)
  - `  CommitResponse  ` (message)
  - `  CommitResponse.CommitStats  ` (message)
  - `  CreateSessionRequest  ` (message)
  - `  DeleteSessionRequest  ` (message)
  - `  DirectedReadOptions  ` (message)
  - `  DirectedReadOptions.ExcludeReplicas  ` (message)
  - `  DirectedReadOptions.IncludeReplicas  ` (message)
  - `  DirectedReadOptions.ReplicaSelection  ` (message)
  - `  DirectedReadOptions.ReplicaSelection.Type  ` (enum)
  - `  ExecuteBatchDmlRequest  ` (message)
  - `  ExecuteBatchDmlRequest.Statement  ` (message)
  - `  ExecuteBatchDmlResponse  ` (message)
  - `  ExecuteSqlRequest  ` (message)
  - `  ExecuteSqlRequest.QueryMode  ` (enum)
  - `  ExecuteSqlRequest.QueryOptions  ` (message)
  - `  GetSessionRequest  ` (message)
  - `  KeyRange  ` (message)
  - `  KeySet  ` (message)
  - `  ListSessionsRequest  ` (message)
  - `  ListSessionsResponse  ` (message)
  - `  MultiplexedSessionPrecommitToken  ` (message)
  - `  Mutation  ` (message)
  - `  Mutation.Delete  ` (message)
  - `  Mutation.Write  ` (message)
  - `  PartialResultSet  ` (message)
  - `  Partition  ` (message)
  - `  PartitionOptions  ` (message)
  - `  PartitionQueryRequest  ` (message)
  - `  PartitionReadRequest  ` (message)
  - `  PartitionResponse  ` (message)
  - `  PlanNode  ` (message)
  - `  PlanNode.ChildLink  ` (message)
  - `  PlanNode.Kind  ` (enum)
  - `  PlanNode.ShortRepresentation  ` (message)
  - `  QueryAdvisorResult  ` (message)
  - `  QueryAdvisorResult.IndexAdvice  ` (message)
  - `  QueryPlan  ` (message)
  - `  ReadRequest  ` (message)
  - `  RequestOptions  ` (message)
  - `  RequestOptions.Priority  ` (enum)
  - `  ResultSet  ` (message)
  - `  ResultSetMetadata  ` (message)
  - `  ResultSetStats  ` (message)
  - `  RollbackRequest  ` (message)
  - `  Session  ` (message)
  - `  StructType  ` (message)
  - `  StructType.Field  ` (message)
  - `  Transaction  ` (message)
  - `  TransactionOptions  ` (message)
  - `  TransactionOptions.IsolationLevel  ` (enum)
  - `  TransactionOptions.PartitionedDml  ` (message)
  - `  TransactionOptions.ReadOnly  ` (message)
  - `  TransactionOptions.ReadWrite  ` (message)
  - `  TransactionSelector  ` (message)
  - `  Type  ` (message)
  - `  TypeAnnotationCode  ` (enum)
  - `  TypeCode  ` (enum)

## Spanner

Cloud Spanner API

The Cloud Spanner API can be used to manage sessions and execute transactions on data stored in Cloud Spanner databases.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>BatchCreateSessions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc BatchCreateSessions(                         BatchCreateSessionsRequest            </code> ) returns ( <code dir="ltr" translate="no">              BatchCreateSessionsResponse            </code> )</p>
<p>Creates multiple new sessions.</p>
<p>This API can be used to initialize a session cache on the clients. See <a href="https://goo.gl/TgSFN2">https://goo.gl/TgSFN2</a> for best practices on session cache management.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>BatchWrite</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc BatchWrite(                         BatchWriteRequest            </code> ) returns ( <code dir="ltr" translate="no">              BatchWriteResponse            </code> )</p>
<p>Batches the supplied mutation groups in a collection of efficient transactions. All mutations in a group are committed atomically. However, mutations across groups can be committed non-atomically in an unspecified order and thus, they must be independent of each other. Partial failure is possible, that is, some groups might have been committed successfully, while some might have failed. The results of individual batches are streamed into the response as the batches are applied.</p>
<p><code dir="ltr" translate="no">           BatchWrite          </code> requests are not replay protected, meaning that each mutation group can be applied more than once. Replays of non-idempotent mutations can have undesirable effects. For example, replays of an insert mutation can produce an already exists error or if you use generated or commit timestamp-based keys, it can result in additional rows being added to the mutation's table. We recommend structuring your mutation groups to be idempotent to avoid this issue.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>BeginTransaction</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc BeginTransaction(                         BeginTransactionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Transaction            </code> )</p>
<p>Begins a new transaction. This step can often be skipped: <code dir="ltr" translate="no">             Read           </code> , <code dir="ltr" translate="no">             ExecuteSql           </code> and <code dir="ltr" translate="no">             Commit           </code> can begin a new transaction as a side-effect.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Commit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc Commit(                         CommitRequest            </code> ) returns ( <code dir="ltr" translate="no">              CommitResponse            </code> )</p>
<p>Commits a transaction. The request includes the mutations to be applied to rows in the database.</p>
<p><code dir="ltr" translate="no">           Commit          </code> might return an <code dir="ltr" translate="no">           ABORTED          </code> error. This can occur at any time; commonly, the cause is conflicts with concurrent transactions. However, it can also happen for a variety of other reasons. If <code dir="ltr" translate="no">           Commit          </code> returns <code dir="ltr" translate="no">           ABORTED          </code> , the caller should retry the transaction from the beginning, reusing the same session.</p>
<p>On very rare occasions, <code dir="ltr" translate="no">           Commit          </code> might return <code dir="ltr" translate="no">           UNKNOWN          </code> . This can happen, for example, if the client job experiences a 1+ hour networking failure. At that point, Cloud Spanner has lost track of the transaction outcome and we recommend that you perform another read from the database to see the state of things as they are now.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>CreateSession</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc CreateSession(                         CreateSessionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Session            </code> )</p>
<p>Creates a new session. A session can be used to perform transactions that read and/or modify data in a Cloud Spanner database. Sessions are meant to be reused for many consecutive transactions.</p>
<p>Sessions can only execute one transaction at a time. To execute multiple concurrent read-write/write-only transactions, create multiple sessions. Note that standalone reads and queries use a transaction internally, and count toward the one transaction limit.</p>
<p>Active sessions use additional server resources, so it's a good idea to delete idle and unneeded sessions. Aside from explicit deletes, Cloud Spanner can delete sessions when no operations are sent for more than an hour. If a session is deleted, requests to it return <code dir="ltr" translate="no">           NOT_FOUND          </code> .</p>
<p>Idle sessions can be kept alive by sending a trivial SQL query periodically, for example, <code dir="ltr" translate="no">           "SELECT 1"          </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>DeleteSession</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc DeleteSession(                         DeleteSessionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Ends a session, releasing server resources associated with it. This asynchronously triggers the cancellation of any operations that are running with this session.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ExecuteBatchDml</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ExecuteBatchDml(                         ExecuteBatchDmlRequest            </code> ) returns ( <code dir="ltr" translate="no">              ExecuteBatchDmlResponse            </code> )</p>
<p>Executes a batch of SQL DML statements. This method allows many statements to be run with lower latency than submitting them sequentially with <code dir="ltr" translate="no">             ExecuteSql           </code> .</p>
<p>Statements are executed in sequential order. A request can succeed even if a statement fails. The <code dir="ltr" translate="no">             ExecuteBatchDmlResponse.status           </code> field in the response provides information about the statement that failed. Clients must inspect this field to determine whether an error occurred.</p>
<p>Execution stops after the first failed statement; the remaining statements are not executed.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ExecuteSql</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ExecuteSql(                         ExecuteSqlRequest            </code> ) returns ( <code dir="ltr" translate="no">              ResultSet            </code> )</p>
<p>Executes an SQL statement, returning all results in a single reply. This method can't be used to return a result set larger than 10 MiB; if the query yields more data than that, the query fails with a <code dir="ltr" translate="no">           FAILED_PRECONDITION          </code> error.</p>
<p>Operations inside read-write transactions might return <code dir="ltr" translate="no">           ABORTED          </code> . If this occurs, the application should restart the transaction from the beginning. See <code dir="ltr" translate="no">             Transaction           </code> for more details.</p>
<p>Larger result sets can be fetched in streaming fashion by calling <code dir="ltr" translate="no">             ExecuteStreamingSql           </code> instead.</p>
<p>The query string can be SQL or <a href="https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro">Graph Query Language (GQL)</a> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ExecuteStreamingSql</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ExecuteStreamingSql(                         ExecuteSqlRequest            </code> ) returns ( <code dir="ltr" translate="no">              PartialResultSet            </code> )</p>
<p>Like <code dir="ltr" translate="no">             ExecuteSql           </code> , except returns the result set as a stream. Unlike <code dir="ltr" translate="no">             ExecuteSql           </code> , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.</p>
<p>The query string can be SQL or <a href="https://cloud.google.com/spanner/docs/reference/standard-sql/graph-intro">Graph Query Language (GQL)</a> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>GetSession</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc GetSession(                         GetSessionRequest            </code> ) returns ( <code dir="ltr" translate="no">              Session            </code> )</p>
<p>Gets a session. Returns <code dir="ltr" translate="no">           NOT_FOUND          </code> if the session doesn't exist. This is mainly useful for determining whether a session is still alive.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>ListSessions</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc ListSessions(                         ListSessionsRequest            </code> ) returns ( <code dir="ltr" translate="no">              ListSessionsResponse            </code> )</p>
<p>Lists all sessions in a given database.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>PartitionQuery</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc PartitionQuery(                         PartitionQueryRequest            </code> ) returns ( <code dir="ltr" translate="no">              PartitionResponse            </code> )</p>
<p>Creates a set of partition tokens that can be used to execute a query operation in parallel. Each of the returned partition tokens can be used by <code dir="ltr" translate="no">             ExecuteStreamingSql           </code> to specify a subset of the query result to read. The same session and read-only transaction must be used by the <code dir="ltr" translate="no">           PartitionQueryRequest          </code> used to create the partition tokens and the <code dir="ltr" translate="no">           ExecuteSqlRequests          </code> that use the partition tokens.</p>
<p>Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the query, and the whole operation must be restarted from the beginning.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>PartitionRead</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc PartitionRead(                         PartitionReadRequest            </code> ) returns ( <code dir="ltr" translate="no">              PartitionResponse            </code> )</p>
<p>Creates a set of partition tokens that can be used to execute a read operation in parallel. Each of the returned partition tokens can be used by <code dir="ltr" translate="no">             StreamingRead           </code> to specify a subset of the read result to read. The same session and read-only transaction must be used by the <code dir="ltr" translate="no">           PartitionReadRequest          </code> used to create the partition tokens and the <code dir="ltr" translate="no">           ReadRequests          </code> that use the partition tokens. There are no ordering guarantees on rows returned among the returned partition tokens, or even within each individual <code dir="ltr" translate="no">           StreamingRead          </code> call issued with a <code dir="ltr" translate="no">           partition_token          </code> .</p>
<p>Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the read, and the whole operation must be restarted from the beginning.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Read</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc Read(                         ReadRequest            </code> ) returns ( <code dir="ltr" translate="no">              ResultSet            </code> )</p>
<p>Reads rows from the database using key lookups and scans, as a simple key/value style alternative to <code dir="ltr" translate="no">             ExecuteSql           </code> . This method can't be used to return a result set larger than 10 MiB; if the read matches more data than that, the read fails with a <code dir="ltr" translate="no">           FAILED_PRECONDITION          </code> error.</p>
<p>Reads inside read-write transactions might return <code dir="ltr" translate="no">           ABORTED          </code> . If this occurs, the application should restart the transaction from the beginning. See <code dir="ltr" translate="no">             Transaction           </code> for more details.</p>
<p>Larger result sets can be yielded in streaming fashion by calling <code dir="ltr" translate="no">             StreamingRead           </code> instead.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Rollback</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc Rollback(                         RollbackRequest            </code> ) returns ( <code dir="ltr" translate="no">              Empty            </code> )</p>
<p>Rolls back a transaction, releasing any locks it holds. It's a good idea to call this for any transaction that includes one or more <code dir="ltr" translate="no">             Read           </code> or <code dir="ltr" translate="no">             ExecuteSql           </code> requests and ultimately decides not to commit.</p>
<p><code dir="ltr" translate="no">           Rollback          </code> returns <code dir="ltr" translate="no">           OK          </code> if it successfully aborts the transaction, the transaction was already aborted, or the transaction isn't found. <code dir="ltr" translate="no">           Rollback          </code> never returns <code dir="ltr" translate="no">           ABORTED          </code> .</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>StreamingRead</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p><code dir="ltr" translate="no">           rpc StreamingRead(                         ReadRequest            </code> ) returns ( <code dir="ltr" translate="no">              PartialResultSet            </code> )</p>
<p>Like <code dir="ltr" translate="no">             Read           </code> , except returns the result set as a stream. Unlike <code dir="ltr" translate="no">             Read           </code> , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.</p>
<dl>
<dt>Authorization scopes</dt>
<dd><p>Requires one of the following OAuth scopes:</p>
<ul>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/spanner.data             </code></li>
<li><code dir="ltr" translate="no">              https://www.googleapis.com/auth/cloud-platform             </code></li>
</ul>
<p>For more information, see the <a href="/docs/authentication#authorization-gcp">Authentication Overview</a> .</p>
</dd>
</dl></td>
</tr>
</tbody>
</table>

## BatchCreateSessionsRequest

The request for `  BatchCreateSessions  ` .

Fields

`  database  `

`  string  `

Required. The database in which the new sessions are created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.sessions.create  `

`  session_template  `

`  Session  `

Parameters to apply to each created session.

`  session_count  `

`  int32  `

Required. The number of sessions to be created in this batch call. The API can return fewer than the requested number of sessions. If a specific number of sessions are desired, the client can make additional calls to `  BatchCreateSessions  ` (adjusting `  session_count  ` as necessary).

## BatchCreateSessionsResponse

The response for `  BatchCreateSessions  ` .

Fields

`  session[]  `

`  Session  `

The freshly created sessions.

## BatchWriteRequest

The request for `  BatchWrite  ` .

Fields

`  session  `

`  string  `

Required. The session in which the batch request is to be run.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.write  `

`  request_options  `

`  RequestOptions  `

Common options for this request.

`  mutation_groups[]  `

`  MutationGroup  `

Required. The groups of mutations to be applied.

`  exclude_txn_from_change_streams  `

`  bool  `

Optional. If you don't set the `  exclude_txn_from_change_streams  ` option or if it's set to `  false  ` , then any change streams monitoring columns modified by transactions will capture the updates made within that transaction.

## MutationGroup

A group of mutations to be committed together. Related mutations should be placed in a group. For example, two mutations inserting rows with the same primary key prefix in both parent and child tables are related.

Fields

`  mutations[]  `

`  Mutation  `

Required. The mutations in this group.

## BatchWriteResponse

The result of applying a batch of mutations.

Fields

`  indexes[]  `

`  int32  `

The mutation groups applied in this batch. The values index into the `  mutation_groups  ` field in the corresponding `  BatchWriteRequest  ` .

`  status  `

`  Status  `

An `  OK  ` status indicates success. Any other status indicates a failure.

`  commit_timestamp  `

`  Timestamp  `

The commit timestamp of the transaction that applied this batch. Present if `  status  ` is `  OK  ` , absent otherwise.

## BeginTransactionRequest

The request for `  BeginTransaction  ` .

Fields

`  session  `

`  string  `

Required. The session in which the transaction runs.

Authorization requires one or more of the following [IAM](https://cloud.google.com/iam/docs/) permissions on the specified resource `  session  ` :

  - `  spanner.databases.beginReadOnlyTransaction  `
  - `  spanner.databases.beginOrRollbackReadWriteTransaction  `

`  options  `

`  TransactionOptions  `

Required. Options for the new transaction.

`  request_options  `

`  RequestOptions  `

Common options for this request. Priority is ignored for this request. Setting the priority in this `  request_options  ` struct doesn't do anything. To set the priority for a transaction, set it on the reads and writes that are part of this transaction instead.

`  mutation_key  `

`  Mutation  `

Optional. Required for read-write transactions on a multiplexed session that commit mutations but don't perform any reads or queries. You must randomly select one of the mutations from the mutation set and send it as a part of this request.

## CommitRequest

The request for `  Commit  ` .

Fields

`  session  `

`  string  `

Required. The session in which the transaction to be committed is running.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.write  `

`  mutations[]  `

`  Mutation  `

The mutations to be executed when this transaction commits. All mutations are applied atomically, in the order they appear in this list.

`  return_commit_stats  `

`  bool  `

If `  true  ` , then statistics related to the transaction is included in the `  CommitResponse  ` . Default value is `  false  ` .

`  max_commit_delay  `

`  Duration  `

Optional. The amount of latency this request is configured to incur in order to improve throughput. If this field isn't set, Spanner assumes requests are relatively latency sensitive and automatically determines an appropriate delay time. You can specify a commit delay value between 0 and 500 ms.

`  request_options  `

`  RequestOptions  `

Common options for this request.

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

Optional. If the read-write transaction was executed on a multiplexed session, then you must include the precommit token with the highest sequence number received in this transaction attempt. Failing to do so results in a `  FailedPrecondition  ` error.

Union field `  transaction  ` . Required. The transaction in which to commit. `  transaction  ` can be only one of the following:

`  transaction_id  `

`  bytes  `

Commit a previously-started transaction.

`  single_use_transaction  `

`  TransactionOptions  `

Execute mutations in a temporary transaction. Note that unlike commit of a previously-started transaction, commit with a temporary transaction is non-idempotent. That is, if the `  CommitRequest  ` is sent to Cloud Spanner more than once (for instance, due to retries in the application, or in the transport library), it's possible that the mutations are executed more than once. If this is undesirable, use `  BeginTransaction  ` and `  Commit  ` instead.

## CommitResponse

The response for `  Commit  ` .

Fields

`  commit_timestamp  `

`  Timestamp  `

The Cloud Spanner timestamp at which the transaction committed.

`  commit_stats  `

`  CommitStats  `

The statistics about this `  Commit  ` . Not returned by default. For more information, see `  CommitRequest.return_commit_stats  ` .

Union field `  MultiplexedSessionRetry  ` . You must examine and retry the commit if the following is populated. `  MultiplexedSessionRetry  ` can be only one of the following:

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

If specified, transaction has not committed yet. You must retry the commit with the new precommit token.

## CommitStats

Additional statistics about a commit.

Fields

`  mutation_count  `

`  int64  `

The total number of mutations for the transaction. Knowing the `  mutation_count  ` value can help you maximize the number of mutations in a transaction and minimize the number of API round trips. You can also monitor this value to prevent transactions from exceeding the system [limit](https://cloud.google.com/spanner/quotas#limits_for_creating_reading_updating_and_deleting_data) . If the number of mutations exceeds the limit, the server returns [INVALID\_ARGUMENT](https://cloud.google.com/spanner/docs/reference/rest/v1/Code#ENUM_VALUES.INVALID_ARGUMENT) .

## CreateSessionRequest

The request for `  CreateSession  ` .

Fields

`  database  `

`  string  `

Required. The database in which the new session is created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.sessions.create  `

`  session  `

`  Session  `

Required. The session to create.

## DeleteSessionRequest

The request for `  DeleteSession  ` .

Fields

`  name  `

`  string  `

Required. The name of the session to delete.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.sessions.delete  `

## DirectedReadOptions

The `  DirectedReadOptions  ` can be used to indicate which replicas or regions should be used for non-transactional reads or queries.

`  DirectedReadOptions  ` can only be specified for a read-only transaction, otherwise the API returns an `  INVALID_ARGUMENT  ` error.

Fields

Union field `  replicas  ` . Required. At most one of either `  include_replicas  ` or `  exclude_replicas  ` should be present in the message. `  replicas  ` can be only one of the following:

`  include_replicas  `

`  IncludeReplicas  `

`  Include_replicas  ` indicates the order of replicas (as they appear in this list) to process the request. If `  auto_failover_disabled  ` is set to `  true  ` and all replicas are exhausted without finding a healthy replica, Spanner waits for a replica in the list to become available, requests might fail due to `  DEADLINE_EXCEEDED  ` errors.

`  exclude_replicas  `

`  ExcludeReplicas  `

`  Exclude_replicas  ` indicates that specified replicas should be excluded from serving requests. Spanner doesn't route requests to the replicas in this list.

## ExcludeReplicas

An ExcludeReplicas contains a repeated set of ReplicaSelection that should be excluded from serving requests.

Fields

`  replica_selections[]  `

`  ReplicaSelection  `

The directed read replica selector.

## IncludeReplicas

An `  IncludeReplicas  ` contains a repeated set of `  ReplicaSelection  ` which indicates the order in which replicas should be considered.

Fields

`  replica_selections[]  `

`  ReplicaSelection  `

The directed read replica selector.

`  auto_failover_disabled  `

`  bool  `

If `  true  ` , Spanner doesn't route requests to a replica outside the \< `  include_replicas  ` list when all of the specified replicas are unavailable or unhealthy. Default value is `  false  ` .

## ReplicaSelection

The directed read replica selector. Callers must provide one or more of the following fields for replica selection:

  - `  location  ` - The location must be one of the regions within the multi-region configuration of your database.
  - `  type  ` - The type of the replica.

Some examples of using replica\_selectors are:

  - `  location:us-east1  ` --\> The "us-east1" replica(s) of any available type is used to process the request.
  - `  type:READ_ONLY  ` --\> The "READ\_ONLY" type replica(s) in the nearest available location are used to process the request.
  - `  location:us-east1 type:READ_ONLY  ` --\> The "READ\_ONLY" type replica(s) in location "us-east1" is used to process the request.

Fields

`  location  `

`  string  `

The location or region of the serving requests, for example, "us-east1".

`  type  `

`  Type  `

The type of replica.

## Type

Indicates the type of replica.

Enums

`  TYPE_UNSPECIFIED  `

Not specified.

`  READ_WRITE  `

Read-write replicas support both reads and writes.

`  READ_ONLY  `

Read-only replicas only support reads (not writes).

## ExecuteBatchDmlRequest

The request for `  ExecuteBatchDml  ` .

Fields

`  session  `

`  string  `

Required. The session in which the DML statements should be performed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.write  `

`  transaction  `

`  TransactionSelector  `

Required. The transaction to use. Must be a read-write transaction.

To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.

`  statements[]  `

`  Statement  `

Required. The list of statements to execute in this batch. Statements are executed serially, such that the effects of statement `  i  ` are visible to statement `  i+1  ` . Each statement must be a DML statement. Execution stops at the first failed statement; the remaining statements are not executed.

Callers must provide at least one statement.

`  seqno  `

`  int64  `

Required. A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.

The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction might be aborted. Replays of previously handled requests yield the same response as the first execution.

`  request_options  `

`  RequestOptions  `

Common options for this request.

`  last_statements  `

`  bool  `

Optional. If set to `  true  ` , this request marks the end of the transaction. After these statements execute, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.

Setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of statements shouldn't be assumed until a subsequent `  Commit  ` call completes successfully.

## Statement

A single DML statement.

Fields

`  sql  `

`  string  `

Required. The DML string.

`  params  `

`  Struct  `

Parameter names and values that bind to placeholders in the DML string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names can contain letters, numbers, and underscores.

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  param_types  `

`  map<string, Type  ` \>

It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, `  param_types  ` can be used to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of `  Type  ` for more information about SQL types.

## ExecuteBatchDmlResponse

The response for `  ExecuteBatchDml  ` . Contains a list of `  ResultSet  ` messages, one for each DML statement that has successfully executed, in the same order as the statements in the request. If a statement fails, the status in the response body identifies the cause of the failure.

To check for DML statements that failed, use the following approach:

1.  Check the status in the response message. The `  google.rpc.Code  ` enum value `  OK  ` indicates that all statements were executed successfully.
2.  If the status was not `  OK  ` , check the number of result sets in the response. If the response contains `  N  ` `  ResultSet  ` messages, then statement `  N+1  ` in the request failed.

Example 1:

  - Request: 5 DML statements, all executed successfully.
  - Response: 5 `  ResultSet  ` messages, with the status `  OK  ` .

Example 2:

  - Request: 5 DML statements. The third statement has a syntax error.
  - Response: 2 `  ResultSet  ` messages, and a syntax error ( `  INVALID_ARGUMENT  ` ) status. The number of `  ResultSet  ` messages indicates that the third statement failed, and the fourth and fifth statements were not executed.

Fields

`  result_sets[]  `

`  ResultSet  `

One `  ResultSet  ` for each statement in the request that ran successfully, in the same order as the statements in the request. Each `  ResultSet  ` does not contain any rows. The `  ResultSetStats  ` in each `  ResultSet  ` contain the number of rows modified by the statement.

Only the first `  ResultSet  ` in the response contains valid `  ResultSetMetadata  ` .

`  status  `

`  Status  `

If all DML statements are executed successfully, the status is `  OK  ` . Otherwise, the error status of the first failed statement.

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt should be passed to the `  Commit  ` request for this transaction.

## ExecuteSqlRequest

The request for `  ExecuteSql  ` and `  ExecuteStreamingSql  ` .

Fields

`  session  `

`  string  `

Required. The session in which the SQL query should be performed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.select  `

`  transaction  `

`  TransactionSelector  `

The transaction to use.

For queries, if none is provided, the default is a temporary read-only transaction with strong concurrency.

Standard DML statements require a read-write transaction. To protect against replays, single-use transactions are not supported. The caller must either supply an existing transaction ID or begin a new transaction.

Partitioned DML requires an existing Partitioned DML transaction ID.

`  sql  `

`  string  `

Required. The SQL string.

`  params  `

`  Struct  `

Parameter names and values that bind to placeholders in the SQL string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names must conform to the naming requirements of identifiers as specified at <https://cloud.google.com/spanner/docs/lexical#identifiers> .

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  param_types  `

`  map<string, Type  ` \>

It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, you can use `  param_types  ` to specify the exact SQL type for some or all of the SQL statement parameters. See the definition of `  Type  ` for more information about SQL types.

`  resume_token  `

`  bytes  `

If this request is resuming a previously interrupted SQL statement execution, `  resume_token  ` should be copied from the last `  PartialResultSet  ` yielded before the interruption. Doing this enables the new SQL statement execution to resume where the last one left off. The rest of the request parameters must exactly match the request that yielded this token.

`  query_mode  `

`  QueryMode  `

Used to control the amount of debugging information returned in `  ResultSetStats  ` . If `  partition_token  ` is set, `  query_mode  ` can only be set to `  QueryMode.NORMAL  ` .

`  partition_token  `

`  bytes  `

If present, results are restricted to the specified partition previously created using `  PartitionQuery  ` . There must be an exact match for the values of fields common to this message and the `  PartitionQueryRequest  ` message used to create this `  partition_token  ` .

`  seqno  `

`  int64  `

A per-transaction sequence number used to identify this request. This field makes each request idempotent such that if the request is received multiple times, at most one succeeds.

The sequence number must be monotonically increasing within the transaction. If a request arrives for the first time with an out-of-order sequence number, the transaction can be aborted. Replays of previously handled requests yield the same response as the first execution.

Required for DML statements. Ignored for queries.

`  query_options  `

`  QueryOptions  `

Query optimizer configuration to use for the given query.

`  request_options  `

`  RequestOptions  `

Common options for this request.

`  directed_read_options  `

`  DirectedReadOptions  `

Directed read options for this request.

`  data_boost_enabled  `

`  bool  `

If this is for a partitioned query and this field is set to `  true  ` , the request is executed with Spanner Data Boost independent compute resources.

If the field is set to `  true  ` but the request doesn't set `  partition_token  ` , the API returns an `  INVALID_ARGUMENT  ` error.

`  last_statement  `

`  bool  `

Optional. If set to `  true  ` , this statement marks the end of the transaction. After this statement executes, you must commit or abort the transaction. Attempts to execute any other requests against this transaction (including reads and queries) are rejected.

For DML statements, setting this option might cause some error reporting to be deferred until commit time (for example, validation of unique constraints). Given this, successful execution of a DML statement shouldn't be assumed until a subsequent `  Commit  ` call completes successfully.

## QueryMode

Mode in which the statement must be processed.

Enums

`  NORMAL  `

The default mode. Only the statement results are returned.

`  PLAN  `

This mode returns only the query plan, without any results or execution statistics information.

`  PROFILE  `

This mode returns the query plan, overall execution statistics, operator level execution statistics along with the results. This has a performance overhead compared to the other modes. It isn't recommended to use this mode for production traffic.

`  WITH_STATS  `

This mode returns the overall (but not operator-level) execution statistics along with the results.

`  WITH_PLAN_AND_STATS  `

This mode returns the query plan, overall (but not operator-level) execution statistics along with the results.

## QueryOptions

Query optimizer configuration.

Fields

`  optimizer_version  `

`  string  `

An option to control the selection of optimizer version.

This parameter allows individual queries to pick different query optimizer versions.

Specifying `  latest  ` as a value instructs Cloud Spanner to use the latest supported query optimizer version. If not specified, Cloud Spanner uses the optimizer version set at the database level options. Any other positive integer (from the list of supported optimizer versions) overrides the default optimizer version for query execution.

The list of supported optimizer versions can be queried from `  SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS  ` .

Executing a SQL statement with an invalid optimizer version fails with an `  INVALID_ARGUMENT  ` error.

See <https://cloud.google.com/spanner/docs/query-optimizer/manage-query-optimizer> for more information on managing the query optimizer.

The `  optimizer_version  ` statement hint has precedence over this setting.

`  optimizer_statistics_package  `

`  string  `

An option to control the selection of optimizer statistics package.

This parameter allows individual queries to use a different query optimizer statistics package.

Specifying `  latest  ` as a value instructs Cloud Spanner to use the latest generated statistics package. If not specified, Cloud Spanner uses the statistics package set at the database level options, or the latest package if the database option isn't set.

The statistics package requested by the query has to be exempt from garbage collection. This can be achieved with the following DDL statement:

``` text
ALTER STATISTICS <package_name> SET OPTIONS (allow_gc=false)
```

The list of available statistics packages can be queried from `  INFORMATION_SCHEMA.SPANNER_STATISTICS  ` .

Executing a SQL statement with an invalid optimizer statistics package or with a statistics package that allows garbage collection fails with an `  INVALID_ARGUMENT  ` error.

## GetSessionRequest

The request for `  GetSession  ` .

Fields

`  name  `

`  string  `

Required. The name of the session to retrieve.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.sessions.get  `

## KeyRange

KeyRange represents a range of rows in a table or index.

A range has a start key and an end key. These keys can be open or closed, indicating if the range includes rows with that key.

Keys are represented by lists, where the ith value in the list corresponds to the ith component of the table or index primary key. Individual values are encoded as described `  here  ` .

For example, consider the following table definition:

``` text
CREATE TABLE UserEvents (
  UserName STRING(MAX),
  EventDate STRING(10)
) PRIMARY KEY(UserName, EventDate);
```

The following keys name rows in this table:

``` text
["Bob", "2014-09-23"]
["Alfred", "2015-06-12"]
```

Since the `  UserEvents  ` table's `  PRIMARY KEY  ` clause names two columns, each `  UserEvents  ` key has two elements; the first is the `  UserName  ` , and the second is the `  EventDate  ` .

Key ranges with multiple components are interpreted lexicographically by component using the table or index key's declared sort order. For example, the following range returns all events for user `  "Bob"  ` that occurred in the year 2015:

``` text
"start_closed": ["Bob", "2015-01-01"]
"end_closed": ["Bob", "2015-12-31"]
```

Start and end keys can omit trailing key components. This affects the inclusion and exclusion of rows that exactly match the provided key components: if the key is closed, then rows that exactly match the provided components are included; if the key is open, then rows that exactly match are not included.

For example, the following range includes all events for `  "Bob"  ` that occurred during and after the year 2000:

``` text
"start_closed": ["Bob", "2000-01-01"]
"end_closed": ["Bob"]
```

The next example retrieves all events for `  "Bob"  ` :

``` text
"start_closed": ["Bob"]
"end_closed": ["Bob"]
```

To retrieve events before the year 2000:

``` text
"start_closed": ["Bob"]
"end_open": ["Bob", "2000-01-01"]
```

The following range includes all rows in the table:

``` text
"start_closed": []
"end_closed": []
```

This range returns all users whose `  UserName  ` begins with any character from A to C:

``` text
"start_closed": ["A"]
"end_open": ["D"]
```

This range returns all users whose `  UserName  ` begins with B:

``` text
"start_closed": ["B"]
"end_open": ["C"]
```

Key ranges honor column sort order. For example, suppose a table is defined as follows:

``` text
CREATE TABLE DescendingSortedTable {
  Key INT64,
  ...
) PRIMARY KEY(Key DESC);
```

The following range retrieves all rows with key values between 1 and 100 inclusive:

``` text
"start_closed": ["100"]
"end_closed": ["1"]
```

Note that 100 is passed as the start, and 1 is passed as the end, because `  Key  ` is a descending column in the schema.

Fields

Union field `  start_key_type  ` . The start key must be provided. It can be either closed or open. `  start_key_type  ` can be only one of the following:

`  start_closed  `

`  ListValue  `

If the start is closed, then the range includes all rows whose first `  len(start_closed)  ` key columns exactly match `  start_closed  ` .

`  start_open  `

`  ListValue  `

If the start is open, then the range excludes rows whose first `  len(start_open)  ` key columns exactly match `  start_open  ` .

Union field `  end_key_type  ` . The end key must be provided. It can be either closed or open. `  end_key_type  ` can be only one of the following:

`  end_closed  `

`  ListValue  `

If the end is closed, then the range includes all rows whose first `  len(end_closed)  ` key columns exactly match `  end_closed  ` .

`  end_open  `

`  ListValue  `

If the end is open, then the range excludes rows whose first `  len(end_open)  ` key columns exactly match `  end_open  ` .

## KeySet

`  KeySet  ` defines a collection of Cloud Spanner keys and/or key ranges. All the keys are expected to be in the same table or index. The keys need not be sorted in any particular way.

If the same key is specified multiple times in the set (for example if two ranges, two keys, or a key and a range overlap), Cloud Spanner behaves as if the key were only specified once.

Fields

`  keys[]  `

`  ListValue  `

A list of specific keys. Entries in `  keys  ` should have exactly as many elements as there are columns in the primary or index key with which this `  KeySet  ` is used. Individual key values are encoded as described `  here  ` .

`  ranges[]  `

`  KeyRange  `

A list of key ranges. See `  KeyRange  ` for more information about key range specifications.

`  all  `

`  bool  `

For convenience `  all  ` can be set to `  true  ` to indicate that this `  KeySet  ` matches all keys in the table or index. Note that any keys specified in `  keys  ` or `  ranges  ` are only yielded once.

## ListSessionsRequest

The request for `  ListSessions  ` .

Fields

`  database  `

`  string  `

Required. The database in which to list sessions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.sessions.list  `

`  page_size  `

`  int32  `

Number of sessions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  page_token  `

`  string  `

If non-empty, `  page_token  ` should contain a `  next_page_token  ` from a previous `  ListSessionsResponse  ` .

`  filter  `

`  string  `

An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:

  - `  labels.key  ` where key is the name of a label

Some examples of using filters are:

  - `  labels.env:*  ` --\> The session has the label "env".
  - `  labels.env:dev  ` --\> The session has the label "env" and the value of the label contains the string "dev".

## ListSessionsResponse

The response for `  ListSessions  ` .

Fields

`  sessions[]  `

`  Session  `

The list of requested sessions.

`  next_page_token  `

`  string  `

`  next_page_token  ` can be sent in a subsequent `  ListSessions  ` call to fetch more of the matching sessions.

## MultiplexedSessionPrecommitToken

When a read-write transaction is executed on a multiplexed session, this precommit token is sent back to the client as a part of the `  Transaction  ` message in the `  BeginTransaction  ` response and also as a part of the `  ResultSet  ` and `  PartialResultSet  ` responses.

Fields

`  precommit_token  `

`  bytes  `

Opaque precommit token.

`  seq_num  `

`  int32  `

An incrementing seq number is generated on every precommit token that is returned. Clients should remember the precommit token with the highest sequence number from the current transaction attempt.

## Mutation

A modification to one or more Cloud Spanner rows. Mutations can be applied to a Cloud Spanner database by sending them in a `  Commit  ` call.

Fields

Union field `  operation  ` . Required. The operation to perform. `  operation  ` can be only one of the following:

`  insert  `

`  Write  `

Insert new rows in a table. If any of the rows already exist, the write or transaction fails with error `  ALREADY_EXISTS  ` .

`  update  `

`  Write  `

Update existing rows in a table. If any of the rows does not already exist, the transaction fails with error `  NOT_FOUND  ` .

`  insert_or_update  `

`  Write  `

Like `  insert  ` , except that if the row already exists, then its column values are overwritten with the ones provided. Any column values not explicitly written are preserved.

When using `  insert_or_update  ` , just as when using `  insert  ` , all `  NOT NULL  ` columns in the table must be given a value. This holds true even when the row already exists and will therefore actually be updated.

`  replace  `

`  Write  `

Like `  insert  ` , except that if the row already exists, it is deleted, and the column values provided are inserted instead. Unlike `  insert_or_update  ` , this means any values not explicitly written become `  NULL  ` .

In an interleaved table, if you create the child table with the `  ON DELETE CASCADE  ` annotation, then replacing a parent row also deletes the child rows. Otherwise, you must delete the child rows before you replace the parent row.

`  delete  `

`  Delete  `

Delete rows from a table. Succeeds whether or not the named rows were present.

## Delete

Arguments to `  delete  ` operations.

Fields

`  table  `

`  string  `

Required. The table whose rows will be deleted.

`  key_set  `

`  KeySet  `

Required. The primary keys of the rows within `  table  ` to delete. The primary keys must be specified in the order in which they appear in the `  PRIMARY KEY()  ` clause of the table's equivalent DDL statement (the DDL statement used to create the table). Delete is idempotent. The transaction will succeed even if some or all rows do not exist.

## Write

Arguments to `  insert  ` , `  update  ` , `  insert_or_update  ` , and `  replace  ` operations.

Fields

`  table  `

`  string  `

Required. The table whose rows will be written.

`  columns[]  `

`  string  `

The names of the columns in `  table  ` to be written.

The list of columns must contain enough columns to allow Cloud Spanner to derive values for all primary key columns in the row(s) to be modified.

`  values[]  `

`  ListValue  `

The values to be written. `  values  ` can contain more than one list of values. If it does, then multiple rows are written, one for each entry in `  values  ` . Each list in `  values  ` must have exactly as many entries as there are entries in `  columns  ` above. Sending multiple lists is equivalent to sending multiple `  Mutation  ` s, each containing one `  values  ` entry and repeating `  table  ` and `  columns  ` . Individual values in each list are encoded as described `  here  ` .

## PartialResultSet

Partial results from a streaming read or SQL query. Streaming reads and SQL queries better tolerate large result sets, large rows, and large values, but are a little trickier to consume.

Fields

`  metadata  `

`  ResultSetMetadata  `

Metadata about the result set, such as row type information. Only present in the first response.

`  values[]  `

`  Value  `

A streamed result set consists of a stream of values, which might be split into many `  PartialResultSet  ` messages to accommodate large rows and/or large values. Every N complete values defines a row, where N is equal to the number of entries in `  metadata.row_type.fields  ` .

Most values are encoded based on type as described `  here  ` .

It's possible that the last value in values is "chunked", meaning that the rest of the value is sent in subsequent `  PartialResultSet  ` (s). This is denoted by the `  chunked_value  ` field. Two or more chunked values can be merged to form a complete value as follows:

  - `  bool/number/null  ` : can't be chunked
  - `  string  ` : concatenate the strings
  - `  list  ` : concatenate the lists. If the last element in a list is a `  string  ` , `  list  ` , or `  object  ` , merge it with the first element in the next list by applying these rules recursively.
  - `  object  ` : concatenate the (field name, field value) pairs. If a field name is duplicated, then apply these rules recursively to merge the field values.

Some examples of merging:

``` text
Strings are concatenated.
"foo", "bar" => "foobar"

Lists of non-strings are concatenated.
[2, 3], [4] => [2, 3, 4]

Lists are concatenated, but the last and first elements are merged
because they are strings.
["a", "b"], ["c", "d"] => ["a", "bc", "d"]

Lists are concatenated, but the last and first elements are merged
because they are lists. Recursively, the last and first elements
of the inner lists are merged because they are strings.
["a", ["b", "c"]], [["d"], "e"] => ["a", ["b", "cd"], "e"]

Non-overlapping object fields are combined.
{"a": "1"}, {"b": "2"} => {"a": "1", "b": 2"}

Overlapping object fields are merged.
{"a": "1"}, {"a": "2"} => {"a": "12"}

Examples of merging objects containing lists of strings.
{"a": ["1"]}, {"a": ["2"]} => {"a": ["12"]}
```

For a more complete example, suppose a streaming SQL query is yielding a result set whose rows contain a single string field. The following `  PartialResultSet  ` s might be yielded:

``` text
{
  "metadata": { ... }
  "values": ["Hello", "W"]
  "chunked_value": true
  "resume_token": "Af65..."
}
{
  "values": ["orl"]
  "chunked_value": true
}
{
  "values": ["d"]
  "resume_token": "Zx1B..."
}
```

This sequence of `  PartialResultSet  ` s encodes two rows, one containing the field value `  "Hello"  ` , and a second containing the field value `  "World" = "W" + "orl" + "d"  ` .

Not all `  PartialResultSet  ` s contain a `  resume_token  ` . Execution can only be resumed from a previously yielded `  resume_token  ` . For the above sequence of `  PartialResultSet  ` s, resuming the query with `  "resume_token": "Af65..."  ` yields results from the `  PartialResultSet  ` with value "orl".

`  chunked_value  `

`  bool  `

If true, then the final value in `  values  ` is chunked, and must be combined with more values from subsequent `  PartialResultSet  ` s to obtain a complete field value.

`  resume_token  `

`  bytes  `

Streaming calls might be interrupted for a variety of reasons, such as TCP connection loss. If this occurs, the stream of results can be resumed by re-sending the original request and including `  resume_token  ` . Note that executing any other transaction in the same session invalidates the token.

`  stats  `

`  ResultSetStats  `

Query plan and execution statistics for the statement that produced this streaming result set. These can be requested by setting `  ExecuteSqlRequest.query_mode  ` and are sent only once with the last response in the stream. This field is also present in the last response for DML statements.

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

Optional. A precommit token is included if the read-write transaction has multiplexed sessions enabled. Pass the precommit token with the highest sequence number from this transaction attempt to the `  Commit  ` request for this transaction.

`  last  `

`  bool  `

Optional. Indicates whether this is the last `  PartialResultSet  ` in the stream. The server might optionally set this field. Clients shouldn't rely on this field being set in all cases.

## Partition

Information returned for each partition returned in a PartitionResponse.

Fields

`  partition_token  `

`  bytes  `

This token can be passed to `  Read  ` , `  StreamingRead  ` , `  ExecuteSql  ` , or `  ExecuteStreamingSql  ` requests to restrict the results to those identified by this partition token.

## PartitionOptions

Options for a `  PartitionQueryRequest  ` and `  PartitionReadRequest  ` .

Fields

`  partition_size_bytes  `

`  int64  `

**Note:** This hint is currently ignored by `  PartitionQuery  ` and `  PartitionRead  ` requests.

The desired data size for each partition generated. The default for this option is currently 1 GiB. This is only a hint. The actual size of each partition can be smaller or larger than this size request.

`  max_partitions  `

`  int64  `

**Note:** This hint is currently ignored by `  PartitionQuery  ` and `  PartitionRead  ` requests.

The desired maximum number of partitions to return. For example, this might be set to the number of workers available. The default for this option is currently 10,000. The maximum value is currently 200,000. This is only a hint. The actual number of partitions returned can be smaller or larger than this maximum count request.

## PartitionQueryRequest

The request for `  PartitionQuery  `

Fields

`  session  `

`  string  `

Required. The session used to create the partitions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.partitionQuery  `

`  transaction  `

`  TransactionSelector  `

Read-only snapshot transactions are supported, read and write and single-use transactions are not.

`  sql  `

`  string  `

Required. The query request to generate partitions for. The request fails if the query isn't root partitionable. For a query to be root partitionable, it needs to satisfy a few conditions. For example, if the query execution plan contains a distributed union operator, then it must be the first operator in the plan. For more information about other conditions, see [Read data in parallel](https://cloud.google.com/spanner/docs/reads#read_data_in_parallel) .

The query request must not contain DML commands, such as `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` . Use `  ExecuteStreamingSql  ` with a `  PartitionedDml  ` transaction for large, partition-friendly DML operations.

`  params  `

`  Struct  `

Parameter names and values that bind to placeholders in the SQL string.

A parameter placeholder consists of the `  @  ` character followed by the parameter name (for example, `  @firstName  ` ). Parameter names can contain letters, numbers, and underscores.

Parameters can appear anywhere that a literal value is expected. The same parameter name can be used more than once, for example:

`  "WHERE id > @msg_id AND id < @msg_id + 100"  `

It's an error to execute a SQL statement with unbound parameters.

`  param_types  `

`  map<string, Type  ` \>

It isn't always possible for Cloud Spanner to infer the right SQL type from a JSON value. For example, values of type `  BYTES  ` and values of type `  STRING  ` both appear in `  params  ` as JSON strings.

In these cases, `  param_types  ` can be used to specify the exact SQL type for some or all of the SQL query parameters. See the definition of `  Type  ` for more information about SQL types.

`  partition_options  `

`  PartitionOptions  `

Additional options that affect how many partitions are created.

## PartitionReadRequest

The request for `  PartitionRead  `

Fields

`  session  `

`  string  `

Required. The session used to create the partitions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.partitionRead  `

`  transaction  `

`  TransactionSelector  `

Read only snapshot transactions are supported, read/write and single use transactions are not.

`  table  `

`  string  `

Required. The name of the table in the database to be read.

`  index  `

`  string  `

If non-empty, the name of an index on `  table  ` . This index is used instead of the table primary key when interpreting `  key_set  ` and sorting result rows. See `  key_set  ` for further information.

`  columns[]  `

`  string  `

The columns of `  table  ` to be returned for each row matching this request.

`  key_set  `

`  KeySet  `

Required. `  key_set  ` identifies the rows to be yielded. `  key_set  ` names the primary keys of the rows in `  table  ` to be yielded, unless `  index  ` is present. If `  index  ` is present, then `  key_set  ` instead names index keys in `  index  ` .

It isn't an error for the `  key_set  ` to name rows that don't exist in the database. Read yields nothing for nonexistent rows.

`  partition_options  `

`  PartitionOptions  `

Additional options that affect how many partitions are created.

## PartitionResponse

The response for `  PartitionQuery  ` or `  PartitionRead  `

Fields

`  partitions[]  `

`  Partition  `

Partitions created by this request.

`  transaction  `

`  Transaction  `

Transaction created by this request.

## PlanNode

Node information for nodes appearing in a `  QueryPlan.plan_nodes  ` .

Fields

`  index  `

`  int32  `

The `  PlanNode  ` 's index in `  node list  ` .

`  kind  `

`  Kind  `

Used to determine the type of node. May be needed for visualizing different kinds of nodes differently. For example, If the node is a `  SCALAR  ` node, it will have a condensed representation which can be used to directly embed a description of the node in its parent.

`  display_name  `

`  string  `

The display name for the node.

`  child_links[]  `

`  ChildLink  `

List of child node `  index  ` es and their relationship to this parent.

`  short_representation  `

`  ShortRepresentation  `

Condensed representation for `  SCALAR  ` nodes.

`  metadata  `

`  Struct  `

Attributes relevant to the node contained in a group of key-value pairs. For example, a Parameter Reference node could have the following information in its metadata:

``` text
{
  "parameter_reference": "param1",
  "parameter_type": "array"
}
```

`  execution_stats  `

`  Struct  `

The execution statistics associated with the node, contained in a group of key-value pairs. Only present if the plan was returned as a result of a profile query. For example, number of executions, number of rows/time per execution etc.

## ChildLink

Metadata associated with a parent-child relationship appearing in a `  PlanNode  ` .

Fields

`  child_index  `

`  int32  `

The node to which the link points.

`  type  `

`  string  `

The type of the link. For example, in Hash Joins this could be used to distinguish between the build child and the probe child, or in the case of the child being an output variable, to represent the tag associated with the output variable.

`  variable  `

`  string  `

Only present if the child node is `  SCALAR  ` and corresponds to an output variable of the parent node. The field carries the name of the output variable. For example, a `  TableScan  ` operator that reads rows from a table will have child links to the `  SCALAR  ` nodes representing the output variables created for each column that is read by the operator. The corresponding `  variable  ` fields will be set to the variable names assigned to the columns.

## Kind

The kind of `  PlanNode  ` . Distinguishes between the two different kinds of nodes that can appear in a query plan.

Enums

`  KIND_UNSPECIFIED  `

Not specified.

`  RELATIONAL  `

Denotes a Relational operator node in the expression tree. Relational operators represent iterative processing of rows during query execution. For example, a `  TableScan  ` operation that reads rows from a table.

`  SCALAR  `

Denotes a Scalar node in the expression tree. Scalar nodes represent non-iterable entities in the query plan. For example, constants or arithmetic operators appearing inside predicate expressions or references to column names.

## ShortRepresentation

Condensed representation of a node and its subtree. Only present for `  SCALAR  ` `  PlanNode(s)  ` .

Fields

`  description  `

`  string  `

A string representation of the expression subtree rooted at this node.

`  subqueries  `

`  map<string, int32>  `

A mapping of (subquery variable name) -\> (subquery node id) for cases where the `  description  ` string of this node references a `  SCALAR  ` subquery contained in the expression subtree rooted at this node. The referenced `  SCALAR  ` subquery may not necessarily be a direct child of this node.

## QueryAdvisorResult

Output of query advisor analysis.

Fields

`  index_advice[]  `

`  IndexAdvice  `

Optional. Index Recommendation for a query. This is an optional field and the recommendation will only be available when the recommendation guarantees significant improvement in query performance.

## IndexAdvice

Recommendation to add new indexes to run queries more efficiently.

Fields

`  ddl[]  `

`  string  `

Optional. DDL statements to add new indexes that will improve the query.

`  improvement_factor  `

`  double  `

Optional. Estimated latency improvement factor. For example if the query currently takes 500 ms to run and the estimated latency with new indexes is 100 ms this field will be 5.

## QueryPlan

Contains an ordered list of nodes appearing in the query plan.

Fields

`  plan_nodes[]  `

`  PlanNode  `

The nodes in the query plan. Plan nodes are returned in pre-order starting with the plan root. Each `  PlanNode  ` 's `  id  ` corresponds to its index in `  plan_nodes  ` .

`  query_advice  `

`  QueryAdvisorResult  `

Optional. The advise/recommendations for a query. Currently this field will be serving index recommendations for a query.

## ReadRequest

The request for `  Read  ` and `  StreamingRead  ` .

Fields

`  session  `

`  string  `

Required. The session in which the read should be performed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.read  `

`  transaction  `

`  TransactionSelector  `

The transaction to use. If none is provided, the default is a temporary read-only transaction with strong concurrency.

`  table  `

`  string  `

Required. The name of the table in the database to be read.

`  index  `

`  string  `

If non-empty, the name of an index on `  table  ` . This index is used instead of the table primary key when interpreting `  key_set  ` and sorting result rows. See `  key_set  ` for further information.

`  columns[]  `

`  string  `

Required. The columns of `  table  ` to be returned for each row matching this request.

`  key_set  `

`  KeySet  `

Required. `  key_set  ` identifies the rows to be yielded. `  key_set  ` names the primary keys of the rows in `  table  ` to be yielded, unless `  index  ` is present. If `  index  ` is present, then `  key_set  ` instead names index keys in `  index  ` .

If the `  partition_token  ` field is empty, rows are yielded in table primary key order (if `  index  ` is empty) or index key order (if `  index  ` is non-empty). If the `  partition_token  ` field isn't empty, rows are yielded in an unspecified order.

It isn't an error for the `  key_set  ` to name rows that don't exist in the database. Read yields nothing for nonexistent rows.

`  limit  `

`  int64  `

If greater than zero, only the first `  limit  ` rows are yielded. If `  limit  ` is zero, the default is no limit. A limit can't be specified if `  partition_token  ` is set.

`  resume_token  `

`  bytes  `

If this request is resuming a previously interrupted read, `  resume_token  ` should be copied from the last `  PartialResultSet  ` yielded before the interruption. Doing this enables the new read to resume where the last read left off. The rest of the request parameters must exactly match the request that yielded this token.

`  partition_token  `

`  bytes  `

If present, results are restricted to the specified partition previously created using `  PartitionRead  ` . There must be an exact match for the values of fields common to this message and the PartitionReadRequest message used to create this partition\_token.

`  request_options  `

`  RequestOptions  `

Common options for this request.

`  directed_read_options  `

`  DirectedReadOptions  `

Directed read options for this request.

`  data_boost_enabled  `

`  bool  `

If this is for a partitioned read and this field is set to `  true  ` , the request is executed with Spanner Data Boost independent compute resources.

If the field is set to `  true  ` but the request doesn't set `  partition_token  ` , the API returns an `  INVALID_ARGUMENT  ` error.

## RequestOptions

Common request options for various APIs.

Fields

`  priority  `

`  Priority  `

Priority for the request.

`  request_tag  `

`  string  `

A per-request tag which can be applied to queries or reads, used for statistics collection. Both `  request_tag  ` and `  transaction_tag  ` can be specified for a read or query that belongs to a transaction. This field is ignored for requests where it's not applicable (for example, `  CommitRequest  ` ). Legal characters for `  request_tag  ` values are all printable characters (ASCII 32 - 126) and the length of a request\_tag is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string.

`  transaction_tag  `

`  string  `

A tag used for statistics collection about this transaction. Both `  request_tag  ` and `  transaction_tag  ` can be specified for a read or query that belongs to a transaction. The value of transaction\_tag should be the same for all requests belonging to the same transaction. If this request doesn't belong to any transaction, `  transaction_tag  ` is ignored. Legal characters for `  transaction_tag  ` values are all printable characters (ASCII 32 - 126) and the length of a `  transaction_tag  ` is limited to 50 characters. Values that exceed this limit are truncated. Any leading underscore (\_) characters are removed from the string.

## Priority

The relative priority for requests. Note that priority isn't applicable for `  BeginTransaction  ` .

The priority acts as a hint to the Cloud Spanner scheduler and doesn't guarantee priority or order of execution. For example:

  - Some parts of a write operation always execute at `  PRIORITY_HIGH  ` , regardless of the specified priority. This can cause you to see an increase in high priority workload even when executing a low priority request. This can also potentially cause a priority inversion where a lower priority request is fulfilled ahead of a higher priority request.
  - If a transaction contains multiple operations with different priorities, Cloud Spanner doesn't guarantee to process the higher priority operations first. There might be other constraints to satisfy, such as the order of operations.

Enums

`  PRIORITY_UNSPECIFIED  `

`  PRIORITY_UNSPECIFIED  ` is equivalent to `  PRIORITY_HIGH  ` .

`  PRIORITY_LOW  `

This specifies that the request is low priority.

`  PRIORITY_MEDIUM  `

This specifies that the request is medium priority.

`  PRIORITY_HIGH  `

This specifies that the request is high priority.

## ResultSet

Results from `  Read  ` or `  ExecuteSql  ` .

Fields

`  metadata  `

`  ResultSetMetadata  `

Metadata about the result set, such as row type information.

`  rows[]  `

`  ListValue  `

Each element in `  rows  ` is a row whose format is defined by `  metadata.row_type  ` . The ith element in each row matches the ith field in `  metadata.row_type  ` . Elements are encoded based on type as described `  here  ` .

`  stats  `

`  ResultSetStats  `

Query plan and execution statistics for the SQL statement that produced this result set. These can be requested by setting `  ExecuteSqlRequest.query_mode  ` . DML statements always produce stats containing the number of rows modified, unless executed using the `  ExecuteSqlRequest.QueryMode.PLAN  ` `  ExecuteSqlRequest.query_mode  ` . Other fields might or might not be populated, based on the `  ExecuteSqlRequest.query_mode  ` .

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

Optional. A precommit token is included if the read-write transaction is on a multiplexed session. Pass the precommit token with the highest sequence number from this transaction attempt to the `  Commit  ` request for this transaction.

## ResultSetMetadata

Metadata about a `  ResultSet  ` or `  PartialResultSet  ` .

Fields

`  row_type  `

`  StructType  `

Indicates the field names and types for the rows in the result set. For example, a SQL query like `  "SELECT UserId, UserName FROM Users"  ` could return a `  row_type  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```

`  transaction  `

`  Transaction  `

If the read or SQL query began a transaction as a side-effect, the information about the new transaction is yielded here.

`  undeclared_parameters  `

`  StructType  `

A SQL query can be parameterized. In PLAN mode, these parameters can be undeclared. This indicates the field names and types for those undeclared parameters in the SQL query. For example, a SQL query like `  "SELECT * FROM Users where UserId = @userId and UserName = @userName "  ` could return a `  undeclared_parameters  ` value like:

``` text
"fields": [
  { "name": "UserId", "type": { "code": "INT64" } },
  { "name": "UserName", "type": { "code": "STRING" } },
]
```

## ResultSetStats

Additional statistics about a `  ResultSet  ` or `  PartialResultSet  ` .

Fields

`  query_plan  `

`  QueryPlan  `

`  QueryPlan  ` for the query associated with this result.

`  query_stats  `

`  Struct  `

Aggregated statistics from the execution of the query. Only present when the query is profiled. For example, a query could return the statistics as follows:

``` text
{
  "rows_returned": "3",
  "elapsed_time": "1.22 secs",
  "cpu_time": "1.19 secs"
}
```

Union field `  row_count  ` . The number of rows modified by the DML statement. `  row_count  ` can be only one of the following:

`  row_count_exact  `

`  int64  `

Standard DML returns an exact count of rows that were modified.

`  row_count_lower_bound  `

`  int64  `

Partitioned DML doesn't offer exactly-once semantics, so it returns a lower bound of the rows modified.

## RollbackRequest

The request for `  Rollback  ` .

Fields

`  session  `

`  string  `

Required. The session in which the transaction to roll back is running.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.beginOrRollbackReadWriteTransaction  `

`  transaction_id  `

`  bytes  `

Required. The transaction to roll back.

## Session

A session in the Cloud Spanner API.

Fields

`  name  `

`  string  `

Output only. The name of the session. This is always system-assigned.

`  labels  `

`  map<string, string>  `

The labels for the session.

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z]([-a-z0-9]*[a-z0-9])?  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  ([a-z]([-a-z0-9]*[a-z0-9])?)?  ` .
  - No more than 64 labels can be associated with a given session.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

`  create_time  `

`  Timestamp  `

Output only. The timestamp when the session is created.

`  approximate_last_use_time  `

`  Timestamp  `

Output only. The approximate timestamp when the session is last used. It's typically earlier than the actual last use time.

`  creator_role  `

`  string  `

The database role which created this session.

`  multiplexed  `

`  bool  `

Optional. If `  true  ` , specifies a multiplexed session. Use a multiplexed session for multiple, concurrent read-only operations. Don't use them for read-write transactions, partitioned reads, or partitioned queries. Use `  sessions.create  ` to create multiplexed sessions. Don't use `  BatchCreateSessions  ` to create a multiplexed session. You can't delete or list multiplexed sessions.

## StructType

`  StructType  ` defines the fields of a `  STRUCT  ` type.

Fields

`  fields[]  `

`  Field  `

The list of fields that make up this struct. Order is significant, because values of this struct type are represented as lists, where the order of field values matches the order of fields in the `  StructType  ` . In turn, the order of fields matches the order of columns in a read request, or the order of fields in the `  SELECT  ` clause of a query.

## Field

Message representing a single field of a struct.

Fields

`  name  `

`  string  `

The name of the field. For reads, this is the column name. For SQL queries, it is the column alias (e.g., `  "Word"  ` in the query `  "SELECT 'hello' AS Word"  ` ), or the column name (e.g., `  "ColName"  ` in the query `  "SELECT ColName FROM Table"  ` ). Some columns might have an empty name (e.g., `  "SELECT UPPER(ColName)"  ` ). Note that a query result can contain multiple fields with the same name.

`  type  `

`  Type  `

The type of the field.

## Transaction

A transaction.

Fields

`  id  `

`  bytes  `

`  id  ` may be used to identify the transaction in subsequent `  Read  ` , `  ExecuteSql  ` , `  Commit  ` , or `  Rollback  ` calls.

Single-use read-only transactions do not have IDs, because single-use transactions do not support multiple requests.

`  read_timestamp  `

`  Timestamp  `

For snapshot read-only transactions, the read timestamp chosen for the transaction. Not returned by default: see `  TransactionOptions.ReadOnly.return_read_timestamp  ` .

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

`  precommit_token  `

`  MultiplexedSessionPrecommitToken  `

A precommit token is included in the response of a BeginTransaction request if the read-write transaction is on a multiplexed session and a mutation\_key was specified in the `  BeginTransaction  ` . The precommit token with the highest sequence number from this transaction attempt should be passed to the `  Commit  ` request for this transaction.

## TransactionOptions

Options to use for transactions.

Fields

`  exclude_txn_from_change_streams  `

`  bool  `

When `  exclude_txn_from_change_streams  ` is set to `  true  ` , it prevents read or write transactions from being tracked in change streams.

  - If the DDL option `  allow_txn_exclusion  ` is set to `  true  ` , then the updates made within this transaction aren't recorded in the change stream.

  - If you don't set the DDL option `  allow_txn_exclusion  ` or if it's set to `  false  ` , then the updates made within this transaction are recorded in the change stream.

When `  exclude_txn_from_change_streams  ` is set to `  false  ` or not set, modifications from this transaction are recorded in all change streams that are tracking columns modified by these transactions.

The `  exclude_txn_from_change_streams  ` option can only be specified for read-write or partitioned DML transactions, otherwise the API returns an `  INVALID_ARGUMENT  ` error.

`  isolation_level  `

`  IsolationLevel  `

Isolation level for the transaction.

Union field `  mode  ` . Required. The type of transaction. `  mode  ` can be only one of the following:

`  read_write  `

`  ReadWrite  `

Transaction may write.

Authorization to begin a read-write transaction requires `  spanner.databases.beginOrRollbackReadWriteTransaction  ` permission on the `  session  ` resource.

`  partitioned_dml  `

`  PartitionedDml  `

Partitioned DML transaction.

Authorization to begin a Partitioned DML transaction requires `  spanner.databases.beginPartitionedDmlTransaction  ` permission on the `  session  ` resource.

`  read_only  `

`  ReadOnly  `

Transaction does not write.

Authorization to begin a read-only transaction requires `  spanner.databases.beginReadOnlyTransaction  ` permission on the `  session  ` resource.

## IsolationLevel

`  IsolationLevel  ` is used when setting the [isolation level](/spanner/docs/isolation-levels) for a transaction.

Enums

`  ISOLATION_LEVEL_UNSPECIFIED  `

Default value.

If the value is not specified, the `  SERIALIZABLE  ` isolation level is used.

`  SERIALIZABLE  `

All transactions appear as if they executed in a serial order, even if some of the reads, writes, and other operations of distinct transactions actually occurred in parallel. Spanner assigns commit timestamps that reflect the order of committed transactions to implement this property. Spanner offers a stronger guarantee than serializability called external consistency. For more information, see [TrueTime and external consistency](https://cloud.google.com/spanner/docs/true-time-external-consistency#serializability) .

`  REPEATABLE_READ  `

All reads performed during the transaction observe a consistent snapshot of the database, and the transaction is only successfully committed in the absence of conflicts between its updates and any concurrent updates that have occurred since that snapshot. Consequently, in contrast to `  SERIALIZABLE  ` transactions, only write-write conflicts are detected in snapshot transactions.

This isolation level does not support read-only and partitioned DML transactions.

When `  REPEATABLE_READ  ` is specified on a read-write transaction, the locking semantics default to `  OPTIMISTIC  ` .

## PartitionedDml

This type has no fields.

Message type to initiate a Partitioned DML transaction.

## ReadOnly

Message type to initiate a read-only transaction.

Fields

`  return_read_timestamp  `

`  bool  `

If true, the Cloud Spanner-selected read timestamp is included in the `  Transaction  ` message that describes the transaction.

Union field `  timestamp_bound  ` . How to choose the timestamp for the read-only transaction. `  timestamp_bound  ` can be only one of the following:

`  strong  `

`  bool  `

Read at a timestamp where all previously committed transactions are visible.

`  min_read_timestamp  `

`  Timestamp  `

Executes all reads at a timestamp \>= `  min_read_timestamp  ` .

This is useful for requesting fresher data than some previous read, or data that is fresh enough to observe the effects of some previously committed transaction whose timestamp is known.

Note that this option can only be used in single-use transactions.

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

`  max_staleness  `

`  Duration  `

Read data at a timestamp \>= `  NOW - max_staleness  ` seconds. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps.

Useful for reading the freshest data available at a nearby replica, while bounding the possible staleness if the local replica has fallen behind.

Note that this option can only be used in single-use transactions.

`  read_timestamp  `

`  Timestamp  `

Executes all reads at the given timestamp. Unlike other modes, reads at a specific timestamp are repeatable; the same read at the same timestamp always returns the same data. If the timestamp is in the future, the read is blocked until the specified timestamp, modulo the read's deadline.

Useful for large scale consistent reads such as mapreduces, or for coordinating many reads against a consistent snapshot of the data.

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

`  exact_staleness  `

`  Duration  `

Executes all reads at a timestamp that is `  exact_staleness  ` old. The timestamp is chosen soon after the read is started.

Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps.

Useful for reading at nearby replicas without the distributed timestamp negotiation overhead of `  max_staleness  ` .

## ReadWrite

Message type to initiate a read-write transaction. Currently this transaction type has no options.

Fields

`  multiplexed_session_previous_transaction_id  `

`  bytes  `

Optional. Clients should pass the transaction ID of the previous transaction attempt that was aborted if this transaction is being executed on a multiplexed session.

## TransactionSelector

This message is used to select the transaction in which a `  Read  ` or `  ExecuteSql  ` call runs.

See `  TransactionOptions  ` for more information about transactions.

Fields

Union field `  selector  ` . If no fields are set, the default is a single use transaction with strong concurrency. `  selector  ` can be only one of the following:

`  single_use  `

`  TransactionOptions  `

Execute the read or SQL query in a temporary transaction. This is the most efficient way to execute a transaction that consists of a single SQL query.

`  id  `

`  bytes  `

Execute the read or SQL query in a previously-started transaction.

`  begin  `

`  TransactionOptions  `

Begin a new transaction and execute this read or SQL query in it. The transaction ID of the new transaction is returned in `  ResultSetMetadata.transaction  ` , which is a `  Transaction  ` .

## Type

`  Type  ` indicates the type of a Cloud Spanner value, as might be stored in a table cell or returned from an SQL query.

Fields

`  code  `

`  TypeCode  `

Required. The `  TypeCode  ` for this type.

`  array_element_type  `

`  Type  `

If `  code  ` == `  ARRAY  ` , then `  array_element_type  ` is the type of the array elements.

`  struct_type  `

`  StructType  `

If `  code  ` == `  STRUCT  ` , then `  struct_type  ` provides type information for the struct's fields.

`  type_annotation  `

`  TypeAnnotationCode  `

The `  TypeAnnotationCode  ` that disambiguates SQL type that Spanner will use to represent values of this type during query processing. This is necessary for some type codes because a single `  TypeCode  ` can be mapped to different SQL types depending on the SQL dialect. `  type_annotation  ` typically is not needed to process the content of a value (it doesn't affect serialization) and clients can ignore it on the read path.

`  proto_type_fqn  `

`  string  `

If `  code  ` == `  PROTO  ` or `  code  ` == `  ENUM  ` , then `  proto_type_fqn  ` is the fully qualified name of the proto type representing the proto/enum definition.

## TypeAnnotationCode

`  TypeAnnotationCode  ` is used as a part of `  Type  ` to disambiguate SQL types that should be used for a given Cloud Spanner value. Disambiguation is needed because the same Cloud Spanner type can be mapped to different SQL types depending on SQL dialect. TypeAnnotationCode doesn't affect the way value is serialized.

Enums

`  TYPE_ANNOTATION_CODE_UNSPECIFIED  `

Not specified.

`  PG_NUMERIC  `

PostgreSQL compatible NUMERIC type. This annotation needs to be applied to `  Type  ` instances having `  NUMERIC  ` type code to specify that values of this type should be treated as PostgreSQL NUMERIC values. Currently this annotation is always needed for `  NUMERIC  ` when a client interacts with PostgreSQL-enabled Spanner databases.

`  PG_JSONB  `

PostgreSQL compatible JSONB type. This annotation needs to be applied to `  Type  ` instances having `  JSON  ` type code to specify that values of this type should be treated as PostgreSQL JSONB values. Currently this annotation is always needed for `  JSON  ` when a client interacts with PostgreSQL-enabled Spanner databases.

`  PG_OID  `

PostgreSQL compatible OID type. This annotation can be used by a client interacting with PostgreSQL-enabled Spanner database to specify that a value should be treated using the semantics of the OID type.

## TypeCode

`  TypeCode  ` is used as part of `  Type  ` to indicate the type of a Cloud Spanner value.

Each legal value of a type can be encoded to or decoded from a JSON value, using the encodings described below. All Cloud Spanner values can be `  null  ` , regardless of type; `  null  ` s are always encoded as a JSON `  null  ` .

Enums

`  TYPE_CODE_UNSPECIFIED  `

Not specified.

`  BOOL  `

Encoded as JSON `  true  ` or `  false  ` .

`  INT64  `

Encoded as `  string  ` , in decimal format.

`  FLOAT64  `

Encoded as `  number  ` , or the strings `  "NaN"  ` , `  "Infinity"  ` , or `  "-Infinity"  ` .

`  FLOAT32  `

Encoded as `  number  ` , or the strings `  "NaN"  ` , `  "Infinity"  ` , or `  "-Infinity"  ` .

`  TIMESTAMP  `

Encoded as `  string  ` in RFC 3339 timestamp format. The time zone must be present, and must be `  "Z"  ` .

If the schema has the column option `  allow_commit_timestamp=true  ` , the placeholder string `  "spanner.commit_timestamp()"  ` can be used to instruct the system to insert the commit timestamp associated with the transaction commit.

`  DATE  `

Encoded as `  string  ` in RFC 3339 date format.

`  STRING  `

Encoded as `  string  ` .

`  BYTES  `

Encoded as a base64-encoded `  string  ` , as described in RFC 4648, section 4.

`  ARRAY  `

Encoded as `  list  ` , where the list elements are represented according to `  array_element_type  ` .

`  STRUCT  `

Encoded as `  list  ` , where list element `  i  ` is represented according to `  struct_type.fields[i]  ` .

`  NUMERIC  `

Encoded as `  string  ` , in decimal format or scientific notation format. Decimal format: `  [+-]Digits[.[Digits]]  ` or `  [+-][Digits].Digits  `

Scientific notation: `  [+-]Digits[.[Digits]][ExponentIndicator[+-]Digits]  ` or `  [+-][Digits].Digits[ExponentIndicator[+-]Digits]  ` (ExponentIndicator is `  "e"  ` or `  "E"  ` )

`  JSON  `

Encoded as a JSON-formatted `  string  ` as described in RFC 7159. The following rules are applied when parsing JSON input:

  - Whitespace characters are not preserved.
  - If a JSON object has duplicate keys, only the first key is preserved.
  - Members of a JSON object are not guaranteed to have their order preserved.
  - JSON array elements will have their order preserved.

`  PROTO  `

Encoded as a base64-encoded `  string  ` , as described in RFC 4648, section 4.

`  ENUM  `

Encoded as `  string  ` , in decimal format.
