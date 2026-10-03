---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get
title: 'Method: projects.instances.get'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get#try-it)

Gets information about a particular instance.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{name=projects/*/instances/*}`

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
<p>Required. The name of the requested instance. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.instances.get</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `fieldMask` | `string ( `[`FieldMask`](https://protobuf.dev/reference/protobuf/google.protobuf/#field-mask)` format)` If fieldMask is present, specifies the subset of [`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) fields that should be returned. If absent, all [`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) fields are returned. This is a comma-separated list of fully qualified names of fields. Example: `"user.displayName,photo"` . |

### Request body

The request body must be empty.

### Response body

If successful, the response body contains an instance of [`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
