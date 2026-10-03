---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions
title: 'REST Resource: projects.instances.instancePartitions'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [Resource: InstancePartition](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#InstancePartition)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#InstancePartition.SCHEMA_REPRESENTATION)
- [State](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#State)
- [Methods](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#METHODS_SUMMARY)

## Resource: InstancePartition

An isolated set of Cloud Spanner resources that databases can define placements on.

**JSON representation**

```
{
  "name": string,
  "config": string,
  "displayName": string,
  "autoscalingConfig": {
    object (AutoscalingConfig)
  },
  "state": enum (State),
  "createTime": string,
  "updateTime": string,
  "referencingDatabases": [
    string
  ],
  "referencingBackups": [
    string
  ],
  "etag": string,

  // Union field compute_capacity can be only one of the following:
  "nodeCount": integer,
  "processingUnits": integer
  // End of list of possible types for union field compute_capacity.
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
<p>Required. A unique identifier for the instance partition. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;/instancePartitions/[a-z][-a-z0-9]*[a-z0-9]</code> . The final segment of the name must be between 2 and 64 characters in length. An instance partition's name cannot be changed after the instance partition is created.</p></td>
</tr>
<tr class="even">
<td><code>config</code></td>
<td><p><code>string</code></p>
<p>Required. The name of the instance partition's configuration. Values are of the form <code>projects/&lt;project&gt;/instanceConfigs/&lt;configuration&gt;</code> . See also <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig"><code>InstanceConfig</code></a> and <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstanceConfigs"><code>ListInstanceConfigs</code></a> .</p></td>
</tr>
<tr class="odd">
<td><code>displayName</code></td>
<td><p><code>string</code></p>
<p>Required. The descriptive name for this instance partition as it appears in UIs. Must be unique per project and between 4 and 30 characters in length.</p></td>
</tr>
<tr class="even">
<td><code>autoscalingConfig</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/AutoscalingConfig"><code>AutoscalingConfig</code></a><code> )</code></p>
<p>Optional. The autoscaling configuration. Autoscaling is enabled if this field is set. When autoscaling is enabled, fields in compute_capacity are treated as OUTPUT_ONLY fields and reflect the current compute capacity allocated to the instance partition.</p></td>
</tr>
<tr class="odd">
<td><code>state</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#State"><code>State</code></a><code> )</code></p>
<p>Output only. The current instance partition state.</p></td>
</tr>
<tr class="even">
<td><code>createTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance partition was created.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="odd">
<td><code>updateTime</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Output only. The time at which the instance partition was most recently updated.</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
<tr class="even">
<td><code>referencingDatabases[]</code></td>
<td><p><code>string</code></p>
<p>Output only. The names of the databases that reference this instance partition. Referencing databases should share the parent instance. The existence of any referencing database prevents the instance partition from being deleted.</p></td>
</tr>
<tr class="odd">
<td><code>referencingBackups[] </code><strong><code>(deprecated)</code></strong></td>
<td><p><code>string</code></p>
<blockquote>
<p>This item is deprecated!</p>
</blockquote>
<p>Output only. Deprecated: This field is not populated. Output only. The names of the backups that reference this instance partition. Referencing backups should share the parent instance. The existence of any referencing backup prevents the instance partition from being deleted.</p></td>
</tr>
<tr class="even">
<td><code>etag</code></td>
<td><p><code>string</code></p>
<p>Used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance partition from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance partition updates in order to avoid race conditions: An etag is returned in the response which contains instance partitions, and systems are expected to put that etag in the request to update instance partitions to ensure that their change will be applied to the same version of the instance partition. If no etag is provided in the call to update instance partition, then the existing instance partition is overwritten blindly.</p></td>
</tr>
<tr class="odd">
<td>Union field <code>compute_capacity</code> . Compute capacity defines amount of server and storage resources that are available to the databases in an instance partition. At most, one of either <code>node_count</code> or <code>processing_units</code> should be present in the message. For more information, see <a href="https://cloud.google.com/spanner/docs/compute-capacity">Compute capacity, nodes, and processing units</a> . <code>compute_capacity</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>nodeCount</code></td>
<td><p><code>integer</code></p>
<p>The number of nodes allocated to this instance partition.</p>
<p>Users can set the <code>nodeCount</code> field to specify the target number of nodes allocated to the instance partition.</p>
<p>This may be zero in API responses for instance partitions that are not yet in state <code>READY</code> .</p></td>
</tr>
<tr class="odd">
<td><code>processingUnits</code></td>
<td><p><code>integer</code></p>
<p>The number of processing units allocated to this instance partition.</p>
<p>Users can set the <code>processingUnits</code> field to specify the target number of processing units allocated to the instance partition.</p>
<p>This might be zero in API responses for instance partitions that are not yet in the <code>READY</code> state.</p></td>
</tr>
</tbody>
</table>

## State

Indicates the current state of the instance partition.

| Enums               |                                                                                                                                                                           |
|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STATE_UNSPECIFIED` | Not specified.                                                                                                                                                            |
| `CREATING`          | The instance partition is still being created. Resources may not be available yet, and operations such as creating placements using this instance partition may not work. |
| `READY`             | The instance partition is fully created and ready to do work such as creating placements and using in databases.                                                          |

| Methods                                                                                                               |                                                                                           |
|-----------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| [`create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/create) | Creates an instance partition and begins preparing it to be used.                         |
| [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/delete) | Deletes an existing instance partition.                                                   |
| [`get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/get)       | Gets information about a particular instance partition.                                   |
| [`list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list)     | Lists all instance partitions for the given instance.                                     |
| [`patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/patch)   | Updates an instance partition, and begins allocating or releasing resources as requested. |
