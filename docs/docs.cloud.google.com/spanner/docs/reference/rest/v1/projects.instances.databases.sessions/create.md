  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a new session. A session can be used to perform transactions that read and/or modify data in a Cloud Spanner database. Sessions are meant to be reused for many consecutive transactions.

Sessions can only execute one transaction at a time. To execute multiple concurrent read-write/write-only transactions, create multiple sessions. Note that standalone reads and queries use a transaction internally, and count toward the one transaction limit.

Active sessions use additional server resources, so it's a good idea to delete idle and unneeded sessions. Aside from explicit deletes, Cloud Spanner can delete sessions when no operations are sent for more than an hour. If a session is deleted, requests to it return `  NOT_FOUND  ` .

Idle sessions can be kept alive by sending a trivial SQL query periodically, for example, `  "SELECT 1"  ` .

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/sessions  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  database  `

`  string  `

Required. The database in which the new session is created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  database  ` :

  - `  spanner.sessions.create  `

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
  &quot;session&quot;: {
    object (Session)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  session  `

`  object ( Session  ` )

Required. The session to create.

### Response body

If successful, the response body contains a newly created instance of `  Session  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.data  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
