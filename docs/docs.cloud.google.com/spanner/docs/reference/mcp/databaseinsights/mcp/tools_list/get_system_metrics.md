## Tool: `get_system_metrics`

Fetches system related telemetry data for a given database instance using a PromQL query from cloud monitoring.

The following sample demonstrate how to use `curl` to invoke the `get_system_metrics` MCP tool.

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
curl --location &#39;https://databaseinsights.googleapis.com/mcp&#39; \
--header &#39;content-type: application/json&#39; \
--header &#39;accept: application/json, text/event-stream&#39; \
--data &#39;{
  &quot;method&quot;: &quot;tools/call&quot;,
  &quot;params&quot;: {
    &quot;name&quot;: &quot;get_system_metrics&quot;,
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

The request of the FetchSystemMetrics RPC.

### FetchSystemMetricsRequest

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
  &quot;parent&quot;: string,
  &quot;resource&quot;: string,
  &quot;promqlQuery&quot;: string,
  &quot;startTime&quot;: string,
  &quot;endTime&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`parent`

`string`

Required. The name of the location where we request metrics. Format: `projects/{project}/locations/{location}`

`resource`

`string`

Required. The resource name used for system metrics. It is the project ID in this case.

`promqlQuery`

`string`

Required. The promql\_query to fetch the system metrics.

`startTime`

`string`

Optional. The start\_time of the metrics in RFC3339 format.

`endTime`

`string`

Optional. The end\_time of the metrics in RFC3339 format.

## Output Schema

The response of the FetchSystemMetrics RPC.

### FetchSystemMetricsResponse

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
  &quot;promqlResult&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`promqlResult`

`string`

The result of the fetch system metrics.

### Tool Annotations

Destructive Hint: ❌ | Idempotent Hint: ✅ | Read Only Hint: ✅ | Open World Hint: ❌
