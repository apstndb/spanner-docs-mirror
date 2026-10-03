---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/databaseinsights/mcp/tools_list/get_system_metrics
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/databaseinsights/mcp/tools_list/get_system_metrics
title: 'MCP Tools Reference: databaseinsights.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `get_system_metrics`

Fetches system related telemetry data for a given database instance using a PromQL query from cloud monitoring.

The following sample demonstrate how to use `curl` to invoke the `get_system_metrics` MCP tool.

**Curl Request**

```
curl --location 'https://databaseinsights.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "get_system_metrics",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request of the FetchSystemMetrics RPC.

### FetchSystemMetricsRequest

**JSON representation**

```
{
  "parent": string,
  "resource": string,
  "promqlQuery": string,
  "startTime": string,
  "endTime": string
}
```

| Fields        |                                                                                                                         |
|---------------|-------------------------------------------------------------------------------------------------------------------------|
| `parent`      | `string` Required. The name of the location where we request metrics. Format: `projects/{project}/locations/{location}` |
| `resource`    | `string` Required. The resource name used for system metrics. It is the project ID in this case.                        |
| `promqlQuery` | `string` Required. The promql_query to fetch the system metrics.                                                        |
| `startTime`   | `string` Optional. The start_time of the metrics in RFC3339 format.                                                     |
| `endTime`     | `string` Optional. The end_time of the metrics in RFC3339 format.                                                       |

## Output Schema

The response of the FetchSystemMetrics RPC.

### FetchSystemMetricsResponse

**JSON representation**

```
{
  "promqlResult": string
}
```

| Fields         |                                                  |
|----------------|--------------------------------------------------|
| `promqlResult` | `string` The result of the fetch system metrics. |

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ✅ \| Read Only Hint: ✅ \| Open World Hint: ❌
