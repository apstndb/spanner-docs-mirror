  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Query parameters](#body.QUERY_PARAMETERS)
  - [Request body](#body.request_body)
  - [Response body](#body.response_body)
      - [JSON representation](#body.ListDatabaseRolesResponse.SCHEMA_REPRESENTATION)
  - [Authorization scopes](#body.aspect)
  - [DatabaseRole](#DatabaseRole)
      - [JSON representation](#DatabaseRole.SCHEMA_REPRESENTATION)
  - [Try it\!](#try-it)

Lists Cloud Spanner database roles.

### HTTP request

Choose a location:

  
`  GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/databaseRoles  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The database whose roles should be listed. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.databasesRoles.list  `

### Query parameters

Parameters

`  pageSize  `

`  integer  `

Number of database roles to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.

`  pageToken  `

`  string  `

If non-empty, `  pageToken  ` should contain a `  nextPageToken  ` from a previous `  ListDatabaseRolesResponse  ` .

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
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;databaseRoles&quot;: [
    {
      object (DatabaseRole)
    }
  ],
  &quot;nextPageToken&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  databaseRoles[]  `

`  object ( DatabaseRole  ` )

Database roles that matched the request.

`  nextPageToken  `

`  string  `

`  nextPageToken  ` can be sent in a subsequent `  databaseRoles.list  ` call to fetch more of the matching roles.

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .

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
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;name&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Required. The name of the database role. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>/databaseRoles/<role>  ` where `  <role>  ` is as specified in the `  CREATE ROLE  ` DDL statement.
