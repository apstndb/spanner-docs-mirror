---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list
title: 'Method: projects.instances.databases.databaseRoles.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.ListDatabaseRolesResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.aspect)
- [DatabaseRole](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#DatabaseRole)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#DatabaseRole.SCHEMA_REPRESENTATION)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#try-it)

Lists Cloud Spanner database roles.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/databaseRoles`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The database whose roles should be listed. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/databases/&lt;database&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.databasesRoles.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters  |                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pageSize`  | `integer` Number of database roles to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                                                                                                                                                                                                   |
| `pageToken` | `string` If non-empty, `pageToken` should contain a [`nextPageToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.ListDatabaseRolesResponse.FIELDS.next_page_token) from a previous [`ListDatabaseRolesResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#body.ListDatabaseRolesResponse) . |

### Request body

The request body must be empty.

### Response body

The response for [`databaseRoles.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#google.spanner.admin.database.v1.DatabaseAdmin.ListDatabaseRoles) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "databaseRoles": [
    {
      object (DatabaseRole)
    }
  ],
  "nextPageToken": string
}
```

| Fields            |                                                                                                                                                                                                                                                                                                      |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `databaseRoles[]` | `object ( `[`DatabaseRole`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#DatabaseRole)` )` Database roles that matched the request.                                                                                                  |
| `nextPageToken`   | `string` `nextPageToken` can be sent in a subsequent [`databaseRoles.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.databaseRoles/list#google.spanner.admin.database.v1.DatabaseAdmin.ListDatabaseRoles) call to fetch more of the matching roles. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .

## DatabaseRole

A Cloud Spanner database role.

**JSON representation**

```
{
  "name": string
}
```

| Fields |                                                                                                                                                                                                                                 |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name` | `string` Required. The name of the database role. Values are of the form `projects/<project>/instances/<instance>/databases/<database>/databaseRoles/<role>` where `<role>` is as specified in the `CREATE ROLE` DDL statement. |
