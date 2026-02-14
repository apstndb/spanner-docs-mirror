  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Rolls back a transaction, releasing any locks it holds. It's a good idea to call this for any transaction that includes one or more `  sessions.read  ` or `  sessions.executeSql  ` requests and ultimately decides not to commit.

`  sessions.rollback  ` returns `  OK  ` if it successfully aborts the transaction, the transaction was already aborted, or the transaction isn't found. `  sessions.rollback  ` never returns `  ABORTED  ` .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{session=projects/*/instances/*/databases/*/sessions/*}:rollback  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  session  `

`  string  `

Required. The session in which the transaction to roll back is running.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  session  ` :

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
  &quot;transactionId&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  transactionId  `

`  string ( bytes format)  `

Required. The transaction to roll back.

A base64-encoded string.

### Response body

If successful, the response body is an empty JSON object.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
