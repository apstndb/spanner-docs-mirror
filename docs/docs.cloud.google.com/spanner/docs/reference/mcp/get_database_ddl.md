## Tool: `       get_database_ddl      `

Get database schema for a given database.

The following sample demonstrate how to use `  curl  ` to invoke the `  get_database_ddl  ` MCP tool.

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
    &quot;name&quot;: &quot;get_database_ddl&quot;,
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

The request for `  GetDatabaseDdl  ` .

### GetDatabaseDdlRequest

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
  &quot;database&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  database  `

`  string  `

Required. The database whose schema we wish to get. Values are of the form `  projects/<project>/instances/<instance>/databases/<database>  `

## Output Schema

The response for `  GetDatabaseDdl  ` .

### GetDatabaseDdlResponse

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
  &quot;statements&quot;: [
    string
  ],
  &quot;protoDescriptors&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  statements[]  `

`  string  `

A list of formatted DDL statements defining the schema of the database specified in the request.

`  protoDescriptors  `

`  string ( bytes format)  `

Proto descriptors stored in the database. Contains a protobuf-serialized [google.protobuf.FileDescriptorSet](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) . For more details, see protobuffer [self description](https://developers.google.com/protocol-buffers/docs/techniques#self-description) .

A base64-encoded string.

### Tool Annotations

Destructive Hint: ❌ | Idempotent Hint: ❌ | Read Only Hint: ✅ | Open World Hint: ❌
