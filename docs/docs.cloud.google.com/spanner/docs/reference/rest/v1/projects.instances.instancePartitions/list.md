---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list
title: 'Method: projects.instances.instancePartitions.list'
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [HTTP request](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.HTTP_TEMPLATE)
- [Path parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.PATH_PARAMETERS)
- [Query parameters](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.QUERY_PARAMETERS)
- [Request body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.request_body)
- [Response body](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.response_body)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.ListInstancePartitionsResponse.SCHEMA_REPRESENTATION)
- [Authorization scopes](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.aspect)
- [Try it!](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#try-it)

Lists all instance partitions for the given instance.

### HTTP request

Choose a location:

  
`GET https://spanner.googleapis.com/v1/{parent=projects/*/instances/*}/instancePartitions`

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
<p>Required. The instance whose instance partitions should be listed. Values are of the form <code>projects/&lt;project&gt;/instances/&lt;instance&gt;</code> . Use <code>{instance} = '-'</code> to list instance partitions for all Instances in a project, e.g., <code>projects/myproject/instances/-</code> .</p>
<p>Authorization requires the following <a href="https://cloud.google.com/iam/docs/">IAM</a> permission on the specified resource <code>parent</code> :</p>
<ul>
<li><code>spanner.instancePartitions.list</code></li>
</ul></td>
</tr>
</tbody>
</table>

### Query parameters

| Parameters                  |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pageSize`                  | `integer` Number of instance partitions to be returned in the response. If 0 or less, defaults to the server's maximum allowed page size.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `pageToken`                 | `string` If non-empty, `pageToken` should contain a [`nextPageToken`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.ListInstancePartitionsResponse.FIELDS.next_page_token) from a previous [`ListInstancePartitionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.ListInstancePartitionsResponse) .                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `instancePartitionDeadline` | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Optional. Deadline used while retrieving metadata for instance partitions. Instance partitions whose metadata cannot be retrieved within this deadline will be added to [`unreachable`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.ListInstancePartitionsResponse.FIELDS.unreachable) in [`ListInstancePartitionsResponse`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.ListInstancePartitionsResponse) . Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |

### Request body

The request body must be empty.

### Response body

The response for [`instancePartitions.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstancePartitions) .

If successful, the response body contains data with the following structure:

**JSON representation**

```
{
  "instancePartitions": [
    {
      object (InstancePartition)
    }
  ],
  "nextPageToken": string,
  "unreachable": [
    string
  ]
}
```

| Fields                 |                                                                                                                                                                                                                                                                                                                                                                      |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instancePartitions[]` | `object ( `[`InstancePartition`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#InstancePartition)` )` The list of requested instancePartitions.                                                                                                                                                                 |
| `nextPageToken`        | `string` `nextPageToken` can be sent in a subsequent [`instancePartitions.list`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#google.spanner.admin.instance.v1.InstanceAdmin.ListInstancePartitions) call to fetch more of the matching instance partitions.                                              |
| `unreachable[]`        | `string` The list of unreachable instances or instance partitions. It includes the names of instances or instance partitions whose metadata could not be retrieved within [`instancePartitionDeadline`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/list#body.QUERY_PARAMETERS.instance_partition_deadline) . |

### Authorization scopes

Requires one of the following OAuth scopes:

- `https://www.googleapis.com/auth/spanner.admin`
- `https://www.googleapis.com/auth/cloud-platform`

For more information, see the [Authentication Overview](https://docs.cloud.google.com/docs/authentication#authorization-gcp) .
