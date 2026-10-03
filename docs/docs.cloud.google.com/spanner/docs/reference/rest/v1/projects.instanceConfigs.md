---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs
title: 'REST Resource: projects.instanceConfigs'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [Resource: InstanceConfig](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.SCHEMA_REPRESENTATION)
- [Type](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#Type)
- [ReplicaInfo](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo.SCHEMA_REPRESENTATION)
- [ReplicaType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaType)
- [State](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#State)
- [FreeInstanceAvailability](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#FreeInstanceAvailability)
- [QuorumType](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#QuorumType)
- [Methods](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#METHODS_SUMMARY)

## Resource: InstanceConfig

A possible configuration for a Cloud Spanner instance. Configurations define the geographic placement of nodes and their replication.

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
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#Type"><code>Type</code></a><code> )</code></p>
<p>Output only. Whether this instance configuration is a Google-managed or user-managed configuration.</p></td>
</tr>
<tr class="even">
<td><code>replicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>The geographic placement of nodes in this instance configuration and their replication properties.</p>
<p>To create user-managed configurations, input <code>replicas</code> must include all replicas in <code>replicas</code> of the <code>baseConfig</code> and include one or more replicas in the <code>optionalReplicas</code> of the <code>baseConfig</code> .</p></td>
</tr>
<tr class="odd">
<td><code>optionalReplicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.</p></td>
</tr>
<tr class="even">
<td><code>baseConfig</code></td>
<td><p><code>string</code></p>
<p>Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. <code>baseConfig</code> must refer to a configuration of type <code>GOOGLE_MANAGED</code> in the same project as this configuration.</p></td>
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
<p>If you plan to use labels in your own code, please note that additional characters may be allowed in the future. Therefore, you are advised to use an internal label representation, such as JSON, which doesn't rely upon specific characters being disallowed. For example, representing labels as the string: name + "_" + value would prove problematic if we were to allow "_" in a future release.</p></td>
</tr>
<tr class="even">
<td><code>etag</code></td>
<td><p><code>string</code></p>
<p>etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.</p></td>
</tr>
<tr class="odd">
<td><code>leaderOptions[]</code></td>
<td><p><code>string</code></p>
<p>Allowed values of the "defaultLeader" schema option for databases in instances that use this instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>reconciling</code></td>
<td><p><code>boolean</code></p>
<p>Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>state</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#State"><code>State</code></a><code> )</code></p>
<p>Output only. The current instance configuration state. Applicable only for <code>USER_MANAGED</code> configurations.</p></td>
</tr>
<tr class="even">
<td><code>freeInstanceAvailability</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#FreeInstanceAvailability"><code>FreeInstanceAvailability</code></a><code> )</code></p>
<p>Output only. Describes whether free instances are available to be created in this instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>quorumType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#QuorumType"><code>QuorumType</code></a><code> )</code></p>
<p>Output only. The <code>QuorumType</code> of the instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>storageLimitPerProcessingUnit</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>int64</code></a><code> format)</code></p>
<p>Output only. The storage limit in bytes per processing unit.</p></td>
</tr>
</tbody>
</table>

## Type

The type of this configuration.

| Enums              |                               |
|--------------------|-------------------------------|
| `TYPE_UNSPECIFIED` | Unspecified.                  |
| `GOOGLE_MANAGED`   | Google-managed configuration. |
| `USER_MANAGED`     | User-managed configuration.   |

## ReplicaInfo

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
| `type`                  | `enum ( `[`ReplicaType`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaType)` )` The type of replica.                                                                                 |
| `defaultLeaderLocation` | `boolean` If true, this location is designated as the default leader location where leader replicas are placed. See the [region types documentation](https://cloud.google.com/spanner/docs/instances#region_types) for more details. |

## ReplicaType

Indicates the type of replica. See the [replica types documentation](https://cloud.google.com/spanner/docs/replication#replica_types) for more details.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Enums</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>TYPE_UNSPECIFIED</code></td>
<td>Not specified.</td>
</tr>
<tr class="even">
<td><code>READ_WRITE</code></td>
<td><p>sessions.read-write replicas support both reads and writes. These replicas:</p>
<ul>
<li>Maintain a full copy of your data.</li>
<li>Serve reads.</li>
<li>Can vote whether to commit a write.</li>
<li>Participate in leadership election.</li>
<li>Are eligible to become a leader.</li>
</ul></td>
</tr>
<tr class="odd">
<td><code>READ_ONLY</code></td>
<td><p>sessions.read-only replicas only support reads (not writes). sessions.read-only replicas:</p>
<ul>
<li>Maintain a full copy of your data.</li>
<li>Serve reads.</li>
<li>Do not participate in voting to commit writes.</li>
<li>Are not eligible to become a leader.</li>
</ul></td>
</tr>
<tr class="even">
<td><code>WITNESS</code></td>
<td><p>Witness replicas don't support reads but do participate in voting to commit writes. Witness replicas:</p>
<ul>
<li>Do not maintain a full copy of data.</li>
<li>Do not serve reads.</li>
<li>Vote whether to commit writes.</li>
<li>Participate in leader election but are not eligible to become leader.</li>
</ul></td>
</tr>
</tbody>
</table>

## State

Indicates the current state of the instance configuration.

| Enums               |                                                                                       |
|---------------------|---------------------------------------------------------------------------------------|
| `STATE_UNSPECIFIED` | Not specified.                                                                        |
| `CREATING`          | The instance configuration is still being created.                                    |
| `READY`             | The instance configuration is fully created and ready to be used to create instances. |

## FreeInstanceAvailability

Describes the availability for free instances to be created in an instance configuration.

| Enums                                    |                                                                                                                                                        |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `FREE_INSTANCE_AVAILABILITY_UNSPECIFIED` | Not specified.                                                                                                                                         |
| `AVAILABLE`                              | Indicates that free instances are available to be created in this instance configuration.                                                              |
| `UNSUPPORTED`                            | Indicates that free instances are not supported in this instance configuration.                                                                        |
| `DISABLED`                               | Indicates that free instances are currently not available to be created in this instance configuration.                                                |
| `QUOTA_EXCEEDED`                         | Indicates that additional free instances cannot be created in this instance configuration because the project has reached its limit of free instances. |

## QuorumType

Indicates the quorum type of this instance configuration.

| Enums                     |                                                                                                                                                                                                                                                |
|---------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `QUORUM_TYPE_UNSPECIFIED` | Quorum type not specified.                                                                                                                                                                                                                     |
| `REGION`                  | An instance configuration tagged with `REGION` quorum type forms a write quorum in a single region.                                                                                                                                            |
| `DUAL_REGION`             | An instance configuration tagged with the `DUAL_REGION` quorum type forms a write quorum with exactly two read-write regions in a multi-region configuration. This instance configuration requires failover in the event of regional failures. |
| `MULTI_REGION`            | An instance configuration tagged with the `MULTI_REGION` quorum type forms a write quorum from replicas that are spread across more than one region in a multi-region configuration.                                                           |

| Methods                                                                                                  |                                                                       |
|----------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|
| [`create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create) | Creates an instance configuration and begins preparing it to be used. |
| [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/delete) | Deletes the instance configuration.                                   |
| [`get`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/get)       | Gets information about a particular instance configuration.           |
| [`list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list)     | Lists the supported instance configurations for a given project.      |
| [`patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch)   | Updates an instance configuration.                                    |
