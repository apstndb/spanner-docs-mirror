---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionOptions
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionOptions
title: PartitionOptions
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionOptions#SCHEMA_REPRESENTATION)

Options for a `PartitionQueryRequest` and `PartitionReadRequest` .

**JSON representation**

```
{
  "partitionSizeBytes": string,
  "maxPartitions": string
}
```

| Fields               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partitionSizeBytes` | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` **Note:** This hint is currently ignored by `sessions.partitionQuery` and `sessions.partitionRead` requests. The desired data size for each partition generated. The default for this option is currently 1 GiB. This is only a hint. The actual size of each partition can be smaller or larger than this size request.                                                                                                                             |
| `maxPartitions`      | `string ( `[`int64`](https://developers.google.com/discovery/v1/type-format)` format)` **Note:** This hint is currently ignored by `sessions.partitionQuery` and `sessions.partitionRead` requests. The desired maximum number of partitions to return. For example, this might be set to the number of workers available. The default for this option is currently 10,000. The maximum value is currently 200,000. This is only a hint. The actual number of partitions returned can be smaller or larger than this maximum count request. |
