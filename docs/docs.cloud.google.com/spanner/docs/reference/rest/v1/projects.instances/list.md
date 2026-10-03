---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list
title: 'Method: projects.instances.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.ListInstancesResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#try-it)

Lists all instances in the given project.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*}/instances`

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
<p>Required. The name of the project for which a list of instances is requested. Values are of the form <code>projects/&lt;project&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.instances.list</code></li>
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
<p>Number of instances to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="even">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>pageToken</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.ListInstancesResponse.FIELDS.next_page_token"><code>nextPageToken</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.ListInstancesResponse"><code>ListInstancesResponse</code></a> .</p></td>
</tr>
<tr class="odd">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:</p>
<ul>
<li><code>name</code></li>
<li><code>displayName</code></li>
<li><code>labels.key</code> where key is the name of a label</li>
</ul>
<p>Some examples of using filters are:</p>
<ul>
<li><code>name:*</code> --&gt; The instance has a name.</li>
<li><code>name:Howl</code> --&gt; The instance's name contains the string "howl".</li>
<li><code>name:HOWL</code> --&gt; Equivalent to above.</li>
<li><code>NAME:howl</code> --&gt; Equivalent to above.</li>
<li><code>labels.env:*</code> --&gt; The instance has the label "env".</li>
<li><code>labels.env:dev</code> --&gt; The instance has the label "env" and the value of the label contains the string "dev".</li>
<li><code>name:howl labels.env:dev</code> --&gt; The instance's name contains "howl" and it has the label "env" with its value containing "dev".</li>
</ul></td>
</tr>
<tr class="even">
<td><code>instanceDeadline</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Deadline used while retrieving metadata for instances. Instances whose metadata cannot be retrieved within this deadline will be added to <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.ListInstancesResponse.FIELDS.unreachable"><code>unreachable</code></a> in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.ListInstancesResponse"><code>ListInstancesResponse</code></a> .</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`instances.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstances) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "instances": [
    {
      object (Instance)
    }
  ],
  "nextPageToken": string,
  "unreachable": [
    string
  ]
}
```

| Fields          |                                                                                                                                                                                                                                                                                  |
|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instances[]`   | `object ( `[`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance)` )` The list of requested instances.                                                                                                                           |
| `nextPageToken` | `string` `nextPageToken` can be sent in a subsequent [`instances.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstances) call to fetch more of the matching instances.         |
| `unreachable[]` | `string` The list of unreachable instances. It includes the names of instances whose metadata could not be retrieved within [`instanceDeadline`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list#body.QUERY_PARAMETERS.instance_deadline) . |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
