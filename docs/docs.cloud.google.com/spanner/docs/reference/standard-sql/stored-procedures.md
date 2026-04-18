This section describes stored system procedures for Spanner.

A stored system procedure contains SQL code that you can reuse. Spanner provides stored system procedures for you to use. You can't create your own stored procedure in Spanner. You can only execute one stored procedure at a time in a `CALL` statement.

## Stored system procedures

To execute a stored system procedure, you use the [`CALL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/procedural-language#call) statement:

    CALL procedure_name(parameters);

Replace procedure\_name with the name of the stored system procedure.

Spanner supports the following stored system procedures:

  - [Query cancellation](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/stored-procedures#query-cancellation)
  - [Major compaction](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/stored-procedures#major-compaction)

### Query cancellation

This section describes the query cancellation stored system procedure.

#### Syntax

Cancels a query with the specified ***query\_id*** .

    CALL cancel_query(query_id)

#### Description

This stored system procedure has the following parameters:

| Parameter  | Type     | Description                                             |
| ---------- | -------- | ------------------------------------------------------- |
| `query_id` | `STRING` | Specifies the ID for the query that you want to cancel. |

Query cancellations might fail in the following circumstances:

  - When Spanner servers are busy due to heavy query loads.
  - When the query is in the [process of restarting](https://docs.cloud.google.com/spanner/docs/introspection/oldest-active-queries#limitations) due to an error.

In both cases, you can run the query cancellation stored system procedure again.

### Major Compaction

This section describes the major compaction stored system procedure.

#### Syntax

    CALL compact_all()

#### Description

The `compact_all` stored procedure initiates a major compaction long-running operation to compact all of the data in the database. The procedure returns an identifier that can be used to query the status of the compaction. See [Manual Data Compaction](https://docs.cloud.google.com/spanner/docs/manual-data-compaction) for more details.
