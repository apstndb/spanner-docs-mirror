---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionSelector
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionSelector
title: TransactionSelector
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionSelector#SCHEMA_REPRESENTATION)

This message is used to select the transaction in which a [`sessions.read`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#google.spanner.v1.Spanner.Read) or [`ExecuteSql`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#google.spanner.v1.Spanner.ExecuteSql) call runs.

See [`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions) for more information about transactions.

**JSON representation**

```
{

  // Union field selector can be only one of the following:
  "singleUse": {
    object (TransactionOptions)
  },
  "id": string,
  "begin": {
    object (TransactionOptions)
  }
  // End of list of possible types for union field selector.
}
```

| Fields                                                                                                                                                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Union field `selector` . If no fields are set, the default is a single use transaction with strong concurrency. `selector` can be only one of the following: |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `singleUse`                                                                                                                                                  | `object ( `[`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions)` )` Execute the read or SQL query in a temporary transaction. This is the most efficient way to execute a transaction that consists of a single SQL query.                                                                                                                                                                                                                   |
| `id`                                                                                                                                                         | `string ( `[`bytes`](https://developers.google.com/discovery/v1/type-format)` format)` Execute the read or SQL query in a previously-started transaction. A base64-encoded string.                                                                                                                                                                                                                                                                                                              |
| `begin`                                                                                                                                                      | `object ( `[`TransactionOptions`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/TransactionOptions)` )` Begin a new transaction and execute this read or SQL query in it. The transaction ID of the new transaction is returned in [`ResultSetMetadata.transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata#FIELDS.transaction) , which is a [`Transaction`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/Transaction) . |
