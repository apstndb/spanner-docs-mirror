---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/MoveInstanceMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MoveInstanceMetadata
title: MoveInstanceMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/MoveInstanceMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`instances.move`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#google.spanner.admin.instance.v1.InstanceAdmin.MoveInstance) .

**JSON representation**

```
{
  "targetConfig": string,
  "progress": {
    object (OperationProgress)
  },
  "cancelTime": string
}
```

| Fields         |                                                                                                                                                                                                                                                                                                                                                                                                                       |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `targetConfig` | `string` The target instance configuration where to move the instance. Values are of the form `projects/<project>/instanceConfigs/<config>` .                                                                                                                                                                                                                                                                         |
| `progress`     | `object ( ``OperationProgress`` )` The progress of the [`instances.move`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/move#google.spanner.admin.instance.v1.InstanceAdmin.MoveInstance) operation. `progressPercent` is reset when cancellation is requested.                                                                                                                     |
| `cancelTime`   | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
