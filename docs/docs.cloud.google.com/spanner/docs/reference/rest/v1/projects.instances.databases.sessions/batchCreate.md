  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.request_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.response_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.BatchCreateSessionsResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/batchCreate#try-it)

Creates multiple new sessions.

This API can be used to initialize a session cache on the clients. See <https://goo.gl/TgSFN2> for best practices on session cache management.

### HTTP request

Choose a location:

global

europe-west8

me-central2

us-central1

us-central2

us-east1

us-east4

us-east5

us-south1

us-west1

us-west2

us-west3

us-west4

us-west8

us-east7

  
`POST https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/sessions:batchCreate`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`database`

`string`

Required. The database in which the new sessions are created.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `database` :

  - `spanner.sessions.create`

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;sessionTemplate&quot;: {object (Session)},&quot;sessionCount&quot;: integer}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`sessionTemplate`

` object ( Session  ` )

Parameters to apply to each created session.

`sessionCount`

`integer`

Required. The number of sessions to be created in this batch call. At least one session is created. The API can return fewer than the requested number of sessions. If a specific number of sessions are desired, the client can make additional calls to `sessions.batchCreate` (adjusting `  sessionCount  ` as necessary).

### Response body

The response for `  sessions.batchCreate  ` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;session&quot;: [{object (Session)}]}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`session[]`

` object ( Session  ` )

The freshly created sessions.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.data`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
