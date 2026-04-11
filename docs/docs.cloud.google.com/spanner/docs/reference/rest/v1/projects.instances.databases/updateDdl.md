  - [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.HTTP_TEMPLATE)
  - [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.PATH_PARAMETERS)
  - [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.request_body)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.request_body.SCHEMA_REPRESENTATION)
  - [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.response_body)
  - [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#body.aspect)
  - [Try it\!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it)

Updates the schema of a Cloud Spanner database by creating/altering/dropping tables, columns, indexes, etc. The returned long-running operation will have a name of the format `<database_name>/operations/<operationId>` and can be used to track execution of the schema changes. The metadata field type is `  UpdateDatabaseDdlMetadata  ` . The operation has no response.

### HTTP request

Choose a location:

global

europe-west8

me-central2

us-central1

us-central2

us-east1

us-east4

us-east5

us-south1

us-west1

us-west2

us-west3

us-west4

us-west8

us-east7

  
`PATCH https://spanner.googleapis.com/v1/{database=projects/*/instances/*/databases/*}/ddl`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

Parameters

`database`

`string`

Required. The database to update.

Authorization requires the following [IAM](https://cloud.google.com/iam/docs/) permission on the specified resource `database` :

  - `spanner.databases.updateDdl`

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;statements&quot;: [
    string
  ],
  &quot;operationId&quot;: string,
  &quot;protoDescriptors&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`statements[]`

`string`

Required. DDL statements to be applied to the database.

`operationId`

`string`

If empty, the new update request is assigned an automatically-generated operation ID. Otherwise, `operationId` is used to construct the name of the resulting Operation.

Specifying an explicit operation ID simplifies determining whether the statements were executed in the event that the `  databases.updateDdl  ` call is replayed, or the return value is otherwise lost: the `  database  ` and `operationId` fields can be combined to form the `name` of the resulting longrunning.Operation: `<database>/operations/<operationId>` .

`operationId` should be unique within the database, and must be a valid identifier: `[a-z][a-z0-9_]*` . Note that automatically-generated operation IDs always begin with an underscore. If the named operation already exists, `  databases.updateDdl  ` returns `ALREADY_EXISTS` .

`protoDescriptors`

`string ( bytes format)`

Optional. Proto descriptors used by CREATE/ALTER PROTO BUNDLE statements. Contains a protobuf-serialized [google.protobuf.FileDescriptorSet](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) . To generate it, [install](https://grpc.io/docs/protoc-installation/) and run `protoc` with --include\_imports and --descriptor\_set\_out. For example, to generate for moon/shot/app.proto, run

    $protoc  --proto_path=/app_path --proto_path=/lib_path \
             --include_imports \
             --descriptor_set_out=descriptors.data \
             moon/shot/app.proto

For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

### Response body

If successful, the response body contains an instance of `  Operation  ` .

### Authorization scopes

Requires one of the following OAuth scopes:

  - `https://www.googleapis.com/auth/spanner.admin`
  - `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
