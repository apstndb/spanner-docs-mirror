---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig
title: AutoscalingConfig
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

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

| Fields                           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `autoscalingLimits`              | `object ( `[`AutoscalingLimits`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingLimits)` )` Required. Autoscaling limits for an instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `autoscalingTargets`             | `object ( `[`AutoscalingTargets`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets)` )` Required. The autoscaling targets for an instance.                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `asymmetricAutoscalingOptions[]` | `object ( `[`AsymmetricAutoscalingOption`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AsymmetricAutoscalingOption)` )` Optional. Optional asymmetric autoscaling options. Replicas matching the replica selection criteria will be autoscaled independently from other replicas. The autoscaler will scale the replicas based on the utilization of replicas identified by the replica selection. Replica selections should not overlap with each other. Other replicas (those do not match any replica selection) will be autoscaled together and will have the same compute capacity allocated to them. |

## AutoscalingLimits

The autoscaling limits for the instance. Users can define the minimum and maximum compute capacity allocated to the instance, and the autoscaler will only scale within that range. Users can either use nodes or processing units to specify the limits, but should use the same unit to set both the min_limit and maxLimit.

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

| Fields                                                                                                                                                                                                                |                                                                                                                                                                             |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `min_limit` . The minimum compute capacity for the instance. `min_limit` can be only one of the following:                                                                                                |                                                                                                                                                                             |
| `minNodes`                                                                                                                                                                                                            | `integer` Minimum number of nodes allocated to the instance. If set, this number should be greater than or equal to 1.                                                      |
| `minProcessingUnits`                                                                                                                                                                                                  | `integer` Minimum number of processing units allocated to the instance. If set, this number should be multiples of 1000.                                                    |
| Union field `max_limit` . The maximum compute capacity for the instance. The maximum compute capacity should be less than or equal to 10X the minimum compute capacity. `max_limit` can be only one of the following: |                                                                                                                                                                             |
| `maxNodes`                                                                                                                                                                                                            | `integer` Maximum number of nodes allocated to the instance. If set, this number should be greater than or equal to minNodes.                                               |
| `maxProcessingUnits`                                                                                                                                                                                                  | `integer` Maximum number of processing units allocated to the instance. If set, this number should be multiples of 1000 and be greater than or equal to minProcessingUnits. |

## AutoscalingTargets

The autoscaling targets for an instance.

**JSON representation**

```
{
  "highPriorityCpuUtilizationPercent": integer,
  "totalCpuUtilizationPercent": integer,
  "storageUtilizationPercent": integer
}
```

| Fields                              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|-------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `highPriorityCpuUtilizationPercent` | `integer` Optional. The target high priority cpu utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on high priority CPU utilization.                                                                                                                                                                                  |
| `totalCpuUtilizationPercent`        | `integer` Optional. The target total CPU utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 90\] inclusive. If not specified or set to 0, the autoscaler skips scaling based on total CPU utilization. If both `highPriorityCpuUtilizationPercent` and `totalCpuUtilizationPercent` are specified, the autoscaler provisions the larger of the two required compute capacities to satisfy both targets. |
| `storageUtilizationPercent`         | `integer` Required. The target storage utilization percentage that the autoscaler should be trying to achieve for the instance. This number is on a scale from 0 (no utilization) to 100 (full utilization). The valid range is \[10, 99\] inclusive.                                                                                                                                                                                                                                                                                               |

## AsymmetricAutoscalingOption

AsymmetricAutoscalingOption specifies the scaling of replicas identified by the given selection.

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

| Fields             |                                                                                                                                                                                                                                                            |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replicaSelection` | `object ( `[`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ReplicaSelection)` )` Required. Selects the replicas to which this AsymmetricAutoscalingOption applies. Only read-only replicas are supported.                |
| `overrides`        | `object ( `[`AutoscalingConfigOverrides`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides)` )` Optional. Overrides applied to the top-level autoscaling configuration for the selected replicas. |

## AutoscalingConfigOverrides

Overrides the top-level autoscaling configuration for the replicas identified by `replicaSelection` . All fields in this message are optional. Any unspecified fields will use the corresponding values from the top-level autoscaling configuration.

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

| Fields                                               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `autoscalingLimits`                                  | `object ( `[`AutoscalingLimits`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingLimits)` )` Optional. If specified, overrides the min/max limit in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `autoscalingTargetHighPriorityCpuUtilizationPercent` | `integer` Optional. If specified, overrides the autoscaling target highPriorityCpuUtilizationPercent in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `autoscalingTargetTotalCpuUtilizationPercent`        | `integer` Optional. If specified, overrides the autoscaling target `totalCpuUtilizationPercent` in the top-level autoscaling configuration for the selected replicas.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `disableHighPriorityCpuAutoscaling`                  | `boolean` Optional. If true, disables high priority CPU autoscaling for the selected replicas and ignores [`highPriorityCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets.FIELDS.high_priority_cpu_utilization_percent) in the top-level autoscaling configuration. When setting this field to true, setting [`autoscalingTargetHighPriorityCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.autoscaling_target_high_priority_cpu_utilization_percent) field to a non-zero value for the same replica is not supported. If false, the [`autoscalingTargetHighPriorityCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.autoscaling_target_high_priority_cpu_utilization_percent) field in the replica will be used if set to a non-zero value. Otherwise, the [`highPriorityCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets.FIELDS.high_priority_cpu_utilization_percent) field in the top-level autoscaling configuration will be used. Setting both [`disableHighPriorityCpuAutoscaling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.disable_high_priority_cpu_autoscaling) and [`disableTotalCpuAutoscaling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.disable_total_cpu_autoscaling) to true for the same replica is not supported. |
| `disableTotalCpuAutoscaling`                         | `boolean` Optional. If true, disables total CPU autoscaling for the selected replicas and ignores [`totalCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets.FIELDS.total_cpu_utilization_percent) in the top-level autoscaling configuration. When setting this field to true, setting [`autoscalingTargetTotalCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.autoscaling_target_total_cpu_utilization_percent) field to a non-zero value for the same replica is not supported. If false, the [`autoscalingTargetTotalCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.autoscaling_target_total_cpu_utilization_percent) field in the replica will be used if set to a non-zero value. Otherwise, the [`totalCpuUtilizationPercent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingTargets.FIELDS.total_cpu_utilization_percent) field in the top-level autoscaling configuration will be used. Setting both [`disableHighPriorityCpuAutoscaling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.disable_high_priority_cpu_autoscaling) and [`disableTotalCpuAutoscaling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig#AutoscalingConfigOverrides.FIELDS.disable_total_cpu_autoscaling) to true for the same replica is not supported.                                                                     |
