  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#SCHEMA_REPRESENTATION)
  - [AutoscalingLimits](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingLimits)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingLimits.SCHEMA_REPRESENTATION)
  - [AutoscalingTargets](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets.SCHEMA_REPRESENTATION)
  - [AsymmetricAutoscalingOption](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AsymmetricAutoscalingOption)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AsymmetricAutoscalingOption.SCHEMA_REPRESENTATION)
  - [AutoscalingConfigOverrides](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides)
      - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.SCHEMA_REPRESENTATION)

Autoscaling configuration for an instance.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;autoscalingLimits&quot;: {object (AutoscalingLimits)},&quot;autoscalingTargets&quot;: {object (AutoscalingTargets)},&quot;asymmetricAutoscalingOptions&quot;: [{object (AsymmetricAutoscalingOption)}]}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`autoscalingLimits`

` object ( AutoscalingLimits  ` )

Required. Autoscaling limits for an instance.

`autoscalingTargets`

` object ( AutoscalingTargets  ` )

Required. The autoscaling targets for an instance.

`asymmetricAutoscalingOptions[]`

` object ( AsymmetricAutoscalingOption  ` )

Optional. Optional asymmetric autoscaling options. Replicas matching the replica selection criteria will be autoscaled independently from other replicas. The autoscaler will scale the replicas based on the utilization of replicas identified by the replica selection. Replica selections should not overlap with each other.

Other replicas (those do not match any replica selection) will be autoscaled together and will have the same compute capacity allocated to them.

## AutoscalingLimits

The autoscaling limits for the instance. Users can define the minimum and maximum compute capacity allocated to the instance, and the autoscaler will only scale within that range. Users can either use nodes or processing units to specify the limits, but should use the same unit to set both the min\_limit and maxLimit.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{// Union field min_limit can be only one of the following:&quot;minNodes&quot;: integer,&quot;minProcessingUnits&quot;: integer// End of list of possible types for union field min_limit.// Union field max_limit can be only one of the following:&quot;maxNodes&quot;: integer,&quot;maxProcessingUnits&quot;: integer// End of list of possible types for union field max_limit.}</code></pre></td>
</tr>
</tbody>
</table>

Fields

Union field `min_limit` . The minimum compute capacity for the instance. `min_limit` can be only one of the following:

`minNodes`

`integer`

Minimum number of nodes allocated to the instance. If set, this number should be greater than or equal to 1.

`minProcessingUnits`

`integer`

Minimum number of processing units allocated to the instance. If set, this number should be multiples of 1000.

Union field `max_limit` . The maximum compute capacity for the instance. The maximum compute capacity should be less than or equal to 10X the minimum compute capacity. `max_limit` can be only one of the following:

`maxNodes`

`integer`

Maximum number of nodes allocated to the instance. If set, this number should be greater than or equal to minNodes.

`maxProcessingUnits`

`integer`

Maximum number of processing units allocated to the instance. If set, this number should be multiples of 1000 and be greater than or equal to minProcessingUnits.

## AutoscalingTargets

The autoscaling targets for an instance.

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
  &quot;highPriorityCpuUtilizationPercent&quot;: integer,
  &quot;totalCpuUtilizationPercent&quot;: integer,
  &quot;storageUtilizationPercent&quot;: integer
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`highPriorityCpuUtilizationPercent`

`integer`

Optional. The target high priority cpu utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on high priority CPU utilization.

`totalCpuUtilizationPercent`

`integer`

Optional. The target total CPU utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on total CPU utilization. If both `highPriorityCpuUtilizationPercent` and `totalCpuUtilizationPercent` are specified, the autoscaler provisions the larger of the two required compute capacities to satisfy both targets.

`storageUtilizationPercent`

`integer`

Required. The target storage utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 99\] inclusive.

## AsymmetricAutoscalingOption

AsymmetricAutoscalingOption specifies the scaling of replicas identified by the given selection.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;replicaSelection&quot;: {object (ReplicaSelection)},&quot;overrides&quot;: {object (AutoscalingConfigOverrides)}}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`replicaSelection`

` object ( ReplicaSelection  ` )

Required. Selects the replicas to which this AsymmetricAutoscalingOption applies. Only read-only replicas are supported.

`overrides`

` object ( AutoscalingConfigOverrides  ` )

Optional. Overrides applied to the top-level autoscaling configuration for the selected replicas.

## AutoscalingConfigOverrides

Overrides the top-level autoscaling configuration for the replicas identified by `replicaSelection` . All fields in this message are optional. Any unspecified fields will use the corresponding values from the top-level autoscaling configuration.

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
<td><pre dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{&quot;autoscalingLimits&quot;: {object (AutoscalingLimits)},&quot;autoscalingTargetHighPriorityCpuUtilizationPercent&quot;: integer,&quot;autoscalingTargetTotalCpuUtilizationPercent&quot;: integer,&quot;disableHighPriorityCpuAutoscaling&quot;: boolean,&quot;disableTotalCpuAutoscaling&quot;: boolean}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`autoscalingLimits`

` object ( AutoscalingLimits  ` )

Optional. If specified, overrides the min/max limit in the top-level autoscaling configuration for the selected replicas.

`autoscalingTargetHighPriorityCpuUtilizationPercent`

`integer`

Optional. If specified, overrides the autoscaling target highPriorityCpuUtilizationPercent in the top-level autoscaling configuration for the selected replicas.

`autoscalingTargetTotalCpuUtilizationPercent`

`integer`

Optional. If specified, overrides the autoscaling target `totalCpuUtilizationPercent` in the top-level autoscaling configuration for the selected replicas.

`disableHighPriorityCpuAutoscaling`

`boolean`

Optional. If true, disables high priority CPU autoscaling for the selected replicas and ignores `  highPriorityCpuUtilizationPercent  ` in the top-level autoscaling configuration.

When setting this field to true, setting `  autoscalingTargetHighPriorityCpuUtilizationPercent  ` field to a non-zero value for the same replica is not supported.

If false, the `  autoscalingTargetHighPriorityCpuUtilizationPercent  ` field in the replica will be used if set to a non-zero value. Otherwise, the `  highPriorityCpuUtilizationPercent  ` field in the top-level autoscaling configuration will be used.

Setting both `  disableHighPriorityCpuAutoscaling  ` and `  disableTotalCpuAutoscaling  ` to true for the same replica is not supported.

`disableTotalCpuAutoscaling`

`boolean`

Optional. If true, disables total CPU autoscaling for the selected replicas and ignores `  totalCpuUtilizationPercent  ` in the top-level autoscaling configuration.

When setting this field to true, setting `  autoscalingTargetTotalCpuUtilizationPercent  ` field to a non-zero value for the same replica is not supported.

If false, the `  autoscalingTargetTotalCpuUtilizationPercent  ` field in the replica will be used if set to a non-zero value. Otherwise, the `  totalCpuUtilizationPercent  ` field in the top-level autoscaling configuration will be used.

Setting both `  disableHighPriorityCpuAutoscaling  ` and `  disableTotalCpuAutoscaling  ` to true for the same replica is not supported.
