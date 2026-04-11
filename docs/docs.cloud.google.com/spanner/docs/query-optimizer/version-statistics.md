Spanner provides a built-in `SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS` table to keep track of query optimizer versions. You can retrieve this data using SQL queries.

## `SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS` table schema

| Column name    | Type    | Description                                 |
| -------------- | ------- | ------------------------------------------- |
| `VERSION`      | `INT64` | The optimizer version.                      |
| `RELEASE_DATE` | `DATE`  | The release date of the optimizer version.  |
| `IS_DEFAULT`   | `BOOL`  | Whether the version is the default version. |

### List all supported optimizer versions

    SELECT * FROM SPANNER_SYS.SUPPORTED_OPTIMIZER_VERSIONS

An example result:

| VERSION | RELEASE\_DATE | IS\_DEFAULT |
| ------- | ------------- | ----------- |
| 1       | 2019-06-18    | false       |
| 2       | 2020-03-01    | false       |
| 3       | 2021-08-01    | true        |

## What's next

  - To learn more about the query optimizer, see [Query optimizer overview](https://docs.cloud.google.com/spanner/docs/query-optimizer/overview) .
  - To learn more about how the query optimizer has evolved, see [Query optimizer versions](https://docs.cloud.google.com/spanner/docs/query-optimizer/versions) .
