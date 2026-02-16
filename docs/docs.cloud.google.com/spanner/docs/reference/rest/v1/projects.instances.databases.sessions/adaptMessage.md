  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
      - [JSON representation](#body.AdaptMessageResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Handles a single message from the client and returns the result as a stream. The server will interpret the message frame and respond with message frames to the client.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/sessions/*}:adaptMessage  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  name  `

`  string  `

Required. The database session in which the adapter request is processed.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  name  ` :

  - `  spanner.databases.adapt  `

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
  &quot;protocol&quot;: string,
  &quot;payload&quot;: string,
  &quot;attachments&quot;: {
    string: string,
    ...
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  protocol  `

`  string  `

Required. Identifier for the underlying wire protocol.

`  payload  `

`  string ( bytes format)  `

Optional. Uninterpreted bytes from the underlying wire protocol.

A base64-encoded string.

`  attachments  `

`  map (key: string, value: string)  `

Optional. Opaque request state passed by the client to the server.

### Response body

Message sent by the adapter to the client.

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
  &quot;payload&quot;: string,
  &quot;stateUpdates&quot;: {
    string: string,
    ...
  },
  &quot;last&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  payload  `

`  string ( bytes format)  `

Optional. Uninterpreted bytes from the underlying wire protocol.

A base64-encoded string.

`  stateUpdates  `

`  map (key: string, value: string)  `

Optional. Opaque state updates to be applied by the client.

`  last  `

`  boolean  `

Optional. Indicates whether this is the last `  AdaptMessageResponse  ` in the stream. This field may be optionally set by the server. Clients should not rely on this field being set in all cases.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
