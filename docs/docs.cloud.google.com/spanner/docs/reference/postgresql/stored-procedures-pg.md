A *stored procedure* is a collection of statements that can be called from other queries or other stored procedures. Some stored procedures are *system procedures* that are built into Spanner. System procedures don't need to be created. Spanner lets you use system procedures.

Because you can't use Spanner to create a stored procedure, Spanner doesn't support stored procedures that aren't built in system procedures.

## Run a stored system procedure

To run a stored system procedure, you use the [`CALL`](https://docs.cloud.google.com/spanner/docs/reference/postgresql/query-syntax#call) statement:

    CALL procedure_name(parameters);

Replace procedure\_name with the name of the stored system procedure. You can run one stored system procedure at a time.

## Stored system procedures

Spanner supports the following stored system procedure:

  - [Query cancellation](https://docs.cloud.google.com/spanner/docs/reference/postgresql/stored-procedures-pg#query-cancellation)
  - [Major compaction](https://docs.cloud.google.com/spanner/docs/reference/postgresql/stored-procedures-pg#major-compaction)

### Query cancellation

This section describes the query cancellation stored system procedure.

#### Syntax

The `cancel_query` stored system procedure cancels a query. You specify the query to cancel using its `query_id` .

    CALL spanner.cancel_query(query_id)

#### Description

The `cancel_query` stored system procedure has the following parameters:

| Parameter  | Type     | Description                                             |
| ---------- | -------- | ------------------------------------------------------- |
| `query_id` | `STRING` | Specifies the ID for the query that you want to cancel. |

Query cancellations might fail because of one or more of the following:

  - The Spanner servers are busy due to heavy query loads.
  - The query is in the process of restarting due to an error.

If one of these causes the query to fail, you can run the query cancellation stored system procedure again.

### Major Compaction

This section describes the major compaction stored system procedure.

#### Syntax

    CALL spanner.compact_all()

#### Description

The `compact_all` stored procedure initiates a major compaction long-running operation to compact all of the data in the database. The procedure returns an identifier that can be used to query the status of the compaction. See [Manual Data Compaction](https://docs.cloud.google.com/spanner/docs/manual-data-compaction) for more details.
