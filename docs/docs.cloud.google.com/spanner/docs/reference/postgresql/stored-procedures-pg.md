A *stored procedure* is a collection of statements that can be called from other queries or other stored procedures. Some stored procedures are *system procedures* that are built into Spanner. System procedures don't need to be created. Spanner lets you use system procedures.

Because you can't use Spanner to create a stored procedure, Spanner doesn't support stored procedures that aren't built in system procedures.

## Run a stored system procedure

To run a stored system procedure, you use the [`  CALL  `](/spanner/docs/reference/postgresql/query-syntax#call) statement:

``` text
CALL procedure_name(parameters);
```

Replace procedure\_name with the name of the stored system procedure. You can run one stored system procedure at a time.

## Stored system procedures

Spanner supports the following stored system procedure:

  - [Query cancellation](#query-cancellation)

### Query cancellation

This section describes the query cancellation stored system procedure.

#### Syntax

The `  cancel_query  ` stored system procedure cancels a query. You specify the query to cancel using its `  query_id  ` .

``` text
CALL spanner.cancel_query(query_id)
```

#### Description

The `  cancel_query  ` stored system procedure has the following parameters:

<table>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       query_id      </code></td>
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td>Specifies the ID for the query that you want to cancel.</td>
</tr>
</tbody>
</table>

Query cancellations might fail because of one or more of the following:

  - The Spanner servers are busy due to heavy query loads.
  - The query is in the process of restarting due to an error.

If one of these causes the query to fail, you can run the query cancellation stored system procedure again.
