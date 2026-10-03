---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_database_ddl
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_database_ddl
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `get_database_ddl`

Get database schema for a given database.

The following sample demonstrate how to use `curl` to invoke the `get_database_ddl` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "get_database_ddl",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `GetDatabaseDdl` .

### GetDatabaseDdlRequest

**JSON representation**

```
{
  "database": string
}
```

| Fields     |                                                                                                                                                    |
|------------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| `database` | `string` Required. The database whose schema we wish to get. Values are of the form `projects/<project>/instances/<instance>/databases/<database>` |

## Output Schema

The response for `GetDatabaseDdl` .

### GetDatabaseDdlResponse

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

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ✅ \| Read Only Hint: ✅ \| Open World Hint: ❌
