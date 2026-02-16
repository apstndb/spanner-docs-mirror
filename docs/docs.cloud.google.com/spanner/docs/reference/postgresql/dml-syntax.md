This page defines the syntax of the SQL data manipulation language (DML) statements supported for PostgreSQL-dialect databases.

## Notations used in the syntax

  - Square brackets `  [ ]  ` indicate optional clauses.
  - Curly braces `  { }  ` enclose a set of options.
  - The vertical bar `  |  ` indicates a logical OR.
  - A comma followed by an ellipsis indicates that the preceding `  item  ` can repeat in a comma-separated list.
      - `  item [, ...]  ` indicates one or more items, and
      - `  [item, ...]  ` indicates zero or more items.
  - Purple-colored text, such as `  item  ` , marks Spanner extensions to open source PostgreSQL.
  - Parentheses `  ( )  ` indicate literal parentheses.
  - A comma `  ,  ` indicates the literal comma.
  - Angle brackets `  <>  ` indicate literal angle brackets.
  - Uppercase words, such as `  INSERT  ` , are keywords.

## INSERT statement

Use the `  INSERT  ` statement to add new rows to a table.

``` text
INSERT INTO table_name [ AS alias ]
    [ ( column_name [, ...] ) ]
    { VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
    [ ON CONFLICT (conflict_target) conflict_action ]
    [ RETURNING select-list ]

where conflict_target must be the primary key column(s) in order
and separated by columns.

and conflict_action is one of:

    DO NOTHING
    DO UPDATE SET { column_name [, ...] = excluded.column_name [, ...] }

and where query is:
    a query (SELECT statement) that supplies the rows to be inserted.

and select-list is:

    { * | expression [ [ AS ] output_name ] [, ...] }
```

See [PostgreSQL queries](/spanner/docs/reference/postgresql/query-syntax) for a description of the `  SELECT  ` syntax.

**Note:** Hints are not supported for ***query*** in an `  INSERT  ` statement.

### Default values

Use the `  DEFAULT  ` keyword to insert the default value of a column. If a column is not included in the column name list, Spanner assigns the default value of the column with little computing overhead. If the column has no defined default value, `  NULL  ` is assigned to the column.

The use of default values is subject to current Spanner limits, including the mutation limit. If a column has a default value and it is used in an insert or update, the column is counted as one mutation. For example, assuming that table `  T  ` has three columns and that `  col_a  ` has a default value, the following inserts each result in three mutations:

``` text
INSERT INTO T (id, col_a, col_b) VALUES (1, DEFAULT, 1);
INSERT INTO T (id, col_a, col_b) VALUES (2, 200, 2);
INSERT INTO T (id, col_b) VALUES (3, 3);
```

For more information about default column values, see the `  DEFAULT ( expression )  ` clause in `  CREATE TABLE  ` .

For more information about mutations, see [What are mutations?](../../dml-versus-mutations.md#mutations-concept) .

### `     ON CONFLICT    ` clause

The `  ON CONFLICT DO NOTHING  ` clause indicates that if the row that you're inserting already exists in the table, `  INSERT  ` doesn't throw a unique constraint violation of the primary key, and the row isn't inserted.

The `  ON CONFLICT DO UPDATE SET  ` clause indicates that if the row that you're inserting already exists in the table, `  INSERT  ` doesn't throw a unique constraint violation of the primary key, and the row is updated.

This clause has a few differences from PostgreSQL, resulting in the following restrictions:

  - Only permits primary key columns as the ***conflict\_target*** .

  - If the table has composite primary keys, you must specify all primary key columns in ***conflict\_target*** .

  - The `  DO UPDATE SET  ` clause must list all columns that you specify in the insert list.

  - For `  DO UPDATE SET  ` , the update value must be set to the insert value in the query using the PostgreSQL special alias `  excluded  ` . For example:
    
    ``` text
    INSERT INTO singers (SingerId, FirstName, LastName)
    VALUES (1, 'Marc', 'Richards')
    ON CONFLICT (SingerId)
    DO UPDATE SET SingerId = excluded.SingerId, FirstName = excluded.FirstName, LastName = excluded.LastName;
    ```

  - The `  WHERE  ` clause in `  ON CONFLICT DO UPDATE  ` clause isn't supported.

#### Examples for `     ON CONFLICT DO UPDATE SET    `

For the following table:

``` text
CREATE TABLE Singers (
  SingerId int primary key,
  FirstName varchar(64),
  LastName varchar(64),
  Birthdate date,
  Status varchar(64),
  SingerInfo varchar(64)
);
```

You can use the following query without a columns list:

``` text
INSERT INTO Singers
VALUES (5, 'Zak', 'Sterling', '1996-03-12', 'active', 'nationality:"U.S.A."'),
       (7, 'Edie', 'Silver', '1998-01-23', 'active', 'nationality:"U.S.A."')
ON CONFLICT (SingerId)
DO UPDATE SET SingerId=excluded.SingerId,
              FirstName=excluded.FirstName,
              LastName=excluded.LastName,
              Birthdate=excluded.Birthdate,
              Status=excluded.Status,
              SingerInfo=excluded.SingerInfo;
```

Or you can use the following query with a columns list:

``` text
INSERT INTO Singers
    (SingerId, LastName)
VALUES (5, 'Sterling'),
       (7, 'Silver')
ON CONFLICT (SingerId)
DO UPDATE SET SingerId=excluded.SingerId, LastName=excluded.LastName;
```

### RETURNING

Use the `  RETURNING  ` clause to return the results of the `  INSERT  ` operation and selected data from the newly inserted rows. This clause is especially useful for retrieving values of columns with default values, generated columns, and auto-generated keys, without having to use additional `  SELECT  ` statements.

The `  RETURNING  ` clause can capture expressions based on newly inserted rows that include the following:

  - `  *  ` : Returns all columns.
  - `  expression  ` : Represents a column name of the table specified by table\_name or an expression that uses any combination of such column names. Column names are valid if they belong to columns of the table\_name. Excluded expressions include aggregate and analytic functions.
  - `  alias  ` : Represents a temporary name for an expression in the query.

For example, the following query inserts two rows into the `  Singers  ` table, uses `  RETURNING  ` to fetch the SingerId column from these rows, and computes a new column called `  FullName  ` .

``` text
INSERT INTO singers (SingerId, FirstName, LastName)
VALUES
    (7, 'Melissa', 'Garcia'),
    (8, 'Russell', 'Morales')
RETURNING SingerId, FirstName || ' ' || LastName AS FullName;
```

In the following query, we use `  ON CONFLICT DO UPDATE SET  ` is used to update existing rows if there is a conflict for `  SingerId  ` .

``` text
INSERT INTO singers (SingerId, FirstName, LastName)
VALUES
    (7, 'Melissa', 'Garcia'),
    (8, 'Russell', 'Morales')
ON CONFLICT (SingerId) DO UPDATE SET
    SingerId = EXCLUDED.SingerId,
    FirstName = EXCLUDED.FirstName,
    LastName = EXCLUDED.LastName
RETURNING SingerId, FirstName || ' ' || LastName AS FullName;
```

For instructions and code samples, see [Modify data with the returning DML statements](/spanner/docs/dml-tasks#client-library-dml-return) .

## DELETE statement

Use the `  DELETE  ` statement to delete rows from a table.

``` text
[ /* @statement_hint_expr [, ...] */ ] DELETE FROM table_name [ /* @table_hint_expr [, ...] */ ]
    [ [ AS ] alias ]
    [ WHERE condition ]
    [ RETURNING select-list ]

where select-list is:
    { * | expression [ [ AS ] output_name ] [, ...] }


and statement_hint_expr is:

    statement_hint_key = statement_hint_value

and table_hint_expr is:

    table_hint_key = table_hint_value
```

### RETURNING

With the optional `  RETURNING  ` clause, you can obtain data from rows that are being deleted in a table. For example, the following query deletes all rows in the `  Singers  ` table that contains a singer called `  Melissa  ` and returns the deleted rows.

``` text
DELETE FROM Singers WHERE Firstname = 'Melissa'
RETURNING *;
```

To learn more about the values that you can use in this clause, see [INSERT RETURNING](#insert-returning) .

## UPDATE statement

Use the `  UPDATE  ` statement to update existing rows in a table.

``` text
[ /* @statement_hint_expr [, ...] */ ] UPDATE [ ONLY ] table_name [ /* @table_hint_expr [, ...] */ ] [ * ]
    [ [ AS ] alias ]
    SET {
        column_name = { expression | DEFAULT } |
        ( column_name [, ...] ) = [ ROW ] ( { expression | DEFAULT } [, ...] )
    } [, ...]
    [ WHERE condition ]
    [ RETURNING select-list ]

where select-list is:
    { * | expression [ [ AS ] output_name ] [, ...] }


and statement_hint_expr is:

    statement_hint_key = statement_hint_value

and table_hint_expr is:

    table_hint_key = table_hint_value
```

Where:

  - `  table_name  ` is the name of a table to update.

  - The `  SET  ` clause specifies how to modify columns in the rows that match the `  WHERE  ` clause's condition. It's a list of column names or expression pairs.

  - `  expression  ` is an update expression. The expression can be a literal, a SQL expression, or a SQL subquery.

  - `  statement_hint_expr  ` is a statement-level hint. The following hints are supported:
    
    <table>
    <colgroup>
    <col style="width: 33%" />
    <col style="width: 33%" />
    <col style="width: 33%" />
    </colgroup>
    <thead>
    <tr class="header">
    <th><code dir="ltr" translate="no">         statement_hint_key        </code></th>
    <th><code dir="ltr" translate="no">         statement_hint_value        </code></th>
    <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>PDML_MAX_PARALLELISM</td>
    <td>An integer between 1 to 1000</td>
    <td>Sets the maximum parallelism for <a href="/spanner/docs/dml-partitioned">Partitioned DML</a> queries.<br />
    This hint is only valid with the <a href="/spanner/docs/dml-partitioned">Partitioned DML</a> query execution mode.</td>
    </tr>
    </tbody>
    </table>

  - `  table_hint_expr  ` is a hint for accessing the table. The following hints are supported:
    
    <table>
    <thead>
    <tr class="header">
    <th><code dir="ltr" translate="no">         table_hint_key        </code></th>
    <th><code dir="ltr" translate="no">         table_hint_value        </code></th>
    <th>Description</th>
    </tr>
    </thead>
    <tbody>
    <tr class="odd">
    <td>FORCE_INDEX</td>
    <td>Index name</td>
    <td>Use specified index when querying rows to be updated.</td>
    </tr>
    <tr class="even">
    <td>FORCE_INDEX</td>
    <td>_BASE_TABLE</td>
    <td>Don't use an index when querying. Instead, scan the base table.</td>
    </tr>
    </tbody>
    </table>

`  UPDATE  ` statements must comply with the following rules:

  - A column can appear only once in the `  SET  ` clause.
  - The columns in the `  SET  ` clause can be listed in any order.
  - Each value must be type compatible with its associated column.
  - The values must comply with any constraints in the schema, such as unique secondary indexes or non-nullable columns.
  - Updates with joins are not supported.
  - You cannot update primary key columns.

### Default values

The `  DEFAULT  ` keyword sets the value of a column to its default value. If the column has no defined default value, the `  DEFAULT  ` keyword sets it to `  NULL  ` .

The use of default values is subject to current Spanner limits, including the mutation limit. If a column has a default value and it is used in an insert or update, the column is counted as one mutation. For example, assume that in table `  T  ` , `  col_a  ` has a default value. The following updates each result in two mutations. One comes from the primary key, and another comes from either the explicit value (1000) or the default value.

``` text
UPDATE T SET col_a = 1000 WHERE id=1;
UPDATE T SET col_a = DEFAULT WHERE id=3;
```

For more information about default column values, see the `  DEFAULT ( expression )  ` clause in `  CREATE TABLE  ` .

For more information about mutations, see [What are mutations?](../../dml-versus-mutations.md#mutations-concept) .

### WHERE clause

The `  WHERE  ` clause is required. This requirement can help prevent accidentally updating all the rows in a table. To update all rows in a table, set the `  condition  ` to `  true  ` .

The `  WHERE  ` clause can contain any valid SQL boolean expression, including a subquery that refers to other tables.

### Aliases

The `  WHERE  ` clause has an implicit alias to `  target_name  ` . This alias lets you reference columns in `  target_name  ` without qualifying them with `  target_name  ` . For example, if your statement starts with `  UPDATE Singers  ` , then you can access any columns of `  Singers  ` in the `  WHERE  ` clause. In this example, `  FirstName  ` and `  LastName  ` are columns in the `  Singers  ` table:

``` text
UPDATE singers
SET birthdate = '1990-10-10'
WHERE firstname = 'Marc' AND lastname = 'Richards';
```

You can also create an explicit alias using the optional `  AS  ` keyword.

### RETURNING

With the optional `  RETURNING  ` clause, you can obtain data from rows that are being updated in a table. For example, the following query updates all rows where the singer first name is equal to `  Russell  ` and returns the updated rows.

``` text
UPDATE Singers
SET BirthDate = '1990-10-10'
WHERE FirstName = 'Russell'
RETURNING *;
```

To learn more about the values that you can use in this clause, see [INSERT RETURNING](#insert-returning) .
