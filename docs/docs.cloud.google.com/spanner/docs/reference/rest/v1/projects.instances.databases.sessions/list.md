---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list
title: 'Method: projects.instances.databases.sessions.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.ListSessionsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#try-it)

Lists all sessions in a given database.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/sessions`

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
<td><code>database</code></td>
<td><p><code>string</code></p>
<p>Required. The database in which to list sessions.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.sessions.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

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
<td><code>pageSize</code></td>
<td><p><code>integer</code></p>
<p>Number of sessions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="even">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>pageToken</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.ListSessionsResponse.FIELDS.next_page_token"><code>nextPageToken</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#body.ListSessionsResponse"><code>ListSessionsResponse</code></a> .</p></td>
</tr>
<tr class="odd">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:</p>
<ul>
<li><code>labels.key</code> where key is the name of a label</li>
</ul>
<p>Some examples of using filters are:</p>
<ul>
<li><code>labels.env:*</code> --&gt; The session has the label "env".</li>
<li><code>labels.env:dev</code> --&gt; The session has the label "env" and the value of the label contains the string "dev".</li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`sessions.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#google.spanner.v1.Spanner.ListSessions) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "sessions": [
    {
      object (Session)
    }
  ],
  "nextPageToken": string
}
```

| Fields          |                                                                                                                                                                                                                                                                     |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `sessions[]`    | `object ( `[`Session`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions#Session)` )` The list of requested sessions.                                                                                              |
| `nextPageToken` | `string` `nextPageToken` can be sent in a subsequent [`sessions.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/list#google.spanner.v1.Spanner.ListSessions) call to fetch more of the matching sessions. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.data`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
