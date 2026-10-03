---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances
title: 'REST Resource: projects.instances'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [Resource: Instance](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance.SCHEMA_REPRESENTATION)
- [ReplicaComputeCapacity](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ReplicaComputeCapacity)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ReplicaComputeCapacity.SCHEMA_REPRESENTATION)
- [State](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#State)
- [InstanceType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#InstanceType)
- [FreeInstanceMetadata](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#FreeInstanceMetadata)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#FreeInstanceMetadata.SCHEMA_REPRESENTATION)
- [ExpireBehavior](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ExpireBehavior)
- [DefaultBackupScheduleType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#DefaultBackupScheduleType)
- [Methods](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#METHODS_SUMMARY)

## Resource: Instance

An isolated set of Cloud Spanner resources on which databases can be hosted.

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
<p>Required. The name of the instance's configuration. Values are of the form <code>projects/&lt;project&gt;/instanceConfigs/&lt;configuration&gt;</code> . See also <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig"><code>InstanceConfig</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigs"><code>ListInstanceConfigs</code></a> .</p></td>
</tr>
<tr class="odd">
<td><code>displayName</code></td>
<td><p><code>string</code></p>
<p>Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.</p></td>
</tr>
<tr class="even">
<td><code>nodeCount</code></td>
<td><p><code>integer</code></p>
<p>The number of nodes allocated to this instance. At most, one of either <code>nodeCount</code> or <code>processingUnits</code> should be present in the message.</p>
<p>Users can set the <code>nodeCount</code> field to specify the target number of nodes allocated to the instance.</p>
<p>If autoscaling is enabled, <code>nodeCount</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of nodes allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying node count across replicas (achieved by setting <code>asymmetricAutoscalingOptions</code> in the autoscaling configuration), the <code>nodeCount</code> set here is the maximum node count across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes, and processing units</a> .</p></td>
</tr>
<tr class="odd">
<td><code>processingUnits</code></td>
<td><p><code>integer</code></p>
<p>The number of processing units allocated to this instance. At most, one of either <code>processingUnits</code> or <code>nodeCount</code> should be present in the message.</p>
<p>Users can set the <code>processingUnits</code> field to specify the target number of processing units allocated to the instance.</p>
<p>If autoscaling is enabled, <code>processingUnits</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of processing units allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying processing units per replica (achieved by setting <code>asymmetricAutoscalingOptions</code> in the autoscaling configuration), the <code>processingUnits</code> set here is the maximum processing units across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes and processing units</a> .</p></td>
</tr>
<tr class="even">
<td><code>replicaComputeCapacity[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ReplicaComputeCapacity"><code>ReplicaComputeCapacity</code></a><code> )</code></p>
<p>Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.</p></td>
</tr>
<tr class="odd">
<td><code>autoscalingConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig"><code>AutoscalingConfig</code></a><code> )</code></p>
<p>Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, nodeCount and processingUnits are treated as OUTPUT_ONLY fields and reflect the current compute capacity allocated to the instance.</p></td>
</tr>
<tr class="even">
<td><code>state</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#State"><code>State</code></a><code> )</code></p>
<p>Output only. The current instance state. For <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstance"><code>instances.create</code></a> , the state must be either omitted or set to <code>CREATING</code> . For <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance"><code>instances.patch</code></a> , the state must be either omitted or set to <code>READY</code> .</p></td>
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
<p>If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "_" + value would prove problematic if we were to allow "_" in a future release.</p></td>
</tr>
<tr class="even">
<td><code>instanceType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#InstanceType"><code>InstanceType</code></a><code> )</code></p>
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
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#FreeInstanceMetadata"><code>FreeInstanceMetadata</code></a><code> )</code></p>
<p>Free instance metadata. Only populated for free instances.</p></td>
</tr>
<tr class="odd">
<td><code>edition</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Edition"><code>Edition</code></a><code> )</code></p>
<p>Optional. The <code>Edition</code> of the current instance.</p></td>
</tr>
<tr class="even">
<td><code>defaultBackupScheduleType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#DefaultBackupScheduleType"><code>DefaultBackupScheduleType</code></a><code> )</code></p>
<p>Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.</p>
<p>Note that the <code>AUTOMATIC</code> value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.</p>
<p>In the <code>instances.get</code> or <code>instances.list</code> response, if the value of <code>defaultBackupScheduleType</code> isn't set, or set to <code>NONE</code> , Spanner doesn't create a default backup schedule for new databases in the instance.</p></td>
</tr>
</tbody>
</table>

## ReplicaComputeCapacity

ReplicaComputeCapacity describes the amount of server resources that are allocated to each replica identified by the replica selection.

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

| Fields                                                                                                                                                                                                                                                                                                                                           |                                                                                                                                                                                                                                                   |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `replicaSelection`                                                                                                                                                                                                                                                                                                                               | `object ( `[`ReplicaSelection`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ReplicaSelection)` )` Required. Identifies replicas by specified properties. All replicas in the selection have the same amount of compute capacity. |
| Union field `compute_capacity` . Compute capacity allocated to each replica identified by the specified selection. The unit is selected based on the unit used to specify the instance size for non-autoscaling instances, or the unit used in autoscaling limit for autoscaling instances. `compute_capacity` can be only one of the following: |                                                                                                                                                                                                                                                   |
| `nodeCount`                                                                                                                                                                                                                                                                                                                                      | `integer` The number of nodes allocated to each replica. This may be zero in API responses for instances that are not yet in state `READY` .                                                                                                      |
| `processingUnits`                                                                                                                                                                                                                                                                                                                                | `integer` The number of processing units allocated to each replica. This may be zero in API responses for instances that are not yet in state `READY` .                                                                                           |

## State

Indicates the current state of the instance.

| Enums               |                                                                                                                                 |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------|
| `STATE_UNSPECIFIED` | Not specified.                                                                                                                  |
| `CREATING`          | The instance is still being created. Resources may not be available yet, and operations such as database creation may not work. |
| `READY`             | The instance is fully created and ready to do work such as creating databases.                                                  |

## InstanceType

The type of this instance. The type can be used to distinguish product variants, that can affect aspects like: usage restrictions, quotas and billing. Currently this is used to distinguish FREE_INSTANCE vs PROVISIONED instances.

| Enums                       |                                                                                                                                                                    |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `INSTANCE_TYPE_UNSPECIFIED` | Not specified.                                                                                                                                                     |
| `PROVISIONED`               | Provisioned instances have dedicated resources, standard usage limits and support.                                                                                 |
| `FREE_INSTANCE`             | Free instances provide no guarantee for dedicated resources, \[nodeCount, processingUnits\] should be 0. They come with stricter usage limits and limited support. |

## FreeInstanceMetadata

Free instance specific metadata that is kept even after an instance has been upgraded for tracking purposes.

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
| `expireBehavior` | `enum ( `[`ExpireBehavior`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ExpireBehavior)` )` Specifies the expiration behavior of a free instance. The default of ExpireBehavior is `REMOVE_AFTER_GRACE_PERIOD` . This can be modified during or after creation, and before expiration.                                                                                                                                                                                                                                                                                                                                   |

## ExpireBehavior

Allows users to change behavior when a free instance expires.

| Enums                         |                                                                                                                                |
|-------------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| `EXPIRE_BEHAVIOR_UNSPECIFIED` | Not specified.                                                                                                                 |
| `FREE_TO_PROVISIONED`         | When the free instance expires, upgrade the instance to a provisioned instance.                                                |
| `REMOVE_AFTER_GRACE_PERIOD`   | When the free instance expires, disable the instance, and delete it after the grace period passes if it has not been upgraded. |

## DefaultBackupScheduleType

Indicates the [default backup schedule](https://cloud.google.com/spanner/docs/backup#default-backup-schedules) behavior for new databases within the instance.

| Enums                                      |                                                                                                                                                                                                                                                                                        |
|--------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DEFAULT_BACKUP_SCHEDULE_TYPE_UNSPECIFIED` | Not specified.                                                                                                                                                                                                                                                                         |
| `NONE`                                     | A default backup schedule isn't created automatically when a new database is created in the instance.                                                                                                                                                                                  |
| `AUTOMATIC`                                | A default backup schedule is created automatically when a new database is created in the instance. The default backup schedule creates a full backup every 24 hours. These full backups are retained for 7 days. You can edit or delete the default backup schedule once it's created. |

| Methods                                                                                                                    |                                                                                 |
|----------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------|
| [`create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/create)                         | Creates an instance and begins preparing it to begin serving.                   |
| [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/delete)                         | Deletes an instance.                                                            |
| [`get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/get)                               | Gets information about a particular instance.                                   |
| [`getIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/getIamPolicy)             | Gets the access control policy for an instance resource.                        |
| [`list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list)                             | Lists all instances in the given project.                                       |
| [`move`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move)                             | Moves an instance to the target instance configuration.                         |
| [`patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch)                           | Updates an instance, and begins allocating or releasing resources as requested. |
| [`setIamPolicy`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/setIamPolicy)             | Sets the access control policy on an instance resource.                         |
| [`testIamPermissions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/testIamPermissions) | Returns permissions that the caller has on the specified instance resource.     |
