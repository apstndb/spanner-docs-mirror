---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateDatabaseMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateDatabaseMetadata
title: CreateDatabaseMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/CreateDatabaseMetadata#SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`databases.create`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#google.spanner.admin.database.v1.DatabaseAdmin.CreateDatabase) .

**JSON representation**

```
{
  "database": string
}
```

| Fields     |                                      |
|------------|--------------------------------------|
| `database` | `string` The database being created. |
