## Tool: `       create_instance      `

Create a Spanner instance in a given project.

The following sample demonstrate how to use `  curl  ` to invoke the `  create_instance  ` MCP tool.

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
    &quot;name&quot;: &quot;create_instance&quot;,
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

The request for `  CreateInstance  ` .

### CreateInstanceRequest

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
  &quot;instanceId&quot;: string,
  &quot;instance&quot;: {
    object (Instance)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  parent  `

`  string  `

Required. The name of the project in which to create the instance. Values are of the form `  projects/<project>  ` .

`  instanceId  `

`  string  `

Required. The ID of the instance to create. Valid identifiers are of the form `  [a-z][-a-z0-9]*[a-z0-9]  ` and must be between 2 and 64 characters in length.

`  instance  `

`  object ( Instance  ` )

Required. The instance to create. The name may be omitted, but if specified must be `  <parent>/instances/<instance_id>  ` .

### Instance

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
  &quot;config&quot;: string,
  &quot;displayName&quot;: string,
  &quot;nodeCount&quot;: integer,
  &quot;processingUnits&quot;: integer,
  &quot;replicaComputeCapacity&quot;: [
    {
      object (ReplicaComputeCapacity)
    }
  ],
  &quot;autoscalingConfig&quot;: {
    object (AutoscalingConfig)
  },
  &quot;state&quot;: enum (State),
  &quot;labels&quot;: {
    string: string,
    ...
  },
  &quot;instanceType&quot;: enum (InstanceType),
  &quot;endpointUris&quot;: [
    string
  ],
  &quot;createTime&quot;: string,
  &quot;updateTime&quot;: string,
  &quot;freeInstanceMetadata&quot;: {
    object (FreeInstanceMetadata)
  },
  &quot;edition&quot;: enum (Edition),
  &quot;defaultBackupScheduleType&quot;: enum (DefaultBackupScheduleType)
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  name  `

`  string  `

Required. A unique identifier for the instance, which cannot be changed after the instance is created. Values are of the form `  projects/<project>/instances/[a-z][-a-z0-9]*[a-z0-9]  ` . The final segment of the name must be between 2 and 64 characters in length.

`  config  `

`  string  `

Required. The name of the instance's configuration. Values are of the form `  projects/<project>/instanceConfigs/<configuration>  ` . See also `  InstanceConfig  ` and `  ListInstanceConfigs  ` .

`  displayName  `

`  string  `

Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.

`  nodeCount  `

`  integer  `

The number of nodes allocated to this instance. At most, one of either `  node_count  ` or `  processing_units  ` should be present in the message.

Users can set the `  node_count  ` field to specify the target number of nodes allocated to the instance.

If autoscaling is enabled, `  node_count  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of nodes allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying node count across replicas (achieved by setting `  asymmetric_autoscaling_options  ` in the autoscaling configuration), the `  node_count  ` set here is the maximum node count across all replicas.

For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  processingUnits  `

`  integer  `

The number of processing units allocated to this instance. At most, one of either `  processing_units  ` or `  node_count  ` should be present in the message.

Users can set the `  processing_units  ` field to specify the target number of processing units allocated to the instance.

If autoscaling is enabled, `  processing_units  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of processing units allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying processing units per replica (achieved by setting `  asymmetric_autoscaling_options  ` in the autoscaling configuration), the `  processing_units  ` set here is the maximum processing units across all replicas.

For more information, see [Compute capacity, nodes and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  replicaComputeCapacity[]  `

`  object ( ReplicaComputeCapacity  ` )

Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.

`  autoscalingConfig  `

`  object ( AutoscalingConfig  ` )

Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, node\_count and processing\_units are treated as OUTPUT\_ONLY fields and reflect the current compute capacity allocated to the instance.

`  state  `

`  enum ( State  ` )

Output only. The current instance state. For `  CreateInstance  ` , the state must be either omitted or set to `  CREATING  ` . For `  UpdateInstance  ` , the state must be either omitted or set to `  READY  ` .

`  labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

An object containing a list of `  "key": value  ` pairs. Example: `  { "name": "wrench", "mass": "1.3kg", "count": "3" }  ` .

`  instanceType  `

`  enum ( InstanceType  ` )

The `  InstanceType  ` of the current instance.

`  endpointUris[]  `

`  string  `

Deprecated. This field is not populated.

`  createTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance was created.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  updateTime  `

`  string ( Timestamp  ` format)

Output only. The time at which the instance was most recently updated.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  freeInstanceMetadata  `

`  object ( FreeInstanceMetadata  ` )

Free instance metadata. Only populated for free instances.

`  edition  `

`  enum ( Edition  ` )

Optional. The `  Edition  ` of the current instance.

`  defaultBackupScheduleType  `

`  enum ( DefaultBackupScheduleType  ` )

Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.

Note that the `  AUTOMATIC  ` value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.

In the `  GetInstance  ` or `  ListInstances  ` response, if the value of `  default_backup_schedule_type  ` isn't set, or set to `  NONE  ` , Spanner doesn't create a default backup schedule for new databases in the instance.

### ReplicaComputeCapacity

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
  &quot;replicaSelection&quot;: {
    object (ReplicaSelection)
  },

  // Union field compute_capacity can be only one of the following:
  &quot;nodeCount&quot;: integer,
  &quot;processingUnits&quot;: integer
  // End of list of possible types for union field compute_capacity.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  replicaSelection  `

`  object ( ReplicaSelection  ` )

Required. Identifies replicas by specified properties. All replicas in the selection have the same amount of compute capacity.

Union field `  compute_capacity  ` . Compute capacity allocated to each replica identified by the specified selection. The unit is selected based on the unit used to specify the instance size for non-autoscaling instances, or the unit used in autoscaling limit for autoscaling instances. `  compute_capacity  ` can be only one of the following:

`  nodeCount  `

`  integer  `

The number of nodes allocated to each replica.

This may be zero in API responses for instances that are not yet in state `  READY  ` .

`  processingUnits  `

`  integer  `

The number of processing units allocated to each replica.

This may be zero in API responses for instances that are not yet in state `  READY  ` .

### ReplicaSelection

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
  &quot;location&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  location  `

`  string  `

Required. Name of the location of the replicas (for example, "us-central1").

### AutoscalingConfig

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
  &quot;autoscalingLimits&quot;: {
    object (AutoscalingLimits)
  },
  &quot;autoscalingTargets&quot;: {
    object (AutoscalingTargets)
  },
  &quot;asymmetricAutoscalingOptions&quot;: [
    {
      object (AsymmetricAutoscalingOption)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  autoscalingLimits  `

`  object ( AutoscalingLimits  ` )

Required. Autoscaling limits for an instance.

`  autoscalingTargets  `

`  object ( AutoscalingTargets  ` )

Required. The autoscaling targets for an instance.

`  asymmetricAutoscalingOptions[]  `

`  object ( AsymmetricAutoscalingOption  ` )

Optional. Optional asymmetric autoscaling options. Replicas matching the replica selection criteria will be autoscaled independently from other replicas. The autoscaler will scale the replicas based on the utilization of replicas identified by the replica selection. Replica selections should not overlap with each other.

Other replicas (those do not match any replica selection) will be autoscaled together and will have the same compute capacity allocated to them.

### AutoscalingLimits

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

  // Union field min_limit can be only one of the following:
  &quot;minNodes&quot;: integer,
  &quot;minProcessingUnits&quot;: integer
  // End of list of possible types for union field min_limit.

  // Union field max_limit can be only one of the following:
  &quot;maxNodes&quot;: integer,
  &quot;maxProcessingUnits&quot;: integer
  // End of list of possible types for union field max_limit.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `  min_limit  ` . The minimum compute capacity for the instance. `  min_limit  ` can be only one of the following:

`  minNodes  `

`  integer  `

Minimum number of nodes allocated to the instance. If set, this number should be greater than or equal to 1.

`  minProcessingUnits  `

`  integer  `

Minimum number of processing units allocated to the instance. If set, this number should be multiples of 1000.

Union field `  max_limit  ` . The maximum compute capacity for the instance. The maximum compute capacity should be less than or equal to 10X the minimum compute capacity. `  max_limit  ` can be only one of the following:

`  maxNodes  `

`  integer  `

Maximum number of nodes allocated to the instance. If set, this number should be greater than or equal to min\_nodes.

`  maxProcessingUnits  `

`  integer  `

Maximum number of processing units allocated to the instance. If set, this number should be multiples of 1000 and be greater than or equal to min\_processing\_units.

### AutoscalingTargets

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
  &quot;highPriorityCpuUtilizationPercent&quot;: integer,
  &quot;totalCpuUtilizationPercent&quot;: integer,
  &quot;storageUtilizationPercent&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  highPriorityCpuUtilizationPercent  `

`  integer  `

Optional. The target high priority cpu utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on high priority CPU utilization.

`  totalCpuUtilizationPercent  `

`  integer  `

Optional. The target total CPU utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on total CPU utilization. If both `  high_priority_cpu_utilization_percent  ` and `  total_cpu_utilization_percent  ` are specified, the autoscaler provisions the larger of the two required compute capacities to satisfy both targets.

`  storageUtilizationPercent  `

`  integer  `

Required. The target storage utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 99\] inclusive.

### AsymmetricAutoscalingOption

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
  &quot;replicaSelection&quot;: {
    object (ReplicaSelection)
  },
  &quot;overrides&quot;: {
    object (AutoscalingConfigOverrides)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  replicaSelection  `

`  object ( ReplicaSelection  ` )

Required. Selects the replicas to which this AsymmetricAutoscalingOption applies. Only read-only replicas are supported.

`  overrides  `

`  object ( AutoscalingConfigOverrides  ` )

Optional. Overrides applied to the top-level autoscaling configuration for the selected replicas.

### AutoscalingConfigOverrides

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
  &quot;autoscalingLimits&quot;: {
    object (AutoscalingLimits)
  },
  &quot;autoscalingTargetHighPriorityCpuUtilizationPercent&quot;: integer,
  &quot;autoscalingTargetTotalCpuUtilizationPercent&quot;: integer,
  &quot;disableHighPriorityCpuAutoscaling&quot;: boolean,
  &quot;disableTotalCpuAutoscaling&quot;: boolean
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  autoscalingLimits  `

`  object ( AutoscalingLimits  ` )

Optional. If specified, overrides the min/max limit in the top-level autoscaling configuration for the selected replicas.

`  autoscalingTargetHighPriorityCpuUtilizationPercent  `

`  integer  `

Optional. If specified, overrides the autoscaling target high\_priority\_cpu\_utilization\_percent in the top-level autoscaling configuration for the selected replicas.

`  autoscalingTargetTotalCpuUtilizationPercent  `

`  integer  `

Optional. If specified, overrides the autoscaling target `  total_cpu_utilization_percent  ` in the top-level autoscaling configuration for the selected replicas.

`  disableHighPriorityCpuAutoscaling  `

`  boolean  `

Optional. If true, disables high priority CPU autoscaling for the selected replicas and ignores `  high_priority_cpu_utilization_percent  ` in the top-level autoscaling configuration.

When setting this field to true, setting `  autoscaling_target_high_priority_cpu_utilization_percent  ` field to a non-zero value for the same replica is not supported.

If false, the `  autoscaling_target_high_priority_cpu_utilization_percent  ` field in the replica will be used if set to a non-zero value. Otherwise, the `  high_priority_cpu_utilization_percent  ` field in the top-level autoscaling configuration will be used.

Setting both `  disable_high_priority_cpu_autoscaling  ` and `  disable_total_cpu_autoscaling  ` to true for the same replica is not supported.

`  disableTotalCpuAutoscaling  `

`  boolean  `

Optional. If true, disables total CPU autoscaling for the selected replicas and ignores `  total_cpu_utilization_percent  ` in the top-level autoscaling configuration.

When setting this field to true, setting `  autoscaling_target_total_cpu_utilization_percent  ` field to a non-zero value for the same replica is not supported.

If false, the `  autoscaling_target_total_cpu_utilization_percent  ` field in the replica will be used if set to a non-zero value. Otherwise, the `  total_cpu_utilization_percent  ` field in the top-level autoscaling configuration will be used.

Setting both `  disable_high_priority_cpu_autoscaling  ` and `  disable_total_cpu_autoscaling  ` to true for the same replica is not supported.

### LabelsEntry

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
  &quot;key&quot;: string,
  &quot;value&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  key  `

`  string  `

`  value  `

`  string  `

### Timestamp

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
  &quot;seconds&quot;: string,
  &quot;nanos&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  seconds  `

`  string ( int64 format)  `

Represents seconds of UTC time since Unix epoch 1970-01-01T00:00:00Z. Must be between -62135596800 and 253402300799 inclusive (which corresponds to 0001-01-01T00:00:00Z to 9999-12-31T23:59:59Z).

`  nanos  `

`  integer  `

Non-negative fractions of a second at nanosecond resolution. This field is the nanosecond portion of the duration, not an alternative to seconds. Negative second values with fractions must still have non-negative nanos values that count forward in time. Must be between 0 and 999,999,999 inclusive.

### FreeInstanceMetadata

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
  &quot;expireTime&quot;: string,
  &quot;upgradeTime&quot;: string,
  &quot;expireBehavior&quot;: enum (ExpireBehavior)
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  expireTime  `

`  string ( Timestamp  ` format)

Output only. Timestamp after which the instance will either be upgraded or scheduled for deletion after a grace period. ExpireBehavior is used to choose between upgrading or scheduling the free instance for deletion. This timestamp is set during the creation of a free instance.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  upgradeTime  `

`  string ( Timestamp  ` format)

Output only. If present, the timestamp at which the free instance was upgraded to a provisioned instance.

Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `  "2014-10-02T15:01:23Z"  ` , `  "2014-10-02T15:01:23.045123456Z"  ` or `  "2014-10-02T15:01:23+05:30"  ` .

`  expireBehavior  `

`  enum ( ExpireBehavior  ` )

Specifies the expiration behavior of a free instance. The default of ExpireBehavior is `  REMOVE_AFTER_GRACE_PERIOD  ` . This can be modified during or after creation, and before expiration.

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
