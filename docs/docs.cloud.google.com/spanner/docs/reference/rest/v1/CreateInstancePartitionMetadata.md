---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstancePartitionMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstancePartitionMetadata
title: CreateInstancePartitionMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstancePartitionMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`instancePartitions.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstancePartition) .

**JSON representation**

```
{
  "instancePartition": {
    object (InstancePartition)
  },
  "startTime": string,
  "cancelTime": string,
  "endTime": string
}
```

| Fields              |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instancePartition` | `object ( `[`InstancePartition`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions#InstancePartition)` )` The instance partition being created.                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `startTime`         | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which the [`instancePartitions.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.instancePartitions/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstancePartition) request was received. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `cancelTime`        | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                  |
| `endTime`           | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation failed or was completed successfully. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                                                                      |
