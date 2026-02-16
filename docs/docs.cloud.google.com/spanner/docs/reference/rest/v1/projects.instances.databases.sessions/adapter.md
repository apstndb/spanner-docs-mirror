  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.Session.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a new session to be used for requests made by the adapter. A session identifies a specific incarnation of a database resource and is meant to be reused across many `  sessions.adaptMessage  ` calls.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/sessions:adapter  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The database in which the new session is created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.sessions.create  `

### Request body

The request body contains an instance of `  Session  ` .

### Response body

A session in the Cloud Spanner Adapter API.

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
  &quot;name&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Identifier. The name of the session. This is always system-assigned.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
