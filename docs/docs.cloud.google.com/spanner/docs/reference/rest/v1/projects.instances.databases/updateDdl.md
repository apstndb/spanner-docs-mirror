---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl
title: 'Method: projects.instances.databases.updateDdl'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it)

Updates the schema of a Cloud Spanner database by creating/altering/dropping tables, columns, indexes, etc. The returned long-running operation will have a name of the format `<database_name>/operations/<operationId>` and can be used to track execution of the schema changes. The metadata field type is [`UpdateDatabaseDdlMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseDdlMetadata) . The operation has no response.

### HTTP request

Choose a location:

  
`PATCH https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/ddl`

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
<p>Required. The database to update.</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>database</code> :</p>
<ul>
<li><code>spanner.databases.updateDdl</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "statements": [
    string
  ],
  "operationId": string,
  "protoDescriptors": string
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>statements[]</code></td>
<td><p><code>string</code></p>
<p>Required. DDL statements to be applied to the database.</p></td>
</tr>
<tr class="even">
<td><code>operationId</code></td>
<td><p><code>string</code></p>
<p>If empty, the new update request is assigned an automatically-generated operation ID. Otherwise, <code>operationId</code> is used to construct the name of the resulting Operation.</p>
<p>Specifying an explicit operation ID simplifies determining whether the statements were executed in the event that the <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl"><code>databases.updateDdl</code></a> call is replayed, or the return value is otherwise lost: the <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.PATH_PARAMETERS.database"><code>database</code></a> and <code>operationId</code> fields can be combined to form the <code>name</code> of the resulting longrunning.Operation: <code>&lt;database&gt;/operations/&lt;operationId&gt;</code> .</p>
<p><code>operationId</code> should be unique within the database, and must be a valid identifier: <code>[a-z][a-z0-9_]*</code> . Note that automatically-generated operation IDs always begin with an underscore. If the named operation already exists, <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabaseDdl"><code>databases.updateDdl</code></a> returns <code>ALREADY_EXISTS</code> .</p></td>
</tr>
<tr class="odd">
<td><code>protoDescriptors</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>bytes</code></a><code> format)</code></p>
<p>Optional. Proto descriptors used by CREATE/ALTER PROTO BUNDLE statements. Contains a protobuf-serialized <a href="https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto">google.protobuf.FileDescriptorSet</a> . To generate it, <a href="https://grpc.io/docs/protoc-installation/">install</a> and run <code>protoc</code> with --include_imports and --descriptor_set_out. For example, to generate for moon/shot/app.proto, run</p>
<pre data-fenced=""><code>$protoc  --proto_path=/app_path --proto_path=/lib_path \
         --include_imports \
         --descriptor_set_out=descriptors.data \
         moon/shot/app.proto</code></pre>
<p>For more details, see protobuffer <a href="https://developers.google.com/protocol-buffers/docs/techniques#self-description">self description</a> .</p>
<p>A base64-encoded string.</p></td>
</tr>
</tbody>
</table>

### Response body

If successful, the response body contains an instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
