---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/OptimizeRestoredDatabaseMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OptimizeRestoredDatabaseMetadata
title: OptimizeRestoredDatabaseMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OptimizeRestoredDatabaseMetadata#SCHEMA_REPRESENTATION)

Metadata type for the long-running operation used to track the progress of optimizations performed on a newly restored database. This long-running operation is automatically created by the system after the successful completion of a database restore, and cannot be cancelled.

**JSON representation**

```
{
  "name": string,
  "progress": {
    object (OperationProgress)
  }
}
```

| Fields     |                                                                                                                                                                      |
|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`     | `string` Name of the restored database being optimized.                                                                                                              |
| `progress` | `object ( `[`OperationProgress`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OperationProgress)` )` The progress of the post-restore optimizations. |
