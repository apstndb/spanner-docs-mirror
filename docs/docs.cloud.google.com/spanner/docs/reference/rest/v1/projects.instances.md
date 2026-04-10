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

The number of nodes allocated to this instance. At most, one of either `  nodeCount  ` or `  processingUnits  ` should be present in the message.

Users can set the `  nodeCount  ` field to specify the target number of nodes allocated to the instance.

If autoscaling is enabled, `  nodeCount  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of nodes allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying node count across replicas (achieved by setting `  asymmetricAutoscalingOptions  ` in the autoscaling configuration), the `  nodeCount  ` set here is the maximum node count across all replicas.

For more information, see [Compute capacity, nodes, and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  processingUnits  `

`  integer  `

The number of processing units allocated to this instance. At most, one of either `  processingUnits  ` or `  nodeCount  ` should be present in the message.

Users can set the `  processingUnits  ` field to specify the target number of processing units allocated to the instance.

If autoscaling is enabled, `  processingUnits  ` is treated as an `  OUTPUT_ONLY  ` field and reflects the current number of processing units allocated to the instance.

This might be zero in API responses for instances that are not yet in the `  READY  ` state.

If the instance has varying processing units per replica (achieved by setting `  asymmetricAutoscalingOptions  ` in the autoscaling configuration), the `  processingUnits  ` set here is the maximum processing units across all replicas.

For more information, see [Compute capacity, nodes and processing units](https://cloud.google.com/spanner/docs/compute-capacity) .

`  replicaComputeCapacity[]  `

`  object ( ReplicaComputeCapacity  ` )

Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.

`  autoscalingConfig  `

`  object ( AutoscalingConfig  ` )

Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, nodeCount and processingUnits are treated as OUTPUT\_ONLY fields and reflect the current compute capacity allocated to the instance.

`  state  `

`  enum ( State  ` )

Output only. The current instance state. For `  instances.create  ` , the state must be either omitted or set to `  CREATING  ` . For `  instances.patch  ` , the state must be either omitted or set to `  READY  ` .

`  labels  `

`  map (key: string, value: string)  `

Cloud Labels are a flexible and lightweight mechanism for organizing cloud resources into groups that reflect a customer's organizational needs and deployment strategies. Cloud Labels can be used to filter collections of resources. They can be used to control how resource metrics are aggregated. And they can be used as arguments to policy management rules (e.g. route, firewall, load balancing, etc.).

  - Label keys must be between 1 and 63 characters long and must conform to the following regular expression: `  [a-z][a-z0-9_-]{0,62}  ` .
  - Label values must be between 0 and 63 characters long and must conform to the regular expression `  [a-z0-9_-]{0,63}  ` .
  - No more than 64 labels can be associated with a given resource.

See <https://goo.gl/xmQnxf> for more information on and examples of labels.

If you plan to use labels in your own code, please note that additional characters may be allowed in the future. And so you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "\_" + value would prove problematic if we were to allow "\_" in a future release.

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

In the `  instances.get  ` or `  instances.list  ` response, if the value of `  defaultBackupScheduleType  ` isn't set, or set to `  NONE  ` , Spanner doesn't create a default backup schedule for new databases in the instance.

## ReplicaComputeCapacity

ReplicaComputeCapacity describes the amount of server resources that are allocated to each replica identified by the replica selection.

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

## State

Indicates the current state of the instance.

Enums

`  STATE_UNSPECIFIED  `

Not specified.

`  CREATING  `

The instance is still being created. Resources may not be available yet, and operations such as database creation may not work.

`  READY  `

The instance is fully created and ready to do work such as creating databases.

## InstanceType

The type of this instance. The type can be used to distinguish product variants, that can affect aspects like: usage restrictions, quotas and billing. Currently this is used to distinguish FREE\_INSTANCE vs PROVISIONED instances.

Enums

`  INSTANCE_TYPE_UNSPECIFIED  `

Not specified.

`  PROVISIONED  `

Provisioned instances have dedicated resources, standard usage limits and support.

`  FREE_INSTANCE  `

Free instances provide no guarantee for dedicated resources, \[nodeCount, processingUnits\] should be 0. They come with stricter usage limits and limited support.

## FreeInstanceMetadata

Free instance specific metadata that is kept even after an instance has been upgraded for tracking purposes.

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

## ExpireBehavior

Allows users to change behavior when a free instance expires.

Enums

`  EXPIRE_BEHAVIOR_UNSPECIFIED  `

Not specified.

`  FREE_TO_PROVISIONED  `

When the free instance expires, upgrade the instance to a provisioned instance.

`  REMOVE_AFTER_GRACE_PERIOD  `

When the free instance expires, disable the instance, and delete it after the grace period passes if it has not been upgraded.

## DefaultBackupScheduleType

Indicates the [default backup schedule](https://cloud.google.com/spanner/docs/backup#default-backup-schedules) behavior for new databases within the instance.

Enums

`  DEFAULT_BACKUP_SCHEDULE_TYPE_UNSPECIFIED  `

Not specified.

`  NONE  `

A default backup schedule isn't created automatically when a new database is created in the instance.

`  AUTOMATIC  `

A default backup schedule is created automatically when a new database is created in the instance. The default backup schedule creates a full backup every 24 hours. These full backups are retained for 7 days. You can edit or delete the default backup schedule once it's created.

## Methods

### `             create           `

Creates an instance and begins preparing it to begin serving.

### `             delete           `

Deletes an instance.

### `             get           `

Gets information about a particular instance.

### `             getIamPolicy           `

Gets the access control policy for an instance resource.

### `             list           `

Lists all instances in the given project.

### `             move           `

Moves an instance to the target instance configuration.

### `             patch           `

Updates an instance, and begins allocating or releasing resources as requested.

### `             setIamPolicy           `

Sets the access control policy on an instance resource.

### `             testIamPermissions           `

Returns permissions that the caller has on the specified instance resource.
