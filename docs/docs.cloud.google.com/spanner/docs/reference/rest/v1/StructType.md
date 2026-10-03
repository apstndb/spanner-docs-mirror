---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType
title: StructType
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#SCHEMA_REPRESENTATION)
- [Field](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#Field)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#Field.SCHEMA_REPRESENTATION)

`StructType` defines the fields of a [`STRUCT`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type#TypeCode.ENUM_VALUES.STRUCT) type.

**JSON representation**

```
{
  "fields": [
    {
      object (Field)
    }
  ]
}
```

| Fields     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `fields[]` | `object ( `[`Field`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType#Field)` )` The list of fields that make up this struct. Order is significant, because values of this struct type are represented as lists, where the order of field values matches the order of fields in the [`StructType`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/StructType) . In turn, the order of fields matches the order of columns in a read request, or the order of fields in the `SELECT` clause of a query. |

## Field

Message representing a single field of a struct.

**JSON representation**

```
{
  "name": string,
  "type": {
    object (Type)
  }
}
```

| Fields |                                                                                                                                                                                                                                                                                                                                                                                                               |
|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name` | `string` The name of the field. For reads, this is the column name. For SQL queries, it is the column alias (e.g., `"Word"` in the query `"SELECT 'hello' AS Word"` ), or the column name (e.g., `"ColName"` in the query `"SELECT ColName FROM Table"` ). Some columns might have an empty name (e.g., `"SELECT UPPER(ColName)"` ). Note that a query result can contain multiple fields with the same name. |
| `type` | `object ( `[`Type`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Type)` )` The type of the field.                                                                                                                                                                                                                                                                                             |
