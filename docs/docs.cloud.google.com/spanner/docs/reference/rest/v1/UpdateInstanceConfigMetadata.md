---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceConfigMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceConfigMetadata
title: UpdateInstanceConfigMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceConfigMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`instanceConfigs.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstanceConfig) .

**JSON representation**

```
{
  "instanceConfig": {
    object (InstanceConfig)
  },
  "progress": {
    object (OperationProgress)
  },
  "cancelTime": string
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instanceConfig` | `object ( `[`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig)` )` The desired instance configuration after updating.                                                                                                                                                                                                                            |
| `progress`       | `object ( ``OperationProgress`` )` The progress of the [`instanceConfigs.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstanceConfig) operation.                                                                                                                                                          |
| `cancelTime`     | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
