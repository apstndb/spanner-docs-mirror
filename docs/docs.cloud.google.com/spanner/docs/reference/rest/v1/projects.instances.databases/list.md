  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.PATH_PARAMETERS)
  - [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.QUERY_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.response_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.ListDatabasesResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#try-it)

Lists Cloud Spanner databases.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`parent`

`string`

Required. The instance whose databases should be listed. Values are of the form `projects/<project>/instances/<instance>` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `parent` :

  - `spanner.databases.list`

### Query parameters

Parameters

`pageSize`

`integer`

Number of databases to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`pageToken`

`string`

If non-empty, `pageToken` should contain a `  nextPageToken  ` from a previous `  ListDatabasesResponse  ` .

### Request body

The request body must be empty.

### Response body

The response for `  databases.list  ` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;databases&quot;: [{object (Database)}],&quot;nextPageToken&quot;: string}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`databases[]`

` object ( Database  ` )

Databases that matched the request.

`nextPageToken`

`string`

`nextPageToken` can be sent in a subsequent `  databases.list  ` call to fetch more of the matching databases.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.admin`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
