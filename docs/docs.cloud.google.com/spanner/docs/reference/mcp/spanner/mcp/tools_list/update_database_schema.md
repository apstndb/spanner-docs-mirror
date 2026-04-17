## Tool: `update_database_schema`

Update schema for a given database.

The following sample demonstrate how to use `curl` to invoke the `update_database_schema` MCP tool.

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
<td><pre dir="ltr" data-is-upgraded="" data-syntax="Bash" translate="no"><code>                  
curl --location &#39;https://spanner.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;update_database_schema&quot;,
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

Enqueues the given DDL statements to be applied, in order but not necessarily all at once, to the database schema at some point (or points) in the future. The server checks that the statements are executable (syntactically valid, name tables that exist, etc.) before enqueueing them, but they may still fail upon later execution (for example, if a statement from another batch of statements is applied first and it conflicts in some way, or if there is some data-related problem like a `NULL` value in a column to which `NOT NULL` would be added). If a statement fails, all subsequent statements in the batch are automatically cancelled.

Each batch of statements is assigned a name which can be used with the `Operations` API to monitor progress. See the `operation_id` field for more details.

### UpdateDatabaseDdlRequest

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
  &quot;database&quot;: string,
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

`database`

`string`

Required. The database to update.

`statements[]`

`string`

Required. DDL statements to be applied to the database.

`operationId`

`string`

If empty, the new update request is assigned an automatically-generated operation ID. Otherwise, `operation_id` is used to construct the name of the resulting Operation.

Specifying an explicit operation ID simplifies determining whether the statements were executed in the event that the `UpdateDatabaseDdl` call is replayed, or the return value is otherwise lost: the `database` and `operation_id` fields can be combined to form the `name` of the resulting longrunning.Operation: `<database>/operations/<operation_id>` .

`operation_id` should be unique within the database, and must be a valid identifier: `[a-z][a-z0-9_]*` . Note that automatically-generated operation IDs always begin with an underscore. If the named operation already exists, `UpdateDatabaseDdl` returns `ALREADY_EXISTS` .

`protoDescriptors`

`string ( bytes format)`

Optional. Proto descriptors used by CREATE/ALTER PROTO BUNDLE statements. Contains a protobuf-serialized [google.protobuf.FileDescriptorSet](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) . To generate it, [install](https://grpc.io/docs/protoc-installation/) and run `protoc` with --include\_imports and --descriptor\_set\_out. For example, to generate for moon/shot/app.proto, run

    $protoc  --proto_path=/app_path --proto_path=/lib_path \
             --include_imports \
             --descriptor_set_out=descriptors.data \
             moon/shot/app.proto

For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;name&quot;: string,&quot;metadata&quot;: {&quot;@type&quot;: string,field1: ...,...},&quot;done&quot;: boolean,// Union field result can be only one of the following:&quot;error&quot;: {object (Status)},&quot;response&quot;: {&quot;@type&quot;: string,field1: ...,...}// End of list of possible types for union field result.}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`name`

`string`

The server-assigned name, which is only unique within the same service that originally returns it. If you use the default HTTP mapping, the `name` should be a resource name ending with `operations/{unique_id}` .

`metadata`

`object`

Service-specific metadata associated with the operation. It typically contains progress information and common metadata such as create time. Some services might not provide such metadata. Any method that returns a long-running operation should document the metadata type, if any.

An object containing fields of an arbitrary type. An additional field `"@type"` contains a URI identifying the type. Example: `{ "id": 1234, "@type": "types.example.com/standard/id" }` .

`done`

`boolean`

If the value is `false` , it means the operation is still in progress. If `true` , the operation is completed, and either `error` or `response` is available.

Union field `result` . The operation result, which can be either an `error` or a valid `response` . If `done` == `false` , neither `error` nor `response` is set. If `done` == `true` , exactly one of `error` or `response` can be set. Some services might not provide the result. `result` can be only one of the following:

`error`

` object ( Status  ` )

The error result of the operation in case of failure or cancellation.

`response`

`object`

The normal, successful response of the operation. If the original method returns no data on success, such as `Delete` , the response is `google.protobuf.Empty` . If the original method is standard `Get` / `Create` / `Update` , the response should be the resource. For other methods, the response should have the type `XxxResponse` , where `Xxx` is the original method name. For example, if the original method name is `TakeSnapshot()` , the inferred response type is `TakeSnapshotResponse` .

An object containing fields of an arbitrary type. An additional field `"@type"` contains a URI identifying the type. Example: `{ "id": 1234, "@type": "types.example.com/standard/id" }` .

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;typeUrl&quot;: string,
  &quot;value&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`typeUrl`

`string`

Identifies the type of the serialized Protobuf message with a URI reference consisting of a prefix ending in a slash and the fully-qualified type name.

Example: type.googleapis.com/google.protobuf.StringValue

This string must contain at least one `/` character, and the content after the last `/` must be the fully-qualified name of the type in canonical form, without a leading dot. Do not write a scheme on these URI references so that clients do not attempt to contact them.

The prefix is arbitrary and Protobuf implementations are expected to simply strip off everything up to and including the last `/` to identify the type. `type.googleapis.com/` is a common default prefix that some legacy implementations require. This prefix does not indicate the origin of the type, and URIs containing it are not expected to respond to any requests.

All type URL strings must be legal URI references with the additional restriction (for the text format) that the content of the reference must consist only of alphanumeric characters, percent-encoded escapes, and characters in the following set (not including the outer backticks): `/-.~_!$&()*+,;=` . Despite our allowing percent encodings, implementations should not unescape them to prevent confusion with existing parsers. For example, `type.googleapis.com%2FFoo` should be rejected.

In the original design of `Any` , the possibility of launching a type resolution service at these type URLs was considered but Protobuf never implemented one and considers contacting these URLs to be problematic and a potential security issue. Do not attempt to contact type URLs.

`value`

`string ( bytes format)`

Holds a Protobuf serialization of the type described by type\_url.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
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

`code`

`integer`

The status code, which should be an enum value of `google.rpc.Code` .

`message`

`string`

A developer-facing error message, which should be in English. Any user-facing error message should be localized and sent in the `google.rpc.Status.details` field, or localized by the client.

`details[]`

`object`

A list of messages that carry the error details. There is a common set of message types for APIs to use.

An object containing fields of an arbitrary type. An additional field `"@type"` contains a URI identifying the type. Example: `{ "id": 1234, "@type": "types.example.com/standard/id" }` .

### Tool Annotations

Destructive Hint: ✅ | Idempotent Hint: ❌ | Read Only Hint: ❌ | Open World Hint: ❌
