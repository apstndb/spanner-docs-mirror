---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch
title: 'Method: projects.instanceConfigs.patch'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.PATH_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.request_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.request_body.SCHEMA_REPRESENTATION)
    - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.request_body.SCHEMA_REPRESENTATION.instance_config.SCHEMA_REPRESENTATION)
    - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.request_body.SCHEMA_REPRESENTATION.instance_config.SCHEMA_REPRESENTATION_1)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.response_body)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#try-it)

Updates an instance configuration. The returned long-running operation can be used to track the progress of updating the instance. If the named instance configuration does not exist, returns `NOT_FOUND` .

Only user-managed configurations can be updated.

Immediately after the request returns:

- The instance configuration's [`reconciling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.FIELDS.reconciling) field is set to true.

While the operation is pending:

- Cancelling the operation sets its metadata's [`cancelTime`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceConfigMetadata#FIELDS.cancel_time) . The operation is guaranteed to succeed at undoing all changes, after which point it terminates with a `CANCELLED` status.
- All other attempts to modify the instance configuration are rejected.
- Reading the instance configuration via the API continues to give the pre-request values.

Upon completion of the returned operation:

- Creating instances using the instance configuration uses the new values.
- The new values of the instance configuration are readable via the API.
- The instance configuration's [`reconciling`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.FIELDS.reconciling) field becomes false.

The returned long-running operation will have a name of the format `<instance_config_name>/operations/<operationId>` and can be used to track the instance configuration modification. The metadata field type is [`UpdateInstanceConfigMetadata`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceConfigMetadata) . The response field type is [`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig) , if successful.

Authorization requires `spanner.instanceConfigs.update` permission on the resource [`name`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig.FIELDS.name) .

### HTTP request

Choose a location:

  
`PATCH https://spanner.googleapis.com/v1/{instanceConfig.name=projects/*/instanceConfigs/*}`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

| Parameters            |                                                                                                                                                                                                    |
|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instanceConfig.name` | `string` A unique identifier for the instance configuration. Values are of the form `projects/<project>/instanceConfigs/[a-z][-a-z0-9]*` . User instance configuration must start with `custom-` . |

### Request body

The request body contains data with the following structure:

**JSON representation**

```
{
  "instanceConfig": {
    "name": string,
    "displayName": string,
    "configType": enum (Type),
    "replicas": [
      {
        "location": string,
        "type": enum (ReplicaType),
        "defaultLeaderLocation": boolean
      }
    ],
    "optionalReplicas": [
      {
        "location": string,
        "type": enum (ReplicaType),
        "defaultLeaderLocation": boolean
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
  },
  "updateMask": string,
  "validateOnly": boolean
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
<td><code>instanceConfig.displayName</code></td>
<td><p><code>string</code></p>
<p>The name of this instance configuration as it appears in UIs.</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.configType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#Type"><code>Type</code></a><code> )</code></p>
<p>Output only. Whether this instance configuration is a Google-managed or user-managed configuration.</p></td>
</tr>
<tr class="odd">
<td><code>instanceConfig.replicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>The geographic placement of nodes in this instance configuration and their replication properties.</p>
<p>To create user-managed configurations, input <code>replicas</code> must include all replicas in <code>replicas</code> of the <code>baseConfig</code> and include one or more replicas in the <code>optionalReplicas</code> of the <code>baseConfig</code> .</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.optionalReplicas[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#ReplicaInfo"><code>ReplicaInfo</code></a><code> )</code></p>
<p>Output only. The available optional replicas to choose from for user-managed configurations. Populated for Google-managed configurations.</p></td>
</tr>
<tr class="odd">
<td><code>instanceConfig.baseConfig</code></td>
<td><p><code>string</code></p>
<p>Base configuration name, e.g. projects/ /instanceConfigs/nam3, based on which this configuration is created. Only set for user-managed configurations. <code>baseConfig</code> must refer to a configuration of type <code>GOOGLE_MANAGED</code> in the same project as this configuration.</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.labels</code></td>
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
<tr class="odd">
<td><code>instanceConfig.etag</code></td>
<td><p><code>string</code></p>
<p>etag is used for optimistic concurrency control as a way to help prevent simultaneous updates of a instance configuration from overwriting each other. It is strongly suggested that systems make use of the etag in the read-modify-write cycle to perform instance configuration updates in order to avoid race conditions: An etag is returned in the response which contains instance configurations, and systems are expected to put that etag in the request to update instance configuration to ensure that their change is applied to the same version of the instance configuration. If no etag is provided in the call to update the instance configuration, then the existing instance configuration is overwritten blindly.</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.leaderOptions[]</code></td>
<td><p><code>string</code></p>
<p>Allowed values of the "defaultLeader" schema option for databases in instances that use this instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>instanceConfig.reconciling</code></td>
<td><p><code>boolean</code></p>
<p>Output only. If true, the instance configuration is being created or updated. If false, there are no ongoing operations for the instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.state</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#State"><code>State</code></a><code> )</code></p>
<p>Output only. The current instance configuration state. Applicable only for <code>USER_MANAGED</code> configurations.</p></td>
</tr>
<tr class="odd">
<td><code>instanceConfig.freeInstanceAvailability</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#FreeInstanceAvailability"><code>FreeInstanceAvailability</code></a><code> )</code></p>
<p>Output only. Describes whether free instances are available to be created in this instance configuration.</p></td>
</tr>
<tr class="even">
<td><code>instanceConfig.quorumType</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#QuorumType"><code>QuorumType</code></a><code> )</code></p>
<p>Output only. The <code>QuorumType</code> of the instance configuration.</p></td>
</tr>
<tr class="odd">
<td><code>instanceConfig.storageLimitPerProcessingUnit</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>int64</code></a><code> format)</code></p>
<p>Output only. The storage limit in bytes per processing unit.</p></td>
</tr>
<tr class="even">
<td><code>updateMask</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#field-mask"><code>FieldMask</code></a><code> format)</code></p>
<p>Required. A mask specifying which fields in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig"><code>InstanceConfig</code></a> should be updated. The field mask must always be specified; this prevents any future fields in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig"><code>InstanceConfig</code></a> from being erased accidentally by clients that do not know about them. Only displayName and labels can be updated.</p>
<p>This is a comma-separated list of fully qualified names of fields. Example: <code>"user.displayName,photo"</code> .</p></td>
</tr>
<tr class="odd">
<td><code>validateOnly</code></td>
<td><p><code>boolean</code></p>
<p>An option to validate, but not actually execute, a request, and provide the same response.</p></td>
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
