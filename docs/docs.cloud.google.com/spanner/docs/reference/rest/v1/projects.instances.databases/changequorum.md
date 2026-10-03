---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum
title: 'Method: projects.instances.databases.changequorum'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/changequorum#try-it)

`databases.changequorum` is strictly restricted to databases that use dual-region instance configurations.

Initiates a background operation to change the quorum of a database from dual-region mode to single-region mode or vice versa.

The returned long-running operation has a name of the format `projects/<project>/instances/<instance>/databases/<database>/operations/<operationId>` and can be used to track execution of the `databases.changequorum` . The metadata field type is [`ChangeQuorumMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ChangeQuorumMetadata) .

Authorization requires `spanner.databases.changequorum` permission on the resource database.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{name=projects/*/instances/*/databases/*}:changequorum`

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
<p>Required. Name of the database in which to apply <code>databases.changequorum</code> . Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/databases/&lt;database&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>name</code> :</p>
<ul>
<li><code>spanner.databases.changequorum</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "quorumType": {
    object (QuorumType)
  },
  "etag": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                          |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `quorumType` | `object ( `[`QuorumType`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#QuorumType)` )` Required. The type of this quorum.                                                                                                                                                                                                                                   |
| `etag`       | `string` Optional. The etag is the hash of the `QuorumInfo` . The `databases.changequorum` operation is only performed if the etag matches that of the `QuorumInfo` in the current database resource. Otherwise the API returns an `ABORTED` error. The etag is used for optimistic concurrency control as a way to help prevent simultaneous change quorum requests that could create a race condition. |

### Response body

If successful, the response body contains an instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
