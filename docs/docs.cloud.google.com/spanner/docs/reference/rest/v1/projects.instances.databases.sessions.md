---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions
title: 'REST Resource: projects.instances.databases.sessions'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [Resource: Session](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions#Session)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions#Session.SCHEMA_REPRESENTATION)
- [Methods](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions#METHODS_SUMMARY)

## Resource: Session

A session in the Cloud Spanner API.

**JSON representation**

```
{
  "name": string,
  "labels": {
    string: string,
    ...
  },
  "createTime": string,
  "approximateLastUseTime": string,
  "creatorRole": string,
  "multiplexed": boolean
}
```

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
<td><p><code>map (key: string, value: string)</code></p>
<p>The labels for the session.</p>
<ul>
<li>Label keys must be between 1 and 63 characters long and must conform to the following regular expression: <code>[a-z]([-a-z0-9]*[a-z0-9])?</code> .</li>
<li>Label values must be between 0 and 63 characters long and must conform to the regular expression <code>([a-z]([-a-z0-9]*[a-z0-9])?)?</code> .</li>
<li>No more than 64 labels can be associated with a given session.</li>
</ul>
<p>See <a href="https://goo.gl/xmQnxf">https://goo.gl/xmQnxf</a> for more information on and examples of labels.</p></td>
</tr>
<tr class="odd">
<td><code>createTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The timestamp when the session is created.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="even">
<td><code>approximateLastUseTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The approximate timestamp when the session is last used. It's typically earlier than the actual last use time.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="odd">
<td><code>creatorRole</code></td>
<td><p><code>string</code></p>
<p>The database role which created this session.</p></td>
</tr>
<tr class="even">
<td><code>multiplexed</code></td>
<td><p><code>boolean</code></p>
<p>Optional. If <code>true</code> , specifies a multiplexed session. Use a multiplexed session for multiple, concurrent operations including any combination of read-only and read-write transactions. Use <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/create#google.spanner.v1.Spanner.CreateSession"><code>sessions.create</code></a> to create multiplexed sessions. Don't use <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#google.spanner.v1.Spanner.BatchCreateSessions"><code>sessions.batchCreate</code></a> to create a multiplexed session. You can't delete or list multiplexed sessions.</p></td>
</tr>
</tbody>
</table>

| Methods                                                                                                                                         |                                                                                                                                                                                                                                                                           |
|-------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`adaptMessage`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adaptMessage)               | Handles a single message from the client and returns the result as a stream.                                                                                                                                                                                              |
| [`adapter`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter)                         | Creates a new session to be used for requests made by the adapter.                                                                                                                                                                                                        |
| [`batchCreate`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate)                 | Creates multiple new sessions.                                                                                                                                                                                                                                            |
| [`batchWrite`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchWrite)                   | Batches the supplied mutation groups in a collection of efficient transactions.                                                                                                                                                                                           |
| [`beginTransaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/beginTransaction)       | Begins a new transaction.                                                                                                                                                                                                                                                 |
| [`commit`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit)                           | Commits a transaction.                                                                                                                                                                                                                                                    |
| [`create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/create)                           | Creates a new session.                                                                                                                                                                                                                                                    |
| [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/delete)                           | Ends a session, releasing server resources associated with it.                                                                                                                                                                                                            |
| [`executeBatchDml`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeBatchDml)         | Executes a batch of SQL DML statements.                                                                                                                                                                                                                                   |
| [`executeSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql)                   | Executes an SQL statement, returning all results in a single reply.                                                                                                                                                                                                       |
| [`executeStreamingSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeStreamingSql) | Like [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#google.spanner.v1.Spanner.ExecuteSql) , except returns the result set as a stream.                                                      |
| [`get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get)                                 | Gets a session.                                                                                                                                                                                                                                                           |
| [`list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list)                               | Lists all sessions in a given database.                                                                                                                                                                                                                                   |
| [`partitionQuery`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery)           | Creates a set of partition tokens that can be used to execute a query operation in parallel.                                                                                                                                                                              |
| [`partitionRead`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionRead)             | Creates a set of partition tokens that can be used to execute a read operation in parallel.                                                                                                                                                                               |
| [`read`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read)                               | Reads rows from the database using key lookups and scans, as a simple key/value style alternative to [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#google.spanner.v1.Spanner.ExecuteSql) . |
| [`rollback`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/rollback)                       | Rolls back a transaction, releasing any locks it holds.                                                                                                                                                                                                                   |
| [`streamingRead`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/streamingRead)             | Like [`Read`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#google.spanner.v1.Spanner.Read) , except returns the result set as a stream.                                                                        |
