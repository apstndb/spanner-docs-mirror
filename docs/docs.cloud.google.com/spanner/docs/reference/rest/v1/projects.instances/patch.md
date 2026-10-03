---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch
title: 'Method: projects.instances.patch'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.request_body.SCHEMA_REPRESENTATION)
    - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION)
    - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION_1)
    - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.request_body.SCHEMA_REPRESENTATION.instance.SCHEMA_REPRESENTATION_2)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#try-it)

Updates an instance, and begins allocating or releasing resources as requested. The returned long-running operation can be used to track the progress of updating the instance. If the named instance does not exist, returns `NOT_FOUND` .

Immediately upon completion of this request:

- For resource types for which a decrease in the instance's allocation has been requested, billing is based on the newly-requested level.

Until completion of the returned operation:

- Cancelling the operation sets its metadata's [`cancelTime`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceMetadata#FIELDS.cancel_time) , and begins restoring resources to their pre-request values. The operation is guaranteed to succeed at undoing all resource changes, after which point it terminates with a `CANCELLED` status.
- All other attempts to modify the instance are rejected.
- Reading the instance via the API continues to give the pre-request resource levels.

Upon completion of the returned operation:

- Billing begins for all successfully-allocated resources (some types may have lower than the requested levels).
- All newly-reserved resources are available for serving the instance's tables.
- The instance's new resource levels are readable via the API.

The returned long-running operation will have a name of the format `<instance_name>/operations/<operationId>` and can be used to track the instance modification. The metadata field type is [`UpdateInstanceMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceMetadata) . The response field type is [`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance) , if successful.

Authorization requires `spanner.instances.update` permission on the resource [`name`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance.FIELDS.name) .

### HTTP request

Choose a location:

  
`PATCH https://spanner.googleapis.com/v1/{instance.name=projects/*/instances/*}`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

| Parameters      |                                                                                                                                                                                                                                                                              |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instance.name` | `string` Required. A unique identifier for the instance, which cannot be changed after the instance is created. Values are of the form `projects/<project>/instances/[a-z][-a-z0-9]*[a-z0-9]` . The final segment of the name must be between 2 and 64 characters in length. |

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "instance": {
    "name": string,
    "config": string,
    "displayName": string,
    "nodeCount": integer,
    "processingUnits": integer,
    "replicaComputeCapacity": [
      {
        "replicaSelection": {
          object (ReplicaSelection)
        },

        // Union field compute_capacity can be only one of the following:
        "nodeCount": integer,
        "processingUnits": integer
        // End of list of possible types for union field compute_capacity.
      }
    ],
    "autoscalingConfig": {
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
      "expireTime": string,
      "upgradeTime": string,
      "expireBehavior": enum (ExpireBehavior)
    },
    "edition": enum (Edition),
    "defaultBackupScheduleType": enum (DefaultBackupScheduleType)
  },
  "fieldMask": string
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
<td><code>instance.config</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the instance's configuration. Values are of the form <code>projects/&lt;project&gt;/instanceConfigs/&lt;configuration&gt;</code> . See also <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig"><code>InstanceConfig</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigs"><code>ListInstanceConfigs</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>instance.displayName</code></td>
<td><p><code>string</code></p>
<p>Required. The descriptive name for this instance as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.</p></td>
</tr>
<tr class="odd">
<td><code>instance.nodeCount</code></td>
<td><p><code>integer</code></p>
<p>The number of nodes allocated to this instance. At most, one of either <code>nodeCount</code> or <code>processingUnits</code> should be present in the message.</p>
<p>Users can set the <code>nodeCount</code> field to specify the target number of nodes allocated to the instance.</p>
<p>If autoscaling is enabled, <code>nodeCount</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of nodes allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying node count across replicas (achieved by setting <code>asymmetricAutoscalingOptions</code> in the autoscaling configuration), the <code>nodeCount</code> set here is the maximum node count across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes, and processing units</a> .</p></td>
</tr>
<tr class="even">
<td><code>instance.processingUnits</code></td>
<td><p><code>integer</code></p>
<p>The number of processing units allocated to this instance. At most, one of either <code>processingUnits</code> or <code>nodeCount</code> should be present in the message.</p>
<p>Users can set the <code>processingUnits</code> field to specify the target number of processing units allocated to the instance.</p>
<p>If autoscaling is enabled, <code>processingUnits</code> is treated as an <code>OUTPUT_ONLY</code> field and reflects the current number of processing units allocated to the instance.</p>
<p>This might be zero in API responses for instances that are not yet in the <code>READY</code> state.</p>
<p>If the instance has varying processing units per replica (achieved by setting <code>asymmetricAutoscalingOptions</code> in the autoscaling configuration), the <code>processingUnits</code> set here is the maximum processing units across all replicas.</p>
<p>For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes and processing units</a> .</p></td>
</tr>
<tr class="odd">
<td><code>instance.replicaComputeCapacity[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#ReplicaComputeCapacity"><code>ReplicaComputeCapacity</code></a><code> )</code></p>
<p>Output only. Lists the compute capacity per ReplicaSelection. A replica selection identifies a set of replicas with common properties. Replicas identified by a ReplicaSelection are scaled with the same compute capacity.</p></td>
</tr>
<tr class="even">
<td><code>instance.autoscalingConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig"><code>AutoscalingConfig</code></a><code> )</code></p>
<p>Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, nodeCount and processingUnits are treated as OUTPUT_ONLY fields and reflect the current compute capacity allocated to the instance.</p></td>
</tr>
<tr class="odd">
<td><code>instance.state</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#State"><code>State</code></a><code> )</code></p>
<p>Output only. The current instance state. For <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstance"><code>instances.create</code></a> , the state must be either omitted or set to <code>CREATING</code> . For <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance"><code>instances.patch</code></a> , the state must be either omitted or set to <code>READY</code> .</p></td>
</tr>
<tr class="even">
<td><code>instance.labels</code></td>
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
<tr class="odd">
<td><code>instance.instanceType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#InstanceType"><code>InstanceType</code></a><code> )</code></p>
<p>The <code>InstanceType</code> of the current instance.</p></td>
</tr>
<tr class="even">
<td><code>instance.endpointUris[]</code></td>
<td><p><code>string</code></p>
<p>Deprecated. This field is not populated.</p></td>
</tr>
<tr class="odd">
<td><code>instance.createTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance was created.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="even">
<td><code>instance.updateTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance was most recently updated.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="odd">
<td><code>instance.freeInstanceMetadata</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#FreeInstanceMetadata"><code>FreeInstanceMetadata</code></a><code> )</code></p>
<p>Free instance metadata. Only populated for free instances.</p></td>
</tr>
<tr class="even">
<td><code>instance.edition</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Edition"><code>Edition</code></a><code> )</code></p>
<p>Optional. The <code>Edition</code> of the current instance.</p></td>
</tr>
<tr class="odd">
<td><code>instance.defaultBackupScheduleType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#DefaultBackupScheduleType"><code>DefaultBackupScheduleType</code></a><code> )</code></p>
<p>Optional. Controls the default backup schedule behavior for new databases within the instance. By default, a backup schedule is created automatically when a new database is created in a new instance.</p>
<p>Note that the <code>AUTOMATIC</code> value isn't permitted for free instances, as backups and backup schedules aren't supported for free instances.</p>
<p>In the <code>instances.get</code> or <code>instances.list</code> response, if the value of <code>defaultBackupScheduleType</code> isn't set, or set to <code>NONE</code> , Spanner doesn't create a default backup schedule for new databases in the instance.</p></td>
</tr>
<tr class="even">
<td><code>fieldMask</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#field-mask"><code>FieldMask</code></a><code> format)</code></p>
<p>Required. A mask specifying which fields in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance"><code>Instance</code></a> should be updated. The field mask must always be specified; this prevents any future fields in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance"><code>Instance</code></a> from being erased accidentally by clients that do not know about them.</p>
<p>This is a comma-separated list of fully qualified names of fields. Example: <code>"user.displayName,photo"</code> .</p></td>
</tr>
</tbody>
</table>

### Response body

If successful, the response body contains an instance of [`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation) .

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
