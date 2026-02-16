## Tool: `       create_database      `

Create a Spanner database in a given instance.

The following sample demonstrate how to use `  curl  ` to invoke the `  create_database  ` MCP tool.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>Curl Request</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;create_database&quot;,
    &quot;arguments&quot;: {
      // provide these details according to the tool&#39;s MCP specification
    }
  },
  &quot;jsonrpc&quot;: &quot;2.0&quot;,
  &quot;id&quot;: 1
}&#39;
                </code></pre></td>
</tr>
</tbody>
</table>

## Input Schema

The request for `  CreateDatabase  ` .

### CreateDatabaseRequest

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
  &quot;parent&quot;: string,
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

`  parent  `

`  string  `

Required. The name of the instance that will serve the new database. Values are of the form `  projects/<project>/instances/<instance>  ` .

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

Optional. Proto descriptors used by `  CREATE/ALTER PROTO BUNDLE  ` statements in 'extra\_statements'. Contains a protobuf-serialized [`  google.protobuf.FileDescriptorSet  `](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) descriptor set. To generate it, [install](https://grpc.io/docs/protoc-installation/) and run `  protoc  ` with --include\_imports and --descriptor\_set\_out. For example, to generate for moon/shot/app.proto, run

``` text
$protoc  --proto_path=/app_path --proto_path=/lib_path \
         --include_imports \
         --descriptor_set_out=descriptors.data \
         moon/shot/app.proto
```

For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

### EncryptionConfig

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
  &quot;kmsKeyName&quot;: string,
  &quot;kmsKeyNames&quot;: [
    string
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  kmsKeyName  `

`  string  `

The Cloud KMS key to be used for encrypting and decrypting the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

`  kmsKeyNames[]  `

`  string  `

Specifies the KMS configuration for one or more keys used to encrypt the database. Values are of the form `  projects/<project>/locations/<location>/keyRings/<key_ring>/cryptoKeys/<kms_key_name>  ` .

The keys referenced by `  kms_key_names  ` must fully cover all regions of the database's instance configuration. Some examples:

  - For regional (single-region) instance configurations, specify a regional location KMS key.
  - For multi-region instance configurations of type `  GOOGLE_MANAGED  ` , either specify a multi-region location KMS key or multiple regional location KMS keys that cover all regions in the instance configuration.
  - For an instance configuration of type `  USER_MANAGED  ` , specify only regional location KMS keys to cover each region in the instance configuration. Multi-region location KMS keys aren't supported for `  USER_MANAGED  ` type instance configurations.

## Output Schema

This resource represents a long-running operation that is the result of a network API call.

### Operation

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
  &quot;name&quot;: string,
  &quot;metadata&quot;: {
    &quot;@type&quot;: string,
    field1: ...,
    ...
  },
  &quot;done&quot;: boolean,

  // Union field result can be only one of the following:
  &quot;error&quot;: {
    object (Status)
  },
  &quot;response&quot;: {
    &quot;@type&quot;: string,
    field1: ...,
    ...
  }
  // End of list of possible types for union field result.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

The server-assigned name, which is only unique within the same service that originally returns it. If you use the default HTTP mapping, the `  name  ` should be a resource name ending with `  operations/{unique_id}  ` .

`  metadata  `

`  object  `

Service-specific metadata associated with the operation. It typically contains progress information and common metadata such as create time. Some services might not provide such metadata. Any method that returns a long-running operation should document the metadata type, if any.

An object containing fields of an arbitrary type. An additional field `  "@type"  ` contains a URI identifying the type. Example: `  { "id": 1234, "@type": "types.example.com/standard/id" }  ` .

`  done  `

`  boolean  `

If the value is `  false  ` , it means the operation is still in progress. If `  true  ` , the operation is completed, and either `  error  ` or `  response  ` is available.

Union field `  result  ` . The operation result, which can be either an `  error  ` or a valid `  response  ` . If `  done  ` == `  false  ` , neither `  error  ` nor `  response  ` is set. If `  done  ` == `  true  ` , exactly one of `  error  ` or `  response  ` can be set. Some services might not provide the result. `  result  ` can be only one of the following:

`  error  `

`  object ( Status  ` )

The error result of the operation in case of failure or cancellation.

`  response  `

`  object  `

The normal, successful response of the operation. If the original method returns no data on success, such as `  Delete  ` , the response is `  google.protobuf.Empty  ` . If the original method is standard `  Get  ` / `  Create  ` / `  Update  ` , the response should be the resource. For other methods, the response should have the type `  XxxResponse  ` , where `  Xxx  ` is the original method name. For example, if the original method name is `  TakeSnapshot()  ` , the inferred response type is `  TakeSnapshotResponse  ` .

An object containing fields of an arbitrary type. An additional field `  "@type"  ` contains a URI identifying the type. Example: `  { "id": 1234, "@type": "types.example.com/standard/id" }  ` .

### Any

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
  &quot;typeUrl&quot;: string,
  &quot;value&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  typeUrl  `

`  string  `

A URL/resource name that uniquely identifies the type of the serialized protocol buffer message. This string must contain at least one "/" character. The last segment of the URL's path must represent the fully qualified name of the type (as in `  path/google.protobuf.Duration  ` ). The name should be in a canonical form (e.g., leading "." is not accepted).

In practice, teams usually precompile into the binary all types that they expect it to use in the context of Any. However, for URLs which use the scheme `  http  ` , `  https  ` , or no scheme, one can optionally set up a type server that maps type URLs to message definitions as follows:

  - If no scheme is provided, `  https  ` is assumed.
  - An HTTP GET on the URL must yield a `  google.protobuf.Type  ` value in binary format, or produce an error.
  - Applications are allowed to cache lookup results based on the URL, or have them precompiled into a binary to avoid any lookup. Therefore, binary compatibility needs to be preserved on changes to types. (Use versioned type names to manage breaking changes.)

Note: this functionality is not currently available in the official protobuf release, and it is not used for type URLs beginning with type.googleapis.com. As of May 2023, there are no widely used type server implementations and no plans to implement one.

Schemes other than `  http  ` , `  https  ` (or the empty scheme) might be used with implementation specific semantics.

`  value  `

`  string ( bytes format)  `

Must be a valid serialized protocol buffer of the above specified type.

A base64-encoded string.

### Status

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
  &quot;code&quot;: integer,
  &quot;message&quot;: string,
  &quot;details&quot;: [
    {
      &quot;@type&quot;: string,
      field1: ...,
      ...
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  code  `

`  integer  `

The status code, which should be an enum value of `  google.rpc.Code  ` .

`  message  `

`  string  `

A developer-facing error message, which should be in English. Any user-facing error message should be localized and sent in the `  google.rpc.Status.details  ` field, or localized by the client.

`  details[]  `

`  object  `

A list of messages that carry the error details. There is a common set of message types for APIs to use.

An object containing fields of an arbitrary type. An additional field `  "@type"  ` contains a URI identifying the type. Example: `  { "id": 1234, "@type": "types.example.com/standard/id" }  ` .

### Tool Annotations

Destructive Hint: ❌ | Idempotent Hint: ❌ | Read Only Hint: ❌ | Open World Hint: ❌
