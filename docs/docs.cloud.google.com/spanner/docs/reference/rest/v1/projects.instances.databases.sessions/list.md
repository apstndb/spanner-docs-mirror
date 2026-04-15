  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.PATH_PARAMETERS)
  - [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.QUERY_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.response_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.ListSessionsResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#try-it)

Lists all sessions in a given database.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/sessions`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`database`

`string`

Required. The database in which to list sessions.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `database` :

  - `spanner.sessions.list`

### Query parameters

Parameters

`pageSize`

`integer`

Number of sessions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`pageToken`

`string`

If non-empty, `pageToken` should contain a `  nextPageToken  ` from a previous `  ListSessionsResponse  ` .

`filter`

`string`

An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:

  - `labels.key` where key is the name of a label

Some examples of using filters are:

  - `labels.env:*` --\> The session has the label "env".
  - `labels.env:dev` --\> The session has the label "env" and the value of the label contains the string "dev".

### Request body

The request body must be empty.

### Response body

The response for `  sessions.list  ` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;sessions&quot;: [{object (Session)}],&quot;nextPageToken&quot;: string}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`sessions[]`

` object ( Session  ` )

The list of requested sessions.

`nextPageToken`

`string`

`nextPageToken` can be sent in a subsequent `  sessions.list  ` call to fetch more of the matching sessions.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.data`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
