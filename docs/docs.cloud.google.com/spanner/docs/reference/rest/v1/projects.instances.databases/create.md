  - [HTTP request](#body.HTTP_TEMPLATE)
  - [Path parameters](#body.PATH_PARAMETERS)
  - [Request body](#body.request_body)
      - [JSON representation](#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](#body.response_body)
  - [Authorization scopes](#body.aspect)
  - [Try it\!](#try-it)

Creates a new Spanner database and starts to prepare it for serving. The returned long-running operation will have a name of the format `  <database_name>/operations/<operationId>  ` and can be used to track preparation of the database. The metadata field type is `  CreateDatabaseMetadata  ` . The response field type is `  Database  ` , if successful.

### HTTP request

Choose a location:

  
`  POST https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/databases  `

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`  parent  `

`  string  `

Required. The name of the instance that will serve the new database. Values are of the form `  projects/<project>/instances/<instance>  ` .

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `  parent  ` :

  - `  spanner.databases.create  `

### Request body

The request body contains data with the following structure:

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;createStatement&quot;: string,
  &quot;extraStatements&quot;: [
    string
  ],
  &quot;encryptionConfig&quot;: {
    object (EncryptionConfig)
  },
  &quot;databaseDialect&quot;: enum (DatabaseDialect),
  &quot;protoDescriptors&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  createStatement  `

`  string  `

Required. A `  CREATE DATABASE  ` statement, which specifies the ID of the new database. The database ID must conform to the regular expression `  [a-z][a-z0-9_\-]*[a-z0-9]  ` and be between 2 and 30 characters in length. If the database ID is a reserved word or if it contains a hyphen, the database ID must be enclosed in backticks ( ``  `  `` ).

`  extraStatements[]  `

`  string  `

Optional. A list of DDL statements to run inside the newly created database. Statements can create tables, indexes, etc. These statements execute atomically with the creation of the database: if there is an error in any statement, the database is not created.

`  encryptionConfig  `

`  object ( EncryptionConfig  ` )

Optional. The encryption configuration for the database. If this field is not specified, Cloud Spanner will encrypt/decrypt all data at rest using Google default encryption.

`  databaseDialect  `

`  enum ( DatabaseDialect  ` )

Optional. The dialect of the Cloud Spanner Database.

`  protoDescriptors  `

`  string ( bytes format)  `

Optional. Proto descriptors used by `  CREATE/ALTER PROTO BUNDLE  ` statements in 'extraStatements'. Contains a protobuf-serialized [`  google.protobuf.FileDescriptorSet  `](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) descriptor set. To generate it, [install](https://grpc.io/docs/protoc-installation/) and run `  protoc  ` with --include\_imports and --descriptor\_set\_out. For example, to generate for moon/shot/app.proto, run

``` text
$protoc  --proto_path=/app_path --proto_path=/lib_path \
         --include_imports \
         --descriptor_set_out=descriptors.data \
         moon/shot/app.proto
```

For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

### Response body

If successful, the response body contains a newly created instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `  https://www.googleapis.com/auth/spanner.admin  `
  - `  https://www.googleapis.com/auth/cloud-platform  `

For more information, see the [Authentication Overview](/docs/authentication#authorization-gcp) .
