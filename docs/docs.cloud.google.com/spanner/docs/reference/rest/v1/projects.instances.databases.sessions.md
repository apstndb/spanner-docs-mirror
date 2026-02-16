  - [Resource: Session](#Session)
      - [JSON representation](#Session.SCHEMA_REPRESENTATION)
  - [Methods](#METHODS_SUMMARY)

## Resource: Session

A session in the Cloud Spanner API.

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
  &quot;name&quot;: string,
  &quot;labels&quot;: {
    string: string,
    ...
  },
  &quot;createTime&quot;: string,
  &quot;approximateLastUseTime&quot;: string,
  &quot;creatorRole&quot;: string,
  &quot;multiplexed&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Output only. The name of the session. This is always system-assigned.

`  labels  `

`  map (key: string, value: string)  `

The labels for the session.

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z]([-a-z0-9]*[a-z0-9])?  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  ([a-z]([-a-z0-9]*[a-z0-9])?)?  ` .
  - No more than 64 labels can be associated with a given session.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

`  createTime  `

`  string ( Timestamp  ` format)

Output only. The timestamp when the session is created.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  approximateLastUseTime  `

`  string ( Timestamp  ` format)

Output only. The approximate timestamp when the session is last used. It's typically earlier than the actual last use time.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  creatorRole  `

`  string  `

The database role which created this session.

`  multiplexed  `

`  boolean  `

Optional. If `  true  ` , specifies a multiplexed session. Use a multiplexed session for multiple, concurrent operations including any combination of read-only and read-write transactions. Use `  sessions.create  ` to create multiplexed sessions. Don't use `  sessions.batchCreate  ` to create a multiplexed session. You can't delete or list multiplexed sessions.

## Methods

### `             adaptMessage           `

Handles a single message from the client and returns the result as a stream.

### `             adapter           `

Creates a new session to be used for requests made by the adapter.

### `             batchCreate           `

Creates multiple new sessions.

### `             batchWrite           `

Batches the supplied mutation groups in a collection of efficient transactions.

### `             beginTransaction           `

Begins a new transaction.

### `             commit           `

Commits a transaction.

### `             create           `

Creates a new session.

### `             delete           `

Ends a session, releasing server resources associated with it.

### `             executeBatchDml           `

Executes a batch of SQL DML statements.

### `             executeSql           `

Executes an SQL statement, returning all results in a single reply.

### `             executeStreamingSql           `

Like `  ExecuteSql  ` , except returns the result set as a stream.

### `             get           `

Gets a session.

### `             list           `

Lists all sessions in a given database.

### `             partitionQuery           `

Creates a set of partition tokens that can be used to execute a query operation in parallel.

### `             partitionRead           `

Creates a set of partition tokens that can be used to execute a read operation in parallel.

### `             read           `

Reads rows from the database using key lookups and scans, as a simple key/value style alternative to `  ExecuteSql  ` .

### `             rollback           `

Rolls back a transaction, releasing any locks it holds.

### `             streamingRead           `

Like `  Read  ` , except returns the result set as a stream.
