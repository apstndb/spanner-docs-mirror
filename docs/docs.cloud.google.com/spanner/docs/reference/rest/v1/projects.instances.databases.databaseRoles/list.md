  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.PATH_PARAMETERS)
  - [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.QUERY_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.request_body)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.response_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.ListDatabaseRolesResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.aspect)
  - [DatabaseRole](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#DatabaseRole)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#DatabaseRole.SCHEMA_REPRESENTATION)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#try-it)

Lists Cloud Spanner database roles.

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

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/databaseRoles`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`parent`

`string`

Required. The database whose roles should be listed. Values are of the form `projects/<project>/instances/<instance>/databases/<database>` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `parent` :

  - `spanner.databasesRoles.list`

### Query parameters

Parameters

`pageSize`

`integer`

Number of database roles to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`pageToken`

`string`

If non-empty, `pageToken` should contain a `  nextPageToken  ` from a previous `  ListDatabaseRolesResponse  ` .

### Request body

The request body must be empty.

### Response body

The response for `  databaseRoles.list  ` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;databaseRoles&quot;: [{object (DatabaseRole)}],&quot;nextPageToken&quot;: string}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`databaseRoles[]`

` object ( DatabaseRole  ` )

Database roles that matched the request.

`nextPageToken`

`string`

`nextPageToken` can be sent in a subsequent `  databaseRoles.list  ` call to fetch more of the matching roles.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.admin`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## DatabaseRole

A Cloud Spanner database role.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;name&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`name`

`string`

Required. The name of the database role. Values are of the form `projects/<project>/instances/<instance>/databases/<database>/databaseRoles/<role>` where `<role>` is as specified in the `CREATE ROLE` DDL statement.
