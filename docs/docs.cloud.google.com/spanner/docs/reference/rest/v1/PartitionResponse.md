---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse
title: PartitionResponse
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse#SCHEMA_REPRESENTATION)
- [Partition](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse#Partition)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse#Partition.SCHEMA_REPRESENTATION)

The response for [`sessions.partitionQuery`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery#google.spanner.v1.Spanner.PartitionQuery) or [`sessions.partitionRead`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionRead#google.spanner.v1.Spanner.PartitionRead)

**JSON representation**

```
{
  "partitions": [
    {
      object (Partition)
    }
  ],
  "transaction": {
    object (Transaction)
  }
}
```

| Fields         |                                                                                                                                                            |
|----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partitions[]` | `object ( `[`Partition`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartitionResponse#Partition)` )` Partitions created by this request. |
| `transaction`  | `object ( `[`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction)` )` Transaction created by this request.              |

## Partition

Information returned for each partition returned in a PartitionResponse.

**JSON representation**

```
{
  "partitionToken": string
}
```

| Fields           |                                                                                                                                                                                                                                                                                                                         |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `partitionToken` | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` This token can be passed to `sessions.read` , `sessions.streamingRead` , `ExecuteSql` , or `sessions.executeStreamingSql` requests to restrict the results to those identified by this partition token. A base64-encoded string. |
