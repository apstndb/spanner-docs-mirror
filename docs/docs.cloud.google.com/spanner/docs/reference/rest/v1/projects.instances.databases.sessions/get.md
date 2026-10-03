---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get
title: 'Method: projects.instances.databases.sessions.get'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get#try-it)

Gets a session. Returns `NOT_FOUND` if the session doesn't exist. This is mainly useful for determining whether a session is still alive.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*/sessions/*}`

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
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the session to retrieve.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.sessions.get</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of [`Session`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions#Session) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
