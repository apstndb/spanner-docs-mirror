Data Manipulation Language (DML) and Mutations are two APIs in Spanner that you can use to modify data. Each offer similar data manipulation features. This page compares both approaches.

## What is Data Manipulation Language (DML)?

The Data Manipulation Language (DML) in Spanner lets you manipulate data in your database tables using `  INSERT  ` , `  UPDATE  ` , and `  DELETE  ` statements. You can run DML statements using the [client libraries](/spanner/docs/reference/libraries) , the [Google Cloud console](/spanner/docs/create-query-database-console#run_a_query) , and [gcloud spanner](/sdk/gcloud/reference/spanner#execute_sql_statements) .

Spanner offers the following two implementations of DML execution, each with different properties.

  - **Standard DML** - suitable for standard [Online Transaction Processing (OLTP)](https://en.wikipedia.org/wiki/Online_transaction_processing) workloads.
    
    For more information, including code samples, see [Using DML](/spanner/docs/dml-tasks#using-dml)

  - **Partitioned DML** - designed for bulk updates and deletes as in the following examples.
    
      - Periodic cleanup and garbage collection. Examples are deleting old rows or setting columns to NULL.
    
      - Backfilling new columns with default values. An example is using an UPDATE statement to set a new column's value to False where it is NULL.
    
    For more information, including code samples, see [Using Partitioned DML](/spanner/docs/dml-tasks#partitioned-dml) .
    
    You can use batch writes for a large number of write operations without read operations that don't require atomic transactions. For more information, see [Modify data using batch writes](/spanner/docs/batch-write) .

## What are mutations?

A mutation represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a database. You can include operations that apply to different rows, or different tables, in a mutation. After you define one or more mutations that contain one or more writes, you must apply the mutation to commit the write(s). Each change is applied in the order in which they were added to the mutation.

For more information, including code samples, see [Inserting, updating, and deleting data using mutations](/spanner/docs/modify-mutation-api) .

## Feature comparison between DML and mutations

The following table summarizes DML and mutation support of common database operation and features.

<table>
<thead>
<tr class="header">
<th>Operations</th>
<th style="text-align: center;">DML</th>
<th style="text-align: center;">Mutations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Insert Data</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Supported</td>
</tr>
<tr class="even">
<td>Delete Data</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Supported</td>
</tr>
<tr class="odd">
<td>Update Data</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Supported</td>
</tr>
<tr class="even">
<td>Insert or Ignore Data</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Unsupported</td>
</tr>
<tr class="odd">
<td>Read Your Writes (RYW)</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Unsupported</td>
</tr>
<tr class="even">
<td>Insert or Update Data (Upsert)</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Supported</td>
</tr>
<tr class="odd">
<td>SQL Syntax</td>
<td style="text-align: center;">Supported</td>
<td style="text-align: center;">Unsupported</td>
</tr>
<tr class="even">
<td>Constraint checking</td>
<td style="text-align: center;">After every statement</td>
<td style="text-align: center;">At commit time</td>
</tr>
</tbody>
</table>

DML and mutations diverge in their support for the following features:

  - **Read Your Writes** : Reading uncommitted results within an active transaction. Changes you make using DML statements are visible to subsequent statements in the same transaction. This is different from using mutations, where changes are not visible in any reads (including reads done in the same transaction) until the transaction commits. This is because mutations in a transaction are buffered client-side (locally) and sent to the server as part of the commit operation. As a result, mutations in the commit request are not visible to SQL or DML statements within the same transaction.

  - **Constraint Checking** : Spanner checks constraints after every DML statement. This is different from using mutations, where Spanner buffers mutations in the client until commit and checks constraints at commit time. Evaluating constraints after each DML statement allows Spanner to guarantee that the data returned by a subsequent query in the same transaction returns data that is consistent with the schema.

  - **SQL Syntax** : DML provides a conventional way to manipulate data. You can reuse SQL skills to alter the data using the DML API.

## Best practice - avoid mixing DML and mutation in the same transaction

If a transaction contains both DML statements and mutations in the commit request, Spanner executes the DML statements before the mutations. To avoid having to account for the order of execution in your client library code, you should use either DML statements or the mutations in a single transaction, but not both.

The following Java example illustrates potentially surprising behavior. The code inserts two rows into Albums using the Mutation API. The snippet, then calls `  executeUpdate()  ` to update the newly inserted rows and calls `  executeQuery()  ` to read updated albums.

``` text
static void updateMarketingBudget(DatabaseClient dbClient) {
  dbClient
      .readWriteTransaction()
      .run(
          new TransactionCallable<Void>() {
            @Override
            public Void run(TransactionContext transaction) throws Exception {
               transaction.buffer(
                    Mutation.newInsertBuilder("Albums")
                        .set("SingerId")
                        .to(1)
                        .set("AlbumId")
                        .to(1)
                        .set("AlbumTitle")
                        .to("Total Junk")
                        .set("MarketingBudget")
                        .to(800)
                        .build());
               transaction.buffer(
                    Mutation.newInsertBuilder("Albums")
                        .set("SingerId")
                        .to(1)
                        .set("AlbumId")
                        .to(2)
                        .set("AlbumTitle")
                        .to("Go Go Go")
                        .set("MarketingBudget")
                        .to(200)
                        .build());

                // This UPDATE will not include the Albums inserted above.
                String sql =
                  "UPDATE Albums SET MarketingBudget = MarketingBudget * 2"
                      + " WHERE SingerId = 1";
                long rowCount = transaction.executeUpdate(Statement.of(sql));
                System.out.printf("%d records updated.\n", rowCount);

                // Read a newly updated record.
                sql =
                  "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
                      + " WHERE SingerId = 1 AND MarketingBudget < 1000";
                ResultSet resultSet =
                                 transaction.executeQuery(Statement.of(sql));
                while (resultSet.next()) {
                   System.out.printf(
                        "%s %s\n",
                        resultSet.getString("FirstName"),
                        resultSet.getString("LastName"));
                }
                return null;
              }
            });
}
```

If you were to execute this code, you'd see *0 records updated* . Why? This happens because the changes we made using Mutations are not visible to subsequent statements until the transaction commits. Ideally, we should have buffered writes only at the very end of the transaction.

## What's next?

  - Learn how to modify data [Using DML](/spanner/docs/dml-tasks#using-dml) .

  - Learn how to modify data [Using mutations](/spanner/docs/modify-mutation-api) .

  - To find the mutation count for a transaction, see [Retrieving commit statistics for a transaction](/spanner/docs/commit-statistics) .

  - Learn about [Data Manipulation Language (DML) best practices](/spanner/docs/dml-best-practices) .
