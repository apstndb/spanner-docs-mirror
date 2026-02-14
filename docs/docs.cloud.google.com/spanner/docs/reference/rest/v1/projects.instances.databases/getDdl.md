  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.GetDatabaseDdlResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Returns the schema of a Cloud Spanner database as a list of formatted DDL statements. This method does not show pending schema updates, those may be queried using the `  Operations  ` API.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/ddl  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  database  `

`  string  `

Required. The database whose schema we wish to get. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  `

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.databases.getDdl  `

### Request body

The request body must be empty.

### Response body

The response for `  databases.getDdl  ` .

If successful, the response body contains data with the following structure:

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
  &quot;statements&quot;: [
    string
  ],
  &quot;protoDescriptors&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  statements[]  `

`  string  `

A list of formatted DDL statements defining the schema of the database specified in the request.

`  protoDescriptors  `

`  string ( bytes format)  `

Proto descriptors stored in the database. Contains a protobuf-serialized [google.protobuf.FileDescriptorSet](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) . For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
