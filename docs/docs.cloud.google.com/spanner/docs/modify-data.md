The Google Cloud console provides an interface for inserting, editing, and deleting data in a Spanner table.

## Find a table

To get started, select to view a Spanner table in the Google Cloud console.

1.  Go to the **Spanner** page in the Google Cloud console.

2.  Click the name of an instance. The instance overview page is displayed.

3.  Under the **Databases** list on this page, click the name of a database.

4.  Under **Tables** , click the name of a table.

5.  In the left pane of the Google Cloud console, click **Data** .

## Insert data

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

1.  On the table's **Data** page, click **Insert** .
    
    The Google Cloud console displays the table's Spanner Studio page with a new query tab containing template `  INSERT  ` and `  SELECT  ` statements that you edit to insert a row in the table and view the result of that insertion.

2.  Edit the `  INSERT  ` statement to the values you want, and edit the `  SELECT  ` statement's `  WHERE  ` clause to match the primary key value of the row you are inserting.
    
    See [INSERT statement](/spanner/docs/reference/standard-sql/dml-syntax#insert-statement) and [Literals](/spanner/docs/reference/standard-sql/lexical#literals) for information about the syntax Spanner supports.

3.  Click **Run** .
    
    Spanner runs the statements.

## Edit data

1.  On the table's **Data** page, select the row you want to edit, then click **Edit** .
    
    The Google Cloud console displays the table's **Spanner Studio** page with a new query tab containing template `  UPDATE  ` and `  SELECT  ` statements that you edit to update the row in the table and view the result of that update. Note that the `  WHERE  ` clauses of both statements denote the row you selected to edit.

2.  Edit the `  UPDATE  ` statement to reflect the updates you want to make.
    
    See [UPDATE statement](/spanner/docs/reference/standard-sql/dml-syntax#update-statement) and [Literals](/spanner/docs/reference/standard-sql/lexical#literals) for information about the syntax Spanner supports.

3.  Click **Run** .
    
    Spanner runs the statements.

## Delete data

**Note:** Deleting a row will also delete its child rows if the child rows have `  ON DELETE CASCADE  ` enabled. However, if the child rows do not have this setting enabled, you will first have to delete all the child rows in order to delete the parent row.

1.  On the table's **Data** page, select one or more rows that you want to delete, then click **Delete** .
    
    Need help finding a row? Type its primary key into the filter box.

2.  In the dialog that appears after you click **Delete** , click **CONFIRM** .
    
    The Google Cloud console displays the data from your table, which no longer contains the deleted rows.

For an interactive example of inserting and modifying data in a Spanner table, see the [Quickstart using the console](/spanner/docs/create-query-database-console) .
