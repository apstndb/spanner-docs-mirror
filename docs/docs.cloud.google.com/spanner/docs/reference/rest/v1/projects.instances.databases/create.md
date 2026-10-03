---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create
title: 'Method: projects.instances.databases.create'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.request_body.SCHEMA_REPRESENTATION)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#try-it)

Creates a new Spanner database and starts to prepare it for serving. The returned long-running operation will have a name of the format `<database_name>/operations/<operationId>` and can be used to track preparation of the database. The metadata field type is [`CreateDatabaseMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateDatabaseMetadata) . The response field type is [`Database`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#Database) , if successful.

### HTTP request

Choose a location:

  
`POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases`

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
<p>Required. The name of the instance that will serve the new database. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.databases.create</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "createStatement": string,
  "extraStatements": [
    string
  ],
  "encryptionConfig": {
    object (EncryptionConfig)
  },
  "databaseDialect": enum (DatabaseDialect),
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
<td><code>createStatement</code></td>
<td><p><code>string</code></p>
<p>Required. A <code>CREATE DATABASE</code> statement, which specifies the ID of the new database. The database ID must conform to the regular expression <code>[a-z][a-z0-9_\-]*[a-z0-9]</code> and be between 2 and 30 characters in length. If the database ID is a reserved word or if it contains a hyphen, the database ID must be enclosed in backticks ( <code>`</code> ).</p></td>
</tr>
<tr class="even">
<td><code>extraStatements[]</code></td>
<td><p><code>string</code></p>
<p>Optional. A list of DDL statements to run inside the newly created database. Statements can create tables, indexes, etc. These statements execute atomically with the creation of the database: if there is an error in any statement, the database is not created.</p></td>
</tr>
<tr class="odd">
<td><code>encryptionConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#EncryptionConfig"><code>EncryptionConfig</code></a><code> )</code></p>
<p>Optional. The encryption configuration for the database. If this field is not specified, Cloud Spanner will encrypt/decrypt all data at rest using Google default encryption.</p></td>
</tr>
<tr class="even">
<td><code>databaseDialect</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/DatabaseDialect"><code>DatabaseDialect</code></a><code> )</code></p>
<p>Optional. The dialect of the Cloud Spanner Database.</p></td>
</tr>
<tr class="odd">
<td><code>protoDescriptors</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>bytes</code></a><code> format)</code></p>
<p>Optional. Proto descriptors used by <code>CREATE/ALTER PROTO BUNDLE</code> statements in 'extraStatements'. Contains a protobuf-serialized <a href="https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto"><code>google.protobuf.FileDescriptorSet</code></a> descriptor set. To generate it, <a href="https://grpc.io/docs/protoc-installation/">install</a> and run <code>protoc</code> with --include_imports and --descriptor_set_out. For example, to generate for moon/shot/app.proto, run</p>
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

If successful, the response body contains a newly created instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
