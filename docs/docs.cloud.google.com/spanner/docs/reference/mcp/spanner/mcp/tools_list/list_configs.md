---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `list_configs`

List instance configs in a given project.

- Response may include next_page_token to fetch additional configs using list_configs tool with page_token set.

The following sample demonstrate how to use `curl` to invoke the `list_configs` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "list_configs",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `ListInstanceConfigs` .

### ListInstanceConfigsRequest

**JSON representation**

```
{
  "parent": string,
  "pageSize": integer,
  "pageToken": string
}
```

| Fields      |                                                                                                                                                                                                                                                                  |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `parent`    | `string` Required. The name of the project for which a list of supported instance configurations is requested. Values are of the form `projects/<project>` .                                                                                                     |
| `pageSize`  | `integer` Number of instance configurations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                    |
| `pageToken` | `string` If non-empty, `page_token` should contain a `next_page_token` from a previous [`ListInstanceConfigsResponse`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs#Output.Schema.ListInstanceConfigsResponse) . |

## Output Schema

The response for `ListInstanceConfigs` .

### ListInstanceConfigsResponse

**JSON representation**

```
{
  "instanceConfigs": [
    {
      object (InstanceConfig)
    }
  ],
  "nextPageToken": string
}
```

| Fields              |                                                                                                                                                                                                             |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instanceConfigs[]` | `object ( `[`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs#Output.Schema.InstanceConfig)` )` The list of requested instance configurations. |
| `nextPageToken`     | `string` `next_page_token` can be sent in a subsequent `ListInstanceConfigs` call to fetch more of the matching instance configurations.                                                                    |

### InstanceConfig

**JSON representation**

```
{
  "name": string,
  "displayName": string,
  "configType": enum (Type),
  "replicas": [
    {
      object (ReplicaInfo)
    }
  ],
  "optionalReplicas": [
    {
      object (ReplicaInfo)
    }
  ],
  "baseConfig": string,
  "labels": {
    string: string,
    ...
  },
  "etag": string,
  "leaderOptions": [
    string
  ],
  "reconciling": boolean,
  "state": enum (State),
  "freeInstanceAvailability": enum (FreeInstanceAvailability),
  "quorumType": enum (QuorumType),
  "storageLimitPerProcessingUnit": string
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
<td><code>name</code></td>
<td><p><code>string</code></p>
<p>A unique identifier for the instance configuration. Values are of the form <code>projects/&lt;project&gt;/instanceConfigs/[a-z][-a-z0-9]*</code> .</p>
<p>User instance configuration must start with <code>custom-</code> .</p></td>
</tr>
<tr class="even">
<td><code>displayName</code></td>
<td><p><code>string</code></p>
<p>The name of this instance configuration as it appears in UIs.</p></td>
</tr>
<tr class="odd">
<td><code>configType</code></td>
<td><p><code>enum ( </code><code>Type</code><code> )</code></p>
<p>Output only. Whether this instance configuration is a Google-managed or user-managed configuration.</p></td>
</tr>
<tr class="even">
<td><code>replicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs#Output.Schema.ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>The geographic placement of nodes in this instance configuration and their replication properties.</p>
<p>To create user-managed configurations, input <code>replicas</code> must include all replicas in <code>replicas</code> of the <code>base_config</code> and include one or more replicas in the <code>optional_replicas</code> of the <code>base_config</code> .</p></td>
</tr>
<tr class="odd">
<td><code>optionalReplicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs#Output.Schema.ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.</p></td>
</tr>
<tr class="even">
<td><code>baseConfig</code></td>
<td><p><code>string</code></p>
<p>Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. <code>base_config</code> must refer to a configuration of type <code>GOOGLE_MANAGED</code> in the same project as this configuration.</p></td>
</tr>
<tr class="odd">
<td><code>labels</code></td>
<td><p><code>map (key: string, value: string)</code></p>
<p>Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).</p>
<ul>
<li>Label keys must be between 1 and 63 characters long and must conform to the following regular expression: <code>[a-z][a-z0-9_-]{0,62}</code> .</li>
<li>Label values must be between 0 and 63 characters long and must conform to the regular expression <code>[a-z0-9_-]{0,63}</code> .</li>
<li>No more than 64 labels can be associated with a given resource.</li>
</ul>
<p>See <a href="https://goo.gl/xmQnxf">https://goo.gl/xmQnxf</a> for more information on and examples of labels.</p>
<p>If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "_" + value would prove problematic if we were to allow "_" in a future release.</p>
<p>An object containing a list of <code>"key": value</code> pairs. Example: <code>{ "name": "wrench", "mass": "1.3kg", "count": "3" }</code> .</p></td>
</tr>
<tr class="even">
<td><code>etag</code></td>
<td><p><code>string</code></p>
<p>etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.</p></td>
</tr>
<tr class="odd">
<td><code>leaderOptions[]</code></td>
<td><p><code>string</code></p>
<p>Allowed values of the "default_leader" schema option for databases in instances that use this instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>reconciling</code></td>
<td><p><code>boolean</code></p>
<p>Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>state</code></td>
<td><p><code>enum ( </code><code>State</code><code> )</code></p>
<p>Output only. The current instance configuration state. Applicable only for <code>USER_MANAGED</code> configurations.</p></td>
</tr>
<tr class="even">
<td><code>freeInstanceAvailability</code></td>
<td><p><code>enum ( </code><code>FreeInstanceAvailability</code><code> )</code></p>
<p>Output only. Describes whether free instances are available to be created in this instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>quorumType</code></td>
<td><p><code>enum ( </code><code>QuorumType</code><code> )</code></p>
<p>Output only. The <code>QuorumType</code> of the instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>storageLimitPerProcessingUnit</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>int64</code></a><code> format)</code></p>
<p>Output only. The storage limit in bytes per processing unit.</p></td>
</tr>
</tbody>
</table>

### ReplicaInfo

**JSON representation**

```
{
  "location": string,
  "type": enum (ReplicaType),
  "defaultLeaderLocation": boolean
}
```

| Fields                  |                                                                                                                                                                                                                                      |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `location`              | `string` The location of the serving resources, e.g., "us-central1".                                                                                                                                                                 |
| `type`                  | `enum ( ``ReplicaType`` )` The type of replica.                                                                                                                                                                                      |
| `defaultLeaderLocation` | `boolean` If true, this location is designated as the default leader location where leader replicas are placed. See the [region types documentation](https://cloud.google.com/spanner/docs/instances#region_types) for more details. |

### LabelsEntry

**JSON representation**

```
{
  "key": string,
  "value": string
}
```

| Fields  |          |
|---------|----------|
| `key`   | `string` |
| `value` | `string` |

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ✅ \| Read Only Hint: ✅ \| Open World Hint: ❌
