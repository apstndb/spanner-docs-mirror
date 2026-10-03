---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation
title: Mutation
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#SCHEMA_REPRESENTATION)
- [Write](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write.SCHEMA_REPRESENTATION)
- [Delete](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Delete)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Delete.SCHEMA_REPRESENTATION)

A modification to one or more Cloud Spanner rows. Mutations can be applied to a Cloud Spanner database by sending them in a [`Commit`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#google.spanner.v1.Spanner.Commit) call.

**JSON representation**

```
{

  // Union field operation can be only one of the following:
  "insert": {
    object (Write)
  },
  "update": {
    object (Write)
  },
  "insertOrUpdate": {
    object (Write)
  },
  "replace": {
    object (Write)
  },
  "delete": {
    object (Delete)
  }
  // End of list of possible types for union field operation.
}
```

| Fields                                                                                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|-------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `operation` . Required. The operation to perform. `operation` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `insert`                                                                                                    | `object ( `[`Write`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write)` )` Insert new rows in a table. If any of the rows already exist, the write or transaction fails with error `ALREADY_EXISTS` .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `update`                                                                                                    | `object ( `[`Write`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write)` )` Update existing rows in a table. If any of the rows does not already exist, the transaction fails with error `NOT_FOUND` .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `insertOrUpdate`                                                                                            | `object ( `[`Write`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write)` )` Like [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert) , except that if the row already exists, then its column values are overwritten with the ones provided. Any column values not explicitly written are preserved. When using [`insertOrUpdate`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert_or_update) , just as when using [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert) , all `NOT NULL` columns in the table must be given a value. This holds true even when the row already exists and will therefore actually be updated. |
| `replace`                                                                                                   | `object ( `[`Write`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write)` )` Like [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert) , except that if the row already exists, it is deleted, and the column values provided are inserted instead. Unlike [`insertOrUpdate`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert_or_update) , this means any values not explicitly written become `NULL` . In an interleaved table, if you create the child table with the `ON DELETE CASCADE` annotation, then replacing a parent row also deletes the child rows. Otherwise, you must delete the child rows before you replace the parent row.                              |
| `delete`                                                                                                    | `object ( `[`Delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Delete)` )` Delete rows from a table. Succeeds whether or not the named rows were present.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |

## Write

Arguments to [`insert`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert) , [`update`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.update) , [`insertOrUpdate`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.insert_or_update) , and [`replace`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.replace) operations.

**JSON representation**

```
{
  "table": string,
  "columns": [
    string
  ],
  "values": [
    array
  ]
}
```

| Fields      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`     | `string` Required. The table whose rows will be written.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `columns[]` | `string` The names of the columns in [`table`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write.FIELDS.table) to be written. The list of columns must contain enough columns to allow Cloud Spanner to derive values for all primary key columns in the row(s) to be modified.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `values[]`  | `array ( `[`ListValue`](https://protobuf.dev/reference/protobuf/google.protobuf/#list-value)` format)` The values to be written. `values` can contain more than one list of values. If it does, then multiple rows are written, one for each entry in `values` . Each list in `values` must have exactly as many entries as there are entries in [`columns`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write.FIELDS.columns) above. Sending multiple lists is equivalent to sending multiple `Mutation` s, each containing one `values` entry and repeating [`table`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write.FIELDS.table) and [`columns`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Write.FIELDS.columns) . Individual values in each list are encoded as described [`here`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode) . |

## Delete

Arguments to [`delete`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#FIELDS.delete) operations.

**JSON representation**

```
{
  "table": string,
  "keySet": {
    object (KeySet)
  }
}
```

| Fields   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table`  | `string` Required. The table whose rows will be deleted.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `keySet` | `object ( `[`KeySet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/KeySet)` )` Required. The primary keys of the rows within [`table`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Mutation#Delete.FIELDS.table) to delete. The primary keys must be specified in the order in which they appear in the `PRIMARY KEY()` clause of the table's equivalent DDL statement (the DDL statement used to create the table). Delete is idempotent. The transaction will succeed even if some or all rows do not exist. |
