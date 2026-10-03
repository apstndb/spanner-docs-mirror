---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/OperationProgress
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OperationProgress
title: OperationProgress
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OperationProgress#SCHEMA_REPRESENTATION)

Encapsulates progress related information for a Cloud Spanner long running operation.

**JSON representation**

```
{
  "progressPercent": integer,
  "startTime": string,
  "endTime": string
}
```

| Fields            |                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `progressPercent` | `integer` Percent completion of the operation. Values are between 0 and 100 inclusive.                                                                                                                                                                                                                                                                                                                                                               |
| `startTime`       | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` Time the request was received. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` .                                                 |
| `endTime`         | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` If set, the time at which this operation failed or was completed successfully. Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |
