---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl
title: 'Method: projects.instances.databases.getDdl'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.GetDatabaseDdlResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#try-it)

Returns the schema of a Cloud Spanner database as a list of formatted DDL statements. This method does not show pending schema updates, those may be queried using the `Operations` API.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/ddl`

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
<p>Required. The database whose schema we wish to get. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/databases/&lt;database&gt;</code></p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.databases.getDdl</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`databases.getDdl`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl#google.spanner.admin.database.v1.DatabaseAdmin.GetDatabaseDdl) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "statements": [
    string
  ],
  "protoDescriptors": string
}
```

| Fields             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|--------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `statements[]`     | `string` A list of formatted DDL statements defining the schema of the database specified in the request.                                                                                                                                                                                                                                                                                                                                                          |
| `protoDescriptors` | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Proto descriptors stored in the database. Contains a protobuf-serialized [google.protobuf.FileDescriptorSet](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) . For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) . A base64-encoded string. |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
