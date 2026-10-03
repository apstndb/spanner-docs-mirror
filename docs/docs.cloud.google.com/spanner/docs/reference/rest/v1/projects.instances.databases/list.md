---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list
title: 'Method: projects.instances.databases.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.ListDatabasesResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#try-it)

Lists Cloud Spanner databases.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases`

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
<p>Required. The instance whose databases should be listed. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.databases.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters  |                                                                                                                                                                                                                                                                                                                                                                                                |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pageSize`  | `integer` Number of databases to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                                                                                                                                                                |
| `pageToken` | `string` If non-empty, `pageToken` should contain a [`nextPageToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.ListDatabasesResponse.FIELDS.next_page_token) from a previous [`ListDatabasesResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#body.ListDatabasesResponse) . |

### Request body

The request body must be empty.

### Response body

The response for [`databases.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#google.spanner.admin.database.v1.DatabaseAdmin.ListDatabases) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "databases": [
    {
      object (Database)
    }
  ],
  "nextPageToken": string
}
```

| Fields          |                                                                                                                                                                                                                                                                                    |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `databases[]`   | `object ( `[`Database`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#Database)` )` Databases that matched the request.                                                                                                                |
| `nextPageToken` | `string` `nextPageToken` can be sent in a subsequent [`databases.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list#google.spanner.admin.database.v1.DatabaseAdmin.ListDatabases) call to fetch more of the matching databases. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
