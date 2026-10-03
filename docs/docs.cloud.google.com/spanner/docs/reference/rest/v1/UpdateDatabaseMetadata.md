---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata
title: UpdateDatabaseMetadata
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata#SCHEMA_REPRESENTATION)
- [UpdateDatabaseRequest](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata#UpdateDatabaseRequest)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata#UpdateDatabaseRequest.SCHEMA_REPRESENTATION)

Metadata type for the operation returned by [`databases.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabase) .

**JSON representation**

```
{
  "request": {
    object (UpdateDatabaseRequest)
  },
  "progress": {
    object (OperationProgress)
  },
  "cancelTime": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|--------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `request`    | `object ( `[`UpdateDatabaseRequest`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/UpdateDatabaseMetadata#UpdateDatabaseRequest)` )` The request for [`databases.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabase) .                                                                                                                                                 |
| `progress`   | `object ( `[`OperationProgress`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/OperationProgress)` )` The progress of the [`databases.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabase) operation.                                                                                                                                                                   |
| `cancelTime` | `string ( `[`Timestamp`](https://protobuf.dev/reference/protobuf/google.protobuf/#timestamp)` format)` The time at which this operation was cancelled. If set, this operation is in the process of undoing itself (which is best-effort). Uses RFC 3339, where generated output will always be Z-normalized and use 0, 3, 6 or 9 fractional digits. Offsets other than "Z" are also accepted. Examples: `"2014-10-02T15:01:23Z"` , `"2014-10-02T15:01:23.045123456Z"` or `"2014-10-02T15:01:23+05:30"` . |

## UpdateDatabaseRequest

The request for [`databases.patch`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/patch#google.spanner.admin.database.v1.DatabaseAdmin.UpdateDatabase) .

**JSON representation**

```
{
  "database": {
    object (Database)
  },
  "updateMask": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                       |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `database`   | `object ( `[`Database`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases#Database)` )` Required. The database to update. The `name` field of the database is of the form `projects/<project>/instances/<instance>/databases/<database>` .                                    |
| `updateMask` | `string ( `[`FieldMask`](https://protobuf.dev/reference/protobuf/google.protobuf/#field-mask)` format)` Required. The list of fields to update. Currently, only `enableDropProtection` field can be updated. This is a comma-separated list of fully qualified names of fields. Example: `"user.displayName,photo"` . |
