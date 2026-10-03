---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter
title: 'Method: projects.instances.databases.sessions.adapter'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.Session.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#try-it)

Creates a new session to be used for requests made by the adapter. A session identifies a specific incarnation of a database resource and is meant to be reused across many `sessions.adaptMessage` calls.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*/databases/*}/sessions:adapter`

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
<p>Required. The database in which the new session is created.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.sessions.create</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains an instance of [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/adapter#body.Session) .

### Response body

A session in the Cloud Spanner Adapter API.

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "name": string
}
```

| Fields |                                                                               |
|--------|-------------------------------------------------------------------------------|
| `name` | `string` Identifier. The name of the session. This is always system-assigned. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
