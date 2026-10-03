---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstanceConfigMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstanceConfigMetadata
title: CreateInstanceConfigMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateInstanceConfigMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`instanceConfigs.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstanceConfig) .

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
| `instanceConfig` | `object ( `[`InstanceConfig`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs#InstanceConfig)` )` The target instance configuration end state.                                                                                                                                                                                                                                  |
| `progress`       | `object ( ``OperationProgress`` )` The progress of the [`instanceConfigs.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/create#google.spanner.admin.instance.v1.InstanceAdmin.CreateInstanceConfig) operation.                                                                                                                                                        |
| `cancelTime`     | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
