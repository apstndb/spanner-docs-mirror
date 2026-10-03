---
name: documents/docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_instances
uri: https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_instances
title: 'MCP Tools Reference: spanner.googleapis.com'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

## Tool: `list_instances`

List Spanner instances in a given project.

- Response may include next_page_token to fetch additional instances using list_instances tool with page_token set.

The following sample demonstrate how to use `curl` to invoke the `list_instances` MCP tool.

**Curl Request**

```
curl --location 'https://spanner.googleapis.com/mcp' \
--header 'content-type: application/json' \
--header 'accept: application/json, text/event-stream' \
--data '{
  "method": "tools/call",
  "params": {
    "name": "list_instances",
    "arguments": {
      // provide these details according to the tool's MCP specification
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}'
```

## Input Schema

The request for `ListInstances` .

### ListInstancesRequest

**JSON representation**

```
{
  "parent": string,
  "pageSize": integer,
  "pageToken": string,
  "filter": string,
  "instanceDeadline": string
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
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the project for which a list of instances is requested. Values are of the form <code>projects/&lt;project&gt;</code> .</p></td>
</tr>
<tr class="even">
<td><code>pageSize</code></td>
<td><p><code>integer</code></p>
<p>Number of instances to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="odd">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>If non-empty, <code>page_token</code> should contain a <code>next_page_token</code> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_instances#Output.Schema.ListInstancesResponse"><code>ListInstancesResponse</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>An expression for filtering the results of the request. Filter rules are case insensitive. The fields eligible for filtering are:</p>
<ul>
<li><code>name</code></li>
<li><code>display_name</code></li>
<li><code>labels.key</code> where key is the name of a label</li>
</ul>
<p>Some examples of using filters are:</p>
<ul>
<li><code>name:*</code> --&gt; The instance has a name.</li>
<li><code>name:Howl</code> --&gt; The instance's name contains the string "howl".</li>
<li><code>name:HOWL</code> --&gt; Equivalent to above.</li>
<li><code>NAME:howl</code> --&gt; Equivalent to above.</li>
<li><code>labels.env:*</code> --&gt; The instance has the label "env".</li>
<li><code>labels.env:dev</code> --&gt; The instance has the label "env" and the value of the label contains the string "dev".</li>
<li><code>name:howl labels.env:dev</code> --&gt; The instance's name contains "howl" and it has the label "env" with its value containing "dev".</li>
</ul></td>
</tr>
<tr class="odd">
<td><code>instanceDeadline</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Deadline used while retrieving metadata for instances. Instances whose metadata cannot be retrieved within this deadline will be added to <code>unreachable</code> in <a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_instances#Output.Schema.ListInstancesResponse"><code>ListInstancesResponse</code></a> .</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
</tbody>
</table>

### Timestamp

**JSON representation**

```
{
  "seconds": string,
  "nanos": integer
}
```

| Fields    |                                                                                                                                                                                                                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `seconds` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).                            |
| `nanos`   | `integer` Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive. |

## Output Schema

The response for `ListInstances` .

### ListInstancesResponse

**JSON representation**

```
{
  "instances": [
    {
      object (Instance)
    }
  ],
  "nextPageToken": string,
  "unreachable": [
    string
  ]
}
```

| Fields          |                                                                                                                                                                                   |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instances[]`   | `object ( `[`Instance`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.Instance)` )` The list of requested instances. |
| `nextPageToken` | `string` `next_page_token` can be sent in a subsequent `ListInstances` call to fetch more of the matching instances.                                                              |
| `unreachable[]` | `string` The list of unreachable instances. It includes the names of instances whose metadata could not be retrieved within `instance_deadline` .                                 |

### Instance

**JSON representation**

```
{
  "name": string,
  "config": string,
  "displayName": string,
  "nodeCount": integer,
  "processingUnits": integer,
  "replicaComputeCapacity": [
    {
      object (ReplicaComputeCapacity)
    }
  ],
  "autoscalingConfig": {
    object (AutoscalingConfig)
  },
  "state": enum (State),
  "labels": {
    string: string,
    ...
  },
  "instanceType": enum (InstanceType),
  "endpointUris": [
    string
  ],
  "createTime": string,
  "updateTime": string,
  "freeInstanceMetadata": {
    object (FreeInstanceMetadata)
  },
  "edition": enum (Edition),
  "defaultBackupScheduleType": enum (DefaultBackupScheduleType)
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
<p>Required. A unique identifier for the instance, which cannot be changed after the instance is created. Values are of the form <code>projects/&lt;project&gt;/instances/[a-z][-a-z0-9]*[a-z0-9]</code> . The final segment of the name must be between 2 and 64 characters in length.</p></td>
</tr>
<tr class="even">
<td><code>config</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the instance's configuration. Values are of the form <code>projects/&lt;project&gt;/instanceConfigs/&lt;configuration&gt;</code> . See also <a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/list_configs#Output.Schema.InstanceConfig"><code>InstanceConfig</code></a> and <code>ListInstanceConfigs</code> .</p></td>
</tr>
<tr class="odd">
<td><code>displayName</code></td>
<td><p><code>string</code></p>
<p>Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.</p></td>
</tr>
<tr class="even">
<td><code>nodeCount</code></td>
<td><p><code>integer</code></p>
<p>The number of nodes allocated to this instance. At most, one of either <code>node_count</code> or <code>processing_units</code> should be present in the message.</p>
<p>Users can set the <code>node_count</code> field to specify the target number of nodes allocated to the instance.</p>
<p>If autoscaling is enabled, <code>node_count</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of nodes allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying node count across replicas (achieved by setting <code>asymmetric_autoscaling_options</code> in the autoscaling configuration), the <code>node_count</code> set here is the maximum node count across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes, and processing units</a> .</p></td>
</tr>
<tr class="odd">
<td><code>processingUnits</code></td>
<td><p><code>integer</code></p>
<p>The number of processing units allocated to this instance. At most, one of either <code>processing_units</code> or <code>node_count</code> should be present in the message.</p>
<p>Users can set the <code>processing_units</code> field to specify the target number of processing units allocated to the instance.</p>
<p>If autoscaling is enabled, <code>processing_units</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of processing units allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying processing units per replica (achieved by setting <code>asymmetric_autoscaling_options</code> in the autoscaling configuration), the <code>processing_units</code> set here is the maximum processing units across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes and processing units</a> .</p></td>
</tr>
<tr class="even">
<td><code>replicaComputeCapacity[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.ReplicaComputeCapacity"><code>ReplicaComputeCapacity</code></a><code> )</code></p>
<p>Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.</p></td>
</tr>
<tr class="odd">
<td><code>autoscalingConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AutoscalingConfig"><code>AutoscalingConfig</code></a><code> )</code></p>
<p>Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, node_count and processing_units are treated as OUTPUT_ONLY fields and reflect the current compute capacity allocated to the instance.</p></td>
</tr>
<tr class="even">
<td><code>state</code></td>
<td><p><code>enum ( </code><code>State</code><code> )</code></p>
<p>Output only. The current instance state. For <code>CreateInstance</code> , the state must be either omitted or set to <code>CREATING</code> . For <code>UpdateInstance</code> , the state must be either omitted or set to <code>READY</code> .</p></td>
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
<p>If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "_" + value would prove problematic if we were to allow "_" in a future release.</p>
<p>An object containing a list of <code>"key": value</code> pairs. Example: <code>{ "name": "wrench", "mass": "1.3kg", "count": "3" }</code> .</p></td>
</tr>
<tr class="even">
<td><code>instanceType</code></td>
<td><p><code>enum ( </code><code>InstanceType</code><code> )</code></p>
<p>The <code>InstanceType</code> of the current instance.</p></td>
</tr>
<tr class="odd">
<td><code>endpointUris[]</code></td>
<td><p><code>string</code></p>
<p>Deprecated. This field is not populated.</p></td>
</tr>
<tr class="even">
<td><code>createTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance was created.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="odd">
<td><code>updateTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance was most recently updated.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="even">
<td><code>freeInstanceMetadata</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.FreeInstanceMetadata"><code>FreeInstanceMetadata</code></a><code> )</code></p>
<p>Free instance metadata. Only populated for free instances.</p></td>
</tr>
<tr class="odd">
<td><code>edition</code></td>
<td><p><code>enum ( </code><code>Edition</code><code> )</code></p>
<p>Optional. The <code>Edition</code> of the current instance.</p></td>
</tr>
<tr class="even">
<td><code>defaultBackupScheduleType</code></td>
<td><p><code>enum ( </code><code>DefaultBackupScheduleType</code><code> )</code></p>
<p>Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.</p>
<p>Note that the <code>AUTOMATIC</code> value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.</p>
<p>In the <code>GetInstance</code> or <code>ListInstances</code> response, if the value of <code>default_backup_schedule_type</code> isn't set, or set to <code>NONE</code> , Spanner doesn't create a default backup schedule for new databases in the instance.</p></td>
</tr>
</tbody>
</table>

### ReplicaComputeCapacity

**JSON representation**

```
{
  "replicaSelection": {
    object (ReplicaSelection)
  },

  // Union field compute_capacity can be only one of the following:
  "nodeCount": integer,
  "processingUnits": integer
  // End of list of possible types for union field compute_capacity.
}
```

| Fields                                                                                                                                                                                                                                                                                                                                           |                                                                                                                                                                                                                                                                                                 |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replicaSelection`                                                                                                                                                                                                                                                                                                                               | `object ( `[`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.ReplicaSelection)` )` Required. Identifies replicas by specified properties. All replicas in the selection have the same amount of compute capacity. |
| Union field `compute_capacity` . Compute capacity allocated to each replica identified by the specified selection. The unit is selected based on the unit used to specify the instance size for non-autoscaling instances, or the unit used in autoscaling limit for autoscaling instances. `compute_capacity` can be only one of the following: |                                                                                                                                                                                                                                                                                                 |
| `nodeCount`                                                                                                                                                                                                                                                                                                                                      | `integer` The number of nodes allocated to each replica. This may be zero in API responses for instances that are not yet in state `READY` .                                                                                                                                                    |
| `processingUnits`                                                                                                                                                                                                                                                                                                                                | `integer` The number of processing units allocated to each replica. This may be zero in API responses for instances that are not yet in state `READY` .                                                                                                                                         |

### ReplicaSelection

**JSON representation**

```
{
  "location": string
}
```

| Fields     |                                                                                       |
|------------|---------------------------------------------------------------------------------------|
| `location` | `string` Required. Name of the location of the replicas (for example, "us-central1"). |

### AutoscalingConfig

**JSON representation**

```
{
  "autoscalingLimits": {
    object (AutoscalingLimits)
  },
  "autoscalingTargets": {
    object (AutoscalingTargets)
  },
  "asymmetricAutoscalingOptions": [
    {
      object (AsymmetricAutoscalingOption)
    }
  ]
}
```

| Fields                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
|----------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `autoscalingLimits`              | `object ( `[`AutoscalingLimits`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AutoscalingLimits)` )` Required. Autoscaling limits for an instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `autoscalingTargets`             | `object ( `[`AutoscalingTargets`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AutoscalingTargets)` )` Required. The autoscaling targets for an instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `asymmetricAutoscalingOptions[]` | `object ( `[`AsymmetricAutoscalingOption`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AsymmetricAutoscalingOption)` )` Optional. Optional asymmetric autoscaling options. Replicas matching the replica selection criteria will be autoscaled independently from other replicas. The autoscaler will scale the replicas based on the utilization of replicas identified by the replica selection. Replica selections should not overlap with each other. Other replicas (those do not match any replica selection) will be autoscaled together and will have the same compute capacity allocated to them. |

### AutoscalingLimits

**JSON representation**

```
{

  // Union field min_limit can be only one of the following:
  "minNodes": integer,
  "minProcessingUnits": integer
  // End of list of possible types for union field min_limit.

  // Union field max_limit can be only one of the following:
  "maxNodes": integer,
  "maxProcessingUnits": integer
  // End of list of possible types for union field max_limit.
}
```

| Fields                                                                                                                                                                                                                |                                                                                                                                                                               |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `min_limit` . The minimum compute capacity for the instance. `min_limit` can be only one of the following:                                                                                                |                                                                                                                                                                               |
| `minNodes`                                                                                                                                                                                                            | `integer` Minimum number of nodes allocated to the instance. If set, this number should be greater than or equal to 1.                                                        |
| `minProcessingUnits`                                                                                                                                                                                                  | `integer` Minimum number of processing units allocated to the instance. If set, this number should be multiples of 1000.                                                      |
| Union field `max_limit` . The maximum compute capacity for the instance. The maximum compute capacity should be less than or equal to 10X the minimum compute capacity. `max_limit` can be only one of the following: |                                                                                                                                                                               |
| `maxNodes`                                                                                                                                                                                                            | `integer` Maximum number of nodes allocated to the instance. If set, this number should be greater than or equal to min_nodes.                                                |
| `maxProcessingUnits`                                                                                                                                                                                                  | `integer` Maximum number of processing units allocated to the instance. If set, this number should be multiples of 1000 and be greater than or equal to min_processing_units. |

### AutoscalingTargets

**JSON representation**

```
{
  "highPriorityCpuUtilizationPercent": integer,
  "totalCpuUtilizationPercent": integer,
  "storageUtilizationPercent": integer
}
```

| Fields                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
|-------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `highPriorityCpuUtilizationPercent` | `integer` Optional. The target high priority cpu utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on high priority CPU utilization.                                                                                                                                                                                         |
| `totalCpuUtilizationPercent`        | `integer` Optional. The target total CPU utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on total CPU utilization. If both `high_priority_cpu_utilization_percent` and `total_cpu_utilization_percent` are specified, the autoscaler provisions the larger of the two required compute capacities to satisfy both targets. |
| `storageUtilizationPercent`         | `integer` Required. The target storage utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 99\] inclusive.                                                                                                                                                                                                                                                                                                      |

### AsymmetricAutoscalingOption

**JSON representation**

```
{
  "replicaSelection": {
    object (ReplicaSelection)
  },
  "overrides": {
    object (AutoscalingConfigOverrides)
  }
}
```

| Fields             |                                                                                                                                                                                                                                                                                           |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replicaSelection` | `object ( `[`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.ReplicaSelection)` )` Required. Selects the replicas to which this AsymmetricAutoscalingOption applies. Only read-only replicas are supported. |
| `overrides`        | `object ( `[`AutoscalingConfigOverrides`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AutoscalingConfigOverrides)` )` Optional. Overrides applied to the top-level autoscaling configuration for the selected replicas.    |

### AutoscalingConfigOverrides

**JSON representation**

```
{
  "autoscalingLimits": {
    object (AutoscalingLimits)
  },
  "autoscalingTargetHighPriorityCpuUtilizationPercent": integer,
  "autoscalingTargetTotalCpuUtilizationPercent": integer,
  "disableHighPriorityCpuAutoscaling": boolean,
  "disableTotalCpuAutoscaling": boolean
}
```

| Fields                                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `autoscalingLimits`                                  | `object ( `[`AutoscalingLimits`](https://docs.cloud.google.com/spanner/docs/reference/mcp/spanner/mcp/tools_list/get_instance#Output.Schema.AutoscalingLimits)` )` Optional. If specified, overrides the min/max limit in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `autoscalingTargetHighPriorityCpuUtilizationPercent` | `integer` Optional. If specified, overrides the autoscaling target high_priority_cpu_utilization_percent in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `autoscalingTargetTotalCpuUtilizationPercent`        | `integer` Optional. If specified, overrides the autoscaling target `total_cpu_utilization_percent` in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `disableHighPriorityCpuAutoscaling`                  | `boolean` Optional. If true, disables high priority CPU autoscaling for the selected replicas and ignores `high_priority_cpu_utilization_percent` in the top-level autoscaling configuration. When setting this field to true, setting `autoscaling_target_high_priority_cpu_utilization_percent` field to a non-zero value for the same replica is not supported. If false, the `autoscaling_target_high_priority_cpu_utilization_percent` field in the replica will be used if set to a non-zero value. Otherwise, the `high_priority_cpu_utilization_percent` field in the top-level autoscaling configuration will be used. Setting both `disable_high_priority_cpu_autoscaling` and `disable_total_cpu_autoscaling` to true for the same replica is not supported. |
| `disableTotalCpuAutoscaling`                         | `boolean` Optional. If true, disables total CPU autoscaling for the selected replicas and ignores `total_cpu_utilization_percent` in the top-level autoscaling configuration. When setting this field to true, setting `autoscaling_target_total_cpu_utilization_percent` field to a non-zero value for the same replica is not supported. If false, the `autoscaling_target_total_cpu_utilization_percent` field in the replica will be used if set to a non-zero value. Otherwise, the `total_cpu_utilization_percent` field in the top-level autoscaling configuration will be used. Setting both `disable_high_priority_cpu_autoscaling` and `disable_total_cpu_autoscaling` to true for the same replica is not supported.                                         |

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

### Timestamp

**JSON representation**

```
{
  "seconds": string,
  "nanos": integer
}
```

| Fields    |                                                                                                                                                                                                                                                                                                                      |
|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `seconds` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).                            |
| `nanos`   | `integer` Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive. |

### FreeInstanceMetadata

**JSON representation**

```
{
  "expireTime": string,
  "upgradeTime": string,
  "expireBehavior": enum (ExpireBehavior)
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `expireTime`     | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. Timestamp after which the instance will either be upgraded or scheduled for deletion after a grace period. ExpireBehavior is used to choose between upgrading or scheduling the free instance for deletion. This timestamp is set during the creation of a free instance. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `upgradeTime`    | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Output only. If present, the timestamp at which the free instance was upgraded to a provisioned instance. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                                              |
| `expireBehavior` | `enum ( ``ExpireBehavior`` )` Specifies the expiration behavior of a free instance. The default of ExpireBehavior is `REMOVE_AFTER_GRACE_PERIOD` . This can be modified during or after creation, and before expiration.                                                                                                                                                                                                                                                                                                                                                                                                                                     |

### Tool Annotations

Destructive Hint: ❌ \| Idempotent Hint: ✅ \| Read Only Hint: ✅ \| Open World Hint: ❌
