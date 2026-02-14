  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a set of partition tokens that can be used to execute a read operation in parallel. Each of the returned partition tokens can be used by `  sessions.streamingRead  ` to specify a subset of the read result to read. The same session and read-only transaction must be used by the `  PartitionReadRequest  ` used to create the partition tokens and the `  ReadRequests  ` that use the partition tokens. There are no ordering guarantees on rows returned among the returned partition tokens, or even within each individual `  sessions.streamingRead  ` call issued with a `  partitionToken  ` .

Partition tokens become invalid when the session used to create them is deleted, is idle for too long, begins a new transaction, or becomes too old. When any of these happen, it isn't possible to resume the read, and the whole operation must be restarted from the beginning.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:partitionRead  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session used to create the partitions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.partitionRead  `

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
  &quot;table&quot;: string,
  &quot;index&quot;: string,
  &quot;columns&quot;: [
    string
  ],
  &quot;keySet&quot;: {
    object (KeySet)
  },
  &quot;partitionOptions&quot;: {
    object (PartitionOptions)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transaction  `

`  object ( TransactionSelector  ` )

sessions.read only snapshot transactions are supported, read/write and single use transactions are not.

`  table  `

`  string  `

Required. The name of the table in the database to be read.

`  index  `

`  string  `

If non-empty, the name of an index on `  table  ` . This index is used instead of the table primary key when interpreting `  keySet  ` and sorting result rows. See `  keySet  ` for further information.

`  columns[]  `

`  string  `

The columns of `  table  ` to be returned for each row matching this request.

`  keySet  `

`  object ( KeySet  ` )

Required. `  keySet  ` identifies the rows to be yielded. `  keySet  ` names the primary keys of the rows in `  table  ` to be yielded, unless `  index  ` is present. If `  index  ` is present, then `  keySet  ` instead names index keys in `  index  ` .

It isn't an error for the `  keySet  ` to name rows that don't exist in the database. sessions.read yields nothing for nonexistent rows.

`  partitionOptions  `

`  object ( PartitionOptions  ` )

Additional options that affect how many partitions are created.

### Response body

If successful, the response body contains an instance of `  PartitionResponse  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
