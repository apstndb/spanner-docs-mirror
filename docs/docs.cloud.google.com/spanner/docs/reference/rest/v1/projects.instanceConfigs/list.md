---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list
title: 'Method: projects.instanceConfigs.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.ListInstanceConfigsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#try-it)

Lists the supported instance configurations for a given project.

Returns both Google-managed configurations and user-managed configurations.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*}/instanceConfigs`

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
<p>Required. The name of the project for which a list of supported instance configurations is requested. Values are of the form <code>projects/&lt;project&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.instanceConfigs.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters  |                                                                                                                                                                                                                                                                                                                                                                                                          |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pageSize`  | `integer` Number of instance configurations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                                                                                                                                                            |
| `pageToken` | `string` If non-empty, `pageToken` should contain a [`nextPageToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.ListInstanceConfigsResponse.FIELDS.next_page_token) from a previous [`ListInstanceConfigsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#body.ListInstanceConfigsResponse) . |

### Request body

The request body must be empty.

### Response body

The response for [`instanceConfigs.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigs) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "instanceConfigs": [
    {
      object (InstanceConfig)
    }
  ],
  "nextPageToken": string
}
```

| Fields              |                                                                                                                                                                                                                                                                                                          |
|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instanceConfigs[]` | `object ( `[`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig)` )` The list of requested instance configurations.                                                                                                                   |
| `nextPageToken`     | `string` `nextPageToken` can be sent in a subsequent [`instanceConfigs.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigs) call to fetch more of the matching instance configurations. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
