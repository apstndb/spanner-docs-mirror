  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [ReadWrite](#ReadWrite)
      - [JSON representation](#ReadWrite.SCHEMA_REPRESENTATION)
  - [PartitionedDml](#PartitionedDml)
  - [ReadOnly](#ReadOnly)
      - [JSON representation](#ReadOnly.SCHEMA_REPRESENTATION)
  - [IsolationLevel](#IsolationLevel)

Options to use for transactions.

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
  &quot;excludeTxnFromChangeStreams&quot;: boolean,
  &quot;isolationLevel&quot;: enum (IsolationLevel),

  // Union field mode can be only one of the following:
  &quot;readWrite&quot;: {
    object (ReadWrite)
  },
  &quot;partitionedDml&quot;: {
    object (PartitionedDml)
  },
  &quot;readOnly&quot;: {
    object (ReadOnly)
  }
  // End of list of possible types for union field mode.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  excludeTxnFromChangeStreams  `

`  boolean  `

When `  excludeTxnFromChangeStreams  ` is set to `  true  ` , it prevents read or write transactions from being tracked in change streams.

  - If the DDL option `  allow_txn_exclusion  ` is set to `  true  ` , then the updates made within this transaction aren't recorded in the change stream.

  - If you don't set the DDL option `  allow_txn_exclusion  ` or if it's set to `  false  ` , then the updates made within this transaction are recorded in the change stream.

When `  excludeTxnFromChangeStreams  ` is set to `  false  ` or not set, modifications from this transaction are recorded in all change streams that are tracking columns modified by these transactions.

The `  excludeTxnFromChangeStreams  ` option can only be specified for read-write or partitioned DML transactions, otherwise the API returns an `  INVALID_ARGUMENT  ` error.

`  isolationLevel  `

`  enum ( IsolationLevel  ` )

Isolation level for the transaction.

Union field `  mode  ` . Required. The type of transaction. `  mode  ` can be only one of the following:

`  readWrite  `

`  object ( ReadWrite  ` )

Transaction may write.

Authorization to begin a read-write transaction requires `  spanner.databases.beginOrRollbackReadWriteTransaction  ` permission on the `  session  ` resource.

`  partitionedDml  `

`  object ( PartitionedDml  ` )

Partitioned DML transaction.

Authorization to begin a Partitioned DML transaction requires `  spanner.databases.beginPartitionedDmlTransaction  ` permission on the `  session  ` resource.

`  readOnly  `

`  object ( ReadOnly  ` )

Transaction does not write.

Authorization to begin a read-only transaction requires `  spanner.databases.beginReadOnlyTransaction  ` permission on the `  session  ` resource.

## ReadWrite

Message type to initiate a read-write transaction. Currently this transaction type has no options.

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
  &quot;multiplexedSessionPreviousTransactionId&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  multiplexedSessionPreviousTransactionId  `

`  string ( bytes format)  `

Optional. Clients should pass the transaction ID of the previous transaction attempt that was aborted if this transaction is being executed on a multiplexed session.

A base64-encoded string.

## PartitionedDml

This type has no fields.

Message type to initiate a Partitioned DML transaction.

## ReadOnly

Message type to initiate a read-only transaction.

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
  &quot;returnReadTimestamp&quot;: boolean,

  // Union field timestamp_bound can be only one of the following:
  &quot;strong&quot;: boolean,
  &quot;minReadTimestamp&quot;: string,
  &quot;maxStaleness&quot;: string,
  &quot;readTimestamp&quot;: string,
  &quot;exactStaleness&quot;: string
  // End of list of possible types for union field timestamp_bound.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  returnReadTimestamp  `

`  boolean  `

If true, the Cloud Spanner-selected read timestamp is included in the `  Transaction  ` message that describes the transaction.

Union field `  timestamp_bound  ` . How to choose the timestamp for the read-only transaction. `  timestamp_bound  ` can be only one of the following:

`  strong  `

`  boolean  `

sessions.read at a timestamp where all previously committed transactions are visible.

`  minReadTimestamp  `

`  string ( Timestamp  ` format)

Executes all reads at a timestamp \>= `  minReadTimestamp  ` .

This is useful for requesting fresher data than some previous read, or data that is fresh enough to observe the effects of some previously committed transaction whose timestamp is known.

Note that this option can only be used in single-use transactions.

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  maxStaleness  `

`  string ( Duration  ` format)

sessions.read data at a timestamp \>= `  NOW - maxStaleness  ` seconds. Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps.

Useful for reading the freshest data available at a nearby replica, while bounding the possible staleness if the local replica has fallen behind.

Note that this option can only be used in single-use transactions.

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .

`  readTimestamp  `

`  string ( Timestamp  ` format)

Executes all reads at the given timestamp. Unlike other modes, reads at a specific timestamp are repeatable; the same read at the same timestamp always returns the same data. If the timestamp is in the future, the read is blocked until the specified timestamp, modulo the read's deadline.

Useful for large scale consistent reads such as mapreduces, or for coordinating many reads against a consistent snapshot of the data.

A timestamp in RFC3339 UTC "Zulu" format, accurate to nanoseconds. Example: `  "2014-10-02T15:01:23.045123456Z"  ` .

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  exactStaleness  `

`  string ( Duration  ` format)

Executes all reads at a timestamp that is `  exactStaleness  ` old. The timestamp is chosen soon after the read is started.

Guarantees that all writes that have committed more than the specified number of seconds ago are visible. Because Cloud Spanner chooses the exact timestamp, this mode works even if the client's local clock is substantially skewed from Cloud Spanner commit timestamps.

Useful for reading at nearby replicas without the distributed timestamp negotiation overhead of `  maxStaleness  ` .

A duration in seconds with up to nine fractional digits, ending with ' `  s  ` '. Example: `  "3.5s"  ` .

## IsolationLevel

`  IsolationLevel  ` is used when setting the [isolation level](https://cloud.google.com/spanner/docs/isolation-levels) for a transaction.

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
