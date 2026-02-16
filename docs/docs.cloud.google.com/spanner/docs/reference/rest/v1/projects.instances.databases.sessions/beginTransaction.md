  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Begins a new transaction. This step can often be skipped: `  sessions.read  ` , `  sessions.executeSql  ` and `  sessions.commit  ` can begin a new transaction as a side-effect.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:beginTransaction  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the transaction runs.

Authorization requires one or more of the following [IAM](https://cloud.google.com/iam/docs/) permissions on the specified resource `  session  ` :

  - `  spanner.databases.beginReadOnlyTransaction  `
  - `  spanner.databases.beginOrRollbackReadWriteTransaction  `

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
  &quot;options&quot;: {
    object (TransactionOptions)
  },
  &quot;requestOptions&quot;: {
    object (RequestOptions)
  },
  &quot;mutationKey&quot;: {
    object (Mutation)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  options  `

`  object ( TransactionOptions  ` )

Required. Options for the new transaction.

`  requestOptions  `

`  object ( RequestOptions  ` )

Common options for this request. Priority is ignored for this request. Setting the priority in this `  requestOptions  ` struct doesn't do anything. To set the priority for a transaction, set it on the reads and writes that are part of this transaction instead.

`  mutationKey  `

`  object ( Mutation  ` )

Optional. Required for read-write transactions on a multiplexed session that commit mutations but don't perform any reads or queries. You must randomly select one of the mutations from the mutation set and send it as a part of this request.

### Response body

If successful, the response body contains an instance of `  Transaction  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
