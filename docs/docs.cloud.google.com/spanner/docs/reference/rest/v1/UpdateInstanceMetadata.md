---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceMetadata
title: UpdateInstanceMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateInstanceMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`instances.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance) .

**JSON representation**

```
{
  "instance": {
    object (Instance)
  },
  "startTime": string,
  "cancelTime": string,
  "endTime": string,
  "expectedFulfillmentPeriod": enum (FulfillmentPeriod)
}
```

| Fields                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|-----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `instance`                  | `object ( `[`Instance`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances#Instance)` )` The desired end state of the update.                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `startTime`                 | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which [`instances.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/patch#google.spanner.admin.instance.v1.InstanceAdmin.UpdateInstance) request was received. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
| `cancelTime`                | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is guaranteed to succeed) and cannot be cancelled again. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                       |
| `endTime`                   | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation failed or was completed successfully. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                                                                                                           |
| `expectedFulfillmentPeriod` | `enum ( `[`FulfillmentPeriod`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/FulfillmentPeriod)` )` The expected fulfillment period of this update operation.                                                                                                                                                                                                                                                                                                                                                                                                           |
