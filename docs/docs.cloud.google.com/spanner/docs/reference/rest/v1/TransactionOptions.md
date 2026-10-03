---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions
title: TransactionOptions
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#SCHEMA_REPRESENTATION)
- [ReadWrite](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadWrite)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadWrite.SCHEMA_REPRESENTATION)
- [ReadLockMode](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadLockMode)
- [PartitionedDml](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#PartitionedDml)
- [ReadOnly](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly.SCHEMA_REPRESENTATION)
- [IsolationLevel](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel)

Options to use for transactions.

**JSON representation**

```
{
  "excludeTxnFromChangeStreams": boolean,
  "isolationLevel": enum (IsolationLevel),

  // Union field mode can be only one of the following:
  "readWrite": {
    object (ReadWrite)
  },
  "partitionedDml": {
    object (PartitionedDml)
  },
  "readOnly": {
    object (ReadOnly)
  }
  // End of list of possible types for union field mode.
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
<td><code>excludeTxnFromChangeStreams</code></td>
<td><p><code>boolean</code></p>
<p>When <code>excludeTxnFromChangeStreams</code> is set to <code>true</code> , it prevents read or write transactions from being tracked in change streams.</p>
<ul>
<li><p>If the DDL option <code>allow_txn_exclusion</code> is set to <code>true</code> , then the updates made within this transaction aren't recorded in the change stream.</p></li>
<li><p>If you don't set the DDL option <code>allow_txn_exclusion</code> or if it's set to <code>false</code> , then the updates made within this transaction are recorded in the change stream.</p></li>
</ul>
<p>When <code>excludeTxnFromChangeStreams</code> is set to <code>false</code> or not set, modifications from this transaction are recorded in all change streams that are tracking columns modified by these transactions.</p>
<p>The <code>excludeTxnFromChangeStreams</code> option can only be specified for read-write or partitioned DML transactions, otherwise the API returns an <code>INVALID_ARGUMENT</code> error.</p></td>
</tr>
<tr class="even">
<td><code>isolationLevel</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel"><code>IsolationLevel</code></a><code> )</code></p>
<p>Isolation level for the transaction.</p></td>
</tr>
<tr class="odd">
<td>Union field <code>mode</code> . Required. The type of transaction. <code>mode</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>readWrite</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadWrite"><code>ReadWrite</code></a><code> )</code></p>
<p>Transaction may write.</p>
<p>Authorization to begin a read-write transaction requires <code>spanner.databases.beginOrRollbackReadWriteTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
<tr class="odd">
<td><code>partitionedDml</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#PartitionedDml"><code>PartitionedDml</code></a><code> )</code></p>
<p>Partitioned DML transaction.</p>
<p>Authorization to begin a Partitioned DML transaction requires <code>spanner.databases.beginPartitionedDmlTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
<tr class="even">
<td><code>readOnly</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadOnly"><code>ReadOnly</code></a><code> )</code></p>
<p>Transaction does not write.</p>
<p>Authorization to begin a read-only transaction requires <code>spanner.databases.beginReadOnlyTransaction</code> permission on the <code>session</code> resource.</p></td>
</tr>
</tbody>
</table>

## ReadWrite

Message type to initiate a read-write transaction. Currently this transaction type has no options.

**JSON representation**

```
{
  "readLockMode": enum (ReadLockMode),
  "multiplexedSessionPreviousTransactionId": string
}
```

| Fields                                    |                                                                                                                                                                                                                                                                                       |
|-------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `readLockMode`                            | `enum ( `[`ReadLockMode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#ReadLockMode)` )` The read lock mode for the transaction.                                                                                                                   |
| `multiplexedSessionPreviousTransactionId` | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Optional. Clients should pass the transaction ID of the previous transaction attempt that was aborted if this transaction is being executed on a multiplexed session. A base64-encoded string. |

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
<li>If isolation level is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.SERIALIZABLE"><code>SERIALIZABLE</code></a> , locking semantics default to <code>PESSIMISTIC</code> .</li>
<li>If isolation level is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> , locking semantics default to <code>OPTIMISTIC</code> .</li>
<li>See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for more details.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>PESSIMISTIC</code></td>
<td><p>Pessimistic lock mode.</p>
<p>Lock acquisition behavior depends on the isolation level in use. In <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.SERIALIZABLE"><code>SERIALIZABLE</code></a> isolation, reads and writes acquire necessary locks during transaction statement execution. In <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> isolation, reads that explicitly request to be locked and writes acquire locks. See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for details on the types of locks acquired at each transaction step.</p></td>
</tr>
<tr class="odd">
<td><code>OPTIMISTIC</code></td>
<td><p>Optimistic lock mode.</p>
<p>Lock acquisition behavior depends on the isolation level in use. In both <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.SERIALIZABLE"><code>SERIALIZABLE</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions#IsolationLevel.ENUM_VALUES.REPEATABLE_READ"><code>REPEATABLE_READ</code></a> isolation, reads and writes do not acquire locks during transaction statement execution. See <a href="https://cloud.google.com/spanner/docs/concurrency-control">Concurrency control</a> for details on how the guarantees of each isolation level are provided at commit time.</p></td>
</tr>
</tbody>
</table>

## PartitionedDml

This type has no fields.

Message type to initiate a Partitioned DML transaction.

## ReadOnly

Message type to initiate a read-only transaction.

**JSON representation**

```
{
  "returnReadTimestamp": boolean,

  // Union field timestamp_bound can be only one of the following:
  "strong": boolean,
  "minReadTimestamp": string,
  "maxStaleness": string,
  "readTimestamp": string,
  "exactStaleness": string
  // End of list of possible types for union field timestamp_bound.
}
```

| Fields                                                                                                                                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `returnReadTimestamp`                                                                                                                          | `boolean` If true, the Cloud Spanner-selected read timestamp is included in the [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction) message that describes the transaction.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| Union field `timestamp_bound` . How to choose the timestamp for the read-only transaction. `timestamp_bound` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `strong`                                                                                                                                       | `boolean` sessions.read at a timestamp where all previously committed transactions are visible.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `minReadTimestamp`                                                                                                                             | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Executes all reads at a timestamp \>= `minReadTimestamp` . This is useful for requesting fresher data than some previous read, or data that is fresh enough to observe the effects of some previously committed transaction whose timestamp is known. Note that this option can only be used in single-use transactions. A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `"2014-10-02T15:01:23.045123456Z"` . Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                               |
| `maxStaleness`                                                                                                                                 | `string ( `[`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration)` format)` sessions.read data at a timestamp \>= `NOW - maxStaleness` seconds. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps. Useful for reading the freshest data available at a nearby replica, while bounding the possible staleness if the local replica has fallen behind. Note that this option can only be used in single-use transactions. A duration in seconds with up to nine fractional digits, ending with ' `s` '. Example: `"3.5s"` .                                                                                                                                                   |
| `readTimestamp`                                                                                                                                | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Executes all reads at the given timestamp. Unlike other modes, reads at a specific timestamp are repeatable; the same read at the same timestamp always returns the same data. If the timestamp is in the future, the read is blocked until the specified timestamp, modulo the read's deadline. Useful for large scale consistent reads such as mapreduces, or for coordinating many reads against a consistent snapshot of the data. A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `"2014-10-02T15:01:23.045123456Z"` . Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `exactStaleness`                                                                                                                               | `string ( `[`Duration`](https://protobuf.dev/reference/protobuf/google.protobuf/#duration)` format)` Executes all reads at a timestamp that is `exactStaleness` old. The timestamp is chosen soon after the read is started. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps. Useful for reading at nearby replicas without the distributed timestamp negotiation overhead of `maxStaleness` . A duration in seconds with up to nine fractional digits, ending with ' `s` '. Example: `"3.5s"` .                                                                                                                                                                                                   |

## IsolationLevel

`IsolationLevel` is used when setting the [isolation level](https://cloud.google.com/spanner/docs/isolation-levels) for a transaction.

| Enums                         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
|-------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ISOLATION_LEVEL_UNSPECIFIED` | Default value. If the value is not specified, the `SERIALIZABLE` isolation level is used.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `SERIALIZABLE`                | All transactions appear as if they executed in a serial order, even if some of the reads, writes, and other operations of distinct transactions actually occurred in parallel. Spanner assigns commit timestamps that reflect the order of committed transactions to implement this property. Spanner offers a stronger guarantee than serializability called external consistency. For more information, see [TrueTime and external consistency](https://cloud.google.com/spanner/docs/true-time-external-consistency#serializability) .                                                      |
| `REPEATABLE_READ`             | All reads performed during the transaction observe a consistent snapshot of the database, and the transaction is only successfully committed in the absence of conflicts between its updates and any concurrent updates that have occurred since that snapshot. Consequently, in contrast to `SERIALIZABLE` transactions, only write-write conflicts are detected in snapshot transactions. This isolation level does not support read-only and partitioned DML transactions. When `REPEATABLE_READ` is specified on a read-write transaction, the locking semantics default to `OPTIMISTIC` . |
