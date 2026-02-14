  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

`  databases.changequorum  ` is strictly restricted to databases that use dual-region instance configurations.

Initiates a background operation to change the quorum of a database from dual-region mode to single-region mode or vice versa.

The returned long-running operation has a name of the format `  projects/<project>/instances/<instance>/databases/<database>/operations/<operationId>  ` and can be used to track execution of the `  databases.changequorum  ` . The metadata field type is `  ChangeQuorumMetadata  ` .

Authorization requires `  spanner.databases.changequorum  ` permission on the resource database.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*}:changequorum  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. Name of the database in which to apply `  databases.changequorum  ` . Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.databases.changequorum  `

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
  &quot;quorumType&quot;: {
    object (QuorumType)
  },
  &quot;etag&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  quorumType  `

`  object ( QuorumType  ` )

Required. The type of this quorum.

`  etag  `

`  string  `

Optional. The etag is the hash of the `  QuorumInfo  ` . The `  databases.changequorum  ` operation is only performed if the etag matches that of the `  QuorumInfo  ` in the current database resource. Otherwise the API returns an `  ABORTED  ` error.

The etag is used for optimistic concurrency control as a way to help prevent simultaneous change quorum requests that could create a race condition.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
