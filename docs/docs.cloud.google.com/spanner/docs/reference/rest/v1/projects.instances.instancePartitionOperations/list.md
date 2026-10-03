---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list
title: 'Method: projects.instances.instancePartitionOperations.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.ListInstancePartitionOperationsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#try-it)

Lists instance partition long-running operations in the given instance. An instance partition operation has a name of the form `projects/<project>/instances/<instance>/instancePartitions/<instancePartition>/operations/<operation>` . The long-running operation metadata field type `metadata.type_url` describes the type of the metadata. Operations returned include those that have completed/failed/canceled within the last 7 days, and pending operations. Operations returned are ordered by `operation.metadata.value.start_time` in descending order starting from the most recently started operation.

Authorization requires `spanner.instancePartitionOperations.list` permission on the resource [`parent`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.PATH_PARAMETERS.parent) .

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/instancePartitionOperations`

The URLs use [gRPC Transcoding](https://google.aip.dev/127) syntax.

### Path parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>parent</code></td>
<td><p><code>string</code></p>
<p>Required. The parent instance of the instance partition operations. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.instancePartitionOperations.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Parameters</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>filter</code></td>
<td><p><code>string</code></p>
<p>Optional. An expression that filters the list of returned operations.</p>
<p>A filter expression consists of a field name, a comparison operator, and a value for filtering. The value must be a string, a number, or a boolean. The comparison operator must be one of: <code>&lt;</code> , <code>&gt;</code> , <code>&lt;=</code> , <code>&gt;=</code> , <code>!=</code> , <code>=</code> , or <code>:</code> . Colon <code>:</code> is the contains operator. Filter rules are not case sensitive.</p>
<p>The following fields in the Operation are eligible for filtering:</p>
<ul>
<li><code>name</code> - The name of the long-running operation</li>
<li><code>done</code> - False if the operation is in progress, else true.</li>
<li><code>metadata.@type</code> - the type of metadata. For example, the type string for <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstancePartitionMetadata"><code>CreateInstancePartitionMetadata</code></a> is <code>type.googleapis.com/google.spanner.admin.instance.v1.CreateInstancePartitionMetadata</code> .</li>
<li><code>metadata.&lt;field_name&gt;</code> - any field in metadata.value. <code>metadata.@type</code> must be specified first, if filtering on metadata fields.</li>
<li><code>error</code> - Error associated with the long-running operation.</li>
<li><code>response.@type</code> - the type of response.</li>
<li><code>response.&lt;field_name&gt;</code> - any field in response.value.</li>
</ul>
<p>You can combine multiple expressions by enclosing each expression in parentheses. By default, expressions are combined with AND logic. However, you can specify AND, OR, and NOT logic explicitly.</p>
<p>Here are a few examples:</p>
<ul>
<li><code>done:true</code> - The operation is complete.</li>
<li><code>(metadata.@type=</code> \ <code>type.googleapis.com/google.spanner.admin.instance.v1.CreateInstancePartitionMetadata) AND</code> \ <code>(metadata.instance_partition.name:custom-instance-partition) AND</code> \ <code>(metadata.start_time &lt; \"2021-03-28T14:50:00Z\") AND</code> \ <code>(error:*)</code> - Return operations where:
<ul>
<li>The operation's metadata type is <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstancePartitionMetadata"><code>CreateInstancePartitionMetadata</code></a> .</li>
<li>The instance partition name contains "custom-instance-partition".</li>
<li>The operation started before 2021-03-28T14:50:00Z.</li>
<li>The operation resulted in an error.</li>
</ul></li>
</ul></td>
</tr>
<tr class="even">
<td><code>pageSize</code></td>
<td><p><code>integer</code></p>
<p>Optional. Number of operations to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.</p></td>
</tr>
<tr class="odd">
<td><code>pageToken</code></td>
<td><p><code>string</code></p>
<p>Optional. If non-empty, <code>pageToken</code> should contain a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.ListInstancePartitionOperationsResponse.FIELDS.next_page_token"><code>nextPageToken</code></a> from a previous <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.ListInstancePartitionOperationsResponse"><code>ListInstancePartitionOperationsResponse</code></a> to the same <code>parent</code> and with the same <code>filter</code> .</p></td>
</tr>
<tr class="even">
<td><code>instancePartitionDeadline</code></td>
<td><p><code>string ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp"><code>Timestamp</code></a><code> format)</code></p>
<p>Optional. Deadline used while retrieving metadata for instance partition operations. Instance partitions whose operation metadata cannot be retrieved within this deadline will be added to <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.ListInstancePartitionOperationsResponse.FIELDS.unreachable_instance_partitions"><code>unreachableInstancePartitions</code></a> in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.ListInstancePartitionOperationsResponse"><code>ListInstancePartitionOperationsResponse</code></a> .</p>
<p>Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: <code>"2014-10-02T15:01:23Z"</code> , <code>"2014-10-02T15:01:23.045123456Z"</code> or <code>"2014-10-02T15:01:23+05:30"</code> .</p></td>
</tr>
</tbody>
</table>

### Request body

The request body must be empty.

### Response body

The response for [`instancePartitionOperations.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstancePartitionOperations) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "operations": [
    {
      object (Operation)
    }
  ],
  "nextPageToken": string,
  "unreachableInstancePartitions": [
    string
  ]
}
```

| Fields                            |                                                                                                                                                                                                                                                                                                                                                                                |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `operations[]`                    | `object ( `[`Operation`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs.operations#Operation)` )` The list of matching instance partition long-running operations. Each operation's name will be prefixed by the instance partition's name. The operation's metadata field type `metadata.type_url` describes the type of the metadata. |
| `nextPageToken`                   | `string` `nextPageToken` can be sent in a subsequent [`instancePartitionOperations.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstancePartitionOperations) call to fetch more of the matching metadata.                                        |
| `unreachableInstancePartitions[]` | `string` The list of unreachable instance partitions. It includes the names of instance partitions whose operation metadata could not be retrieved within [`instancePartitionDeadline`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitionOperations/list#body.QUERY_PARAMETERS.instance_partition_deadline) .                  |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
