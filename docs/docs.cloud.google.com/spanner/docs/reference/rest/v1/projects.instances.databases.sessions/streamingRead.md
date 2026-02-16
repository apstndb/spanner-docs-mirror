  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Like `  sessions.read  ` , except returns the result set as a stream. Unlike `  sessions.read  ` , there is no limit on the size of the returned result set. However, no individual row in the result set can exceed 100 MiB, and no column value can exceed 10 MiB.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:streamingRead  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the read should be performed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

  - `  spanner.databases.read  `

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
  &quot;limit&quot;: string,
  &quot;resumeToken&quot;: string,
  &quot;partitionToken&quot;: string,
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;directedReadOptions&quot;: {
    object (DirectedReadOptions)
  },
  &quot;dataBoostEnabled&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transaction  `

`  object ( TransactionSelector  ` )

The transaction to use. If none is provided, the default is a temporary read-only transaction with strong concurrency.

`  table  `

`  string  `

Required. The name of the table in the database to be read.

`  index  `

`  string  `

If non-empty, the name of an index on `  table  ` . This index is used instead of the table primary key when interpreting `  keySet  ` and sorting result rows. See `  keySet  ` for further information.

`  columns[]  `

`  string  `

Required. The columns of `  table  ` to be returned for each row matching this request.

`  keySet  `

`  object ( KeySet  ` )

Required. `  keySet  ` identifies the rows to be yielded. `  keySet  ` names the primary keys of the rows in `  table  ` to be yielded, unless `  index  ` is present. If `  index  ` is present, then `  keySet  ` instead names index keys in `  index  ` .

If the `  partitionToken  ` field is empty, rows are yielded in table primary key order (if `  index  ` is empty) or index key order (if `  index  ` is non-empty). If the `  partitionToken  ` field isn't empty, rows are yielded in an unspecified order.

It isn't an error for the `  keySet  ` to name rows that don't exist in the database. sessions.read yields nothing for nonexistent rows.

`  limit  `

`  string ( int64 format)  `

If greater than zero, only the first `  limit  ` rows are yielded. If `  limit  ` is zero, the default is no limit. A limit can't be specified if `  partitionToken  ` is set.

`  resumeToken  `

`  string ( bytes format)  `

If this request is resuming a previously interrupted read, `  resumeToken  ` should be copied from the last `  PartialResultSet  ` yielded before the interruption. Doing this enables the new read to resume where the last read left off. The rest of the request parameters must exactly match the request that yielded this token.

A base64-encoded string.

`  partitionToken  `

`  string ( bytes format)  `

If present, results are restricted to the specified partition previously created using `  sessions.partitionRead  ` . There must be an exact match for the values of fields common to this message and the PartitionReadRequest message used to create this partitionToken.

A base64-encoded string.

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request.

`  directedReadOptions  `

`  object ( DirectedReadOptions  ` )

Directed read options for this request.

`  dataBoostEnabled  `

`  boolean  `

If this is for a partitioned read and this field is set to `  true  ` , the request is executed with Spanner Data Boost independent compute resources.

If the field is set to `  true  ` but the request doesn't set `  partitionToken  ` , the API returns an `  INVALID_ARGUMENT  ` error.

### Response body

If successful, the response body contains a stream of `  PartialResultSet  ` instances.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
