---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_session
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/create_session
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `create_session`

Create a session in a given database for query executions using execute_sql tool.

- Session can be reused to execute multiple concurrent operations.

The following sample demonstrate how to use `curl` to invoke the `create_session` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "create_session",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `CreateSession` .

### CreateSessionRequest

**JSON representation**

```
{
  "database": string,
  "fineGrainedAccessRole": string
}
```

| Fields                  |                                                                                                                                                                 |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `database`              | `string` Required. The database in which the new session is created. Format: `projects/{project}/instances/{instance}/databases/{database}`                     |
| `fineGrainedAccessRole` | `string` Optional. The fine grained access role of the session. All operations performed using the session will be performed with the fine grained access role. |

## Output Schema

A session for Spanner API.

### Session

**JSON representation**

```
{
  "name": string
}
```

| Fields |                                                                                                                                                   |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| `name` | `string` Output only. The resource name of the session. Format: `projects/{project}/instances/{instance}/databases/{database}/sessions/{session}` |

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ❌ \| Read Only Hint: ❌ \| Open World Hint: ❌
