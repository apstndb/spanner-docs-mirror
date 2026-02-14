A generated column is a column that is always computed from other columns in a row. These columns can make a query simpler, save the cost of evaluating an expression at query time, and can be indexed or used as a foreign key. This page describes how to manage this column type in your database for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Add a generated column to a new table

In the following `  CREATE TABLE  ` snippet, we create a table to store information about users. We have columns for `  FirstName  ` and `  LastName  ` and define a generated column for `  FullName  ` , which is the concatenation of `  FirstName  ` and `  LastName  ` . The SQL in parentheses is called the *generation expression* .

A generated column can be marked as `  STORED  ` to save the cost of evaluating the expression at query time. As a result, the value of `  FullName  ` is only computed when a new row is inserted or when `  FirstName  ` or `  LastName  ` is updated for an existing row. The computed value is stored along with other columns in the table.

### GoogleSQL

``` text
CREATE TABLE Users (
Id STRING(20) NOT NULL,
FirstName STRING(50),
LastName STRING(50),
Age INT64 NOT NULL,
FullName STRING(100) AS (FirstName || ' ' || LastName) STORED
) PRIMARY KEY (Id);
```

### PostgreSQL

``` text
CREATE TABLE users (
id VARCHAR(20) NOT NULL,
firstname VARCHAR(50),
lastname VARCHAR(50),
age BIGINT NOT NULL,
fullname VARCHAR(100) GENERATED ALWAYS AS (firstname || ' ' || lastname) STORED,
PRIMARY KEY(id)
);
```

You can create a non-stored generated column by omitting the `  STORED  ` attribute in the DDL. This kind of generated column is evaluated at query time and can make a query simpler. In PostgreSQL, you can create a non-stored generated column using the `  VIRTUAL  ` attribute.

### GoogleSQL

``` text
FullName STRING(MAX) AS (CONCAT(FirstName, " ", LastName))
```

### PostgreSQL

``` text
fullname text GENERATED ALWAYS AS (firstname || ' ' || lastname) VIRTUAL
```

  - `  expression  ` can be any valid SQL expression that's assignable to the column data type with the following restrictions.
    
      - The expression can only reference columns in the same table.
    
      - The expression can't contain [subqueries](/spanner/docs/reference/standard-sql/subqueries) .
    
      - Expressions with non-deterministic functions such as [`  PENDING_COMMIT_TIMESTAMP()  `](/spanner/docs/reference/standard-sql/timestamp_functions#pending_commit_timestamp) , [`  CURRENT_DATE()  `](/spanner/docs/reference/standard-sql/date_functions#current_date) , and [`  CURRENT_TIMESTAMP()  `](/spanner/docs/reference/standard-sql/timestamp_functions#current_timestamp) can't be made into a `  STORED  ` generated column or a generated column that is indexed.
    
      - You can't modify the expression of a `  STORED  ` or indexed generated column.

  - For GoogleSQL-dialect databases, a non-stored generated column of type `  STRING  ` or `  BYTES  ` must have a length of `  MAX  ` .

  - For PostgreSQL-dialect databases, a non-stored, or virtual, generated column of type `  VARCHAR  ` must have a length of `  MAX  ` .

  - The `  STORED  ` attribute that follows the expression stores the result of the expression along with other columns of the table. Subsequent updates to any of the referenced columns cause Spanner to re-evaluate and store the expression.

  - Generated columns that are not `  STORED  ` can't be marked as `  NOT NULL  ` .

  - Direct writes to generated columns aren't allowed.

  - Column option `  allow_commit_timestamp  ` isn't allowed on generated columns or any columns that generated columns reference.

  - For `  STORED  ` or generated columns that are indexed, you can't change the data type of the column, or of any columns that the generated column references.

  - You can't drop a column a generated column references.

  - You can use a generated column as a primary key with the following additional restrictions:
    
      - The generated primary key can't reference other generated columns.
    
      - The generated primary key can reference, at most, one non-key column.
    
      - The generated primary key can't depend on a non-key column with a `  DEFAULT  ` clause.

  - The following rules apply when using generated key columns:
    
      - Read APIs: You must fully specify the key columns, including the generated key columns.
      - Mutation APIs: For `  INSERT  ` , `  INSERT_OR_UPDATE  ` , and `  REPLACE  ` , Spanner doesn't allow you to specify generated key columns. For `  UPDATE  ` , you can optionally specify generated key columns. For `  DELETE  ` , you need to fully specify the key columns including the generated keys.
      - DML: You can't explicitly write to generated keys in `  INSERT  ` or `  UPDATE  ` statements.
      - Query: In general, we recommend that you use the generated key column as a filter in your query. Optionally, if the expression for the generated key column uses only one column as a reference, the query can apply an equality ( `  =  ` ) or `  IN  ` condition to the referenced column. For more information and an example, see [Create a unique key derived from a value column](/spanner/docs/generated-column/how-to#primary-key-generated-column) .

The generated column can be queried just like any other column, as shown in the following example.

### GoogleSQL

``` text
SELECT Id, FullName
FROM Users;
```

### PostgreSQL

``` text
SELECT id, fullname
FROM users;
```

The query using `  Fullname  ` is equivalent to the query with the generated expression. Hence a generated column can make the query simpler.

### GoogleSQL

``` text
SELECT Id, ARRAY_TO_STRING([FirstName, LastName], " ") as FullName
FROM Users;
```

### PostgreSQL

``` text
SELECT id, firstname || ' ' || lastname as fullname
FROM users;
```

## Create an index on a generated column

You can also index or use a generated column as a foreign key.

To help with lookups on our `  FullName  ` generated column, we can create a secondary index as shown in the following snippet.

### GoogleSQL

``` text
CREATE INDEX UsersByFullName ON Users (FullName);
```

### PostgreSQL

``` text
CREATE INDEX UserByFullName ON users (fullname);
```

## Add a generated column to an existing table

Using the following `  ALTER TABLE  ` statement, we can add a generated column to the `  Users  ` table to generate and store the user's initials.

### GoogleSQL

``` text
ALTER TABLE Users ADD COLUMN Initials STRING(2)
AS (ARRAY_TO_STRING([SUBSTR(FirstName, 0, 1), SUBSTR(LastName, 0, 1)], "")) STORED;
```

### PostgreSQL

``` text
ALTER TABLE users ADD COLUMN initials VARCHAR(2)
GENERATED ALWAYS AS (SUBSTR(firstname, 0, 1) || SUBSTR(lastname, 0, 1)) STORED;
```

If you add a stored generated column to an existing table, a long-running operation to backfill the column values is started. During backfilling, the stored generated columns can't be read or queried. The backfilling state is reflected in the INFORMATION\_SCHEMA table.

## Create a partial index using a generated column

What if we only wanted to query users who are over 18? A full scan of the table would be inefficient, so we use a partial index.

1.  Use the following statement to add another generated column which returns the user's age if they are over 18, and returns `  NULL  ` otherwise.
    
    ### GoogleSQL
    
    ``` text
    ALTER TABLE Users ADD COLUMN AgeAbove18 INT64
    AS (IF(Age > 18, Age, NULL));
    ```
    
    ### PostgreSQL
    
    ``` text
    ALTER TABLE Users ADD COLUMN AgeAbove18 BIGINT
    GENERATED ALWAYS AS (nullif( Age , least( 18, Age) )) VIRTUAL;
    ```

2.  Create an index on this new column, and disable the indexing of `  NULL  ` values with the `  NULL_FILTERED  ` keyword in GoogleSQL or the `  IS NOT NULL  ` predicate in PostgreSQL. This partial index is smaller and more efficient than a normal index because it excludes everyone who is 18 or younger.
    
    ### GoogleSQL
    
    ``` text
    CREATE NULL_FILTERED INDEX UsersAbove18ByAge
    ON Users (AgeAbove18);
    ```
    
    ### PostgreSQL
    
    ``` text
    CREATE INDEX UsersAbove18ByAge ON users (AgeAbove18)
    WHERE AgeAbove18 IS NOT NULL;
    ```

3.  To retrieve the `  Id  ` and `  Age  ` of all users who are over 18, run the following query.
    
    ### GoogleSQL
    
    ``` text
    SELECT Id, Age
    FROM Users@{FORCE_INDEX=UsersAbove18ByAge}
    WHERE AgeAbove18 IS NOT NULL;
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT Id, Age
    FROM users /*@ FORCE_INDEX = UsersAbove18ByAge */
    WHERE AgeAbove18 IS NOT NULL;
    ```

4.  To filter on a different age, for example, to retrieve all users who are over 21, use the same index and filter on the generated column as follows:
    
    ### GoogleSQL
    
    ``` text
    SELECT Id, Age
    FROM Users@{FORCE_INDEX=UsersAbove18ByAge}
    WHERE AgeAbove18 > 21;
    ```
    
    ### PostgreSQL
    
    ``` text
    SELECT Id, Age
    FROM users /*@ FORCE_INDEX = UsersAbove18ByAge */
    WHERE AgeAbove18 > 21;
    ```
    
    An indexed generated column can save the cost of evaluating an expression at query time and avoid storing the values twice (in the base table and index) as compared to a `  STORED  ` generated column.

## Remove a generated column

The following DDL statement drops a generated column from the `  Users  ` table:

### GoogleSQL

``` text
  ALTER TABLE Users DROP COLUMN Initials;
```

### PostgreSQL

``` text
  ALTER TABLE users DROP COLUMN initials;
```

## Modify a generated column expression

### GoogleSQL

``` text
ALTER TABLE Users ALTER COLUMN FullName STRING(100)
AS (ARRAY_TO_STRING(ARRAY_TO_STRING([LastName, FirstName ], " ")));
```

### PostgreSQL

``` text
ALTER TABLE users ADD COLUMN Initials VARCHAR(2)
GENERATED ALWAYS AS (lastname || ' ' || firstname) VIRTUAL;
```

Updating the expression of a `  STORED  ` generated column or an indexed non-stored generated column isn't allowed.

## Create a primary key on a generated column

In Spanner, you can use a `  STORED  ` generated column in the primary key.

The following example shows a DDL statement that creates the `  UserInfoLog  ` table with a `  ShardId  ` generated column. The value of the `  ShardId  ` column depends on another column. It's derived by using a `  MOD  ` function on the `  UserId  ` column. `  ShardId  ` is declared as part of the primary key.

### GoogleSQL

``` text
CREATE TABLE UserInfoLog (
  ShardId INT64 NOT NULL
  AS (MOD(UserId, 2048)) STORED,
  UserId INT64 NOT NULL,
  FullName STRING(1024) NOT NULL,
) PRIMARY KEY (ShardId, UserId);
```

### PostgreSQL

``` text
CREATE TABLE UserInfoLog (
  ShardId BIGINT GENERATED ALWAYS
  AS (MOD(UserId, '2048'::BIGINT)) STORED NOT NULL,
  UserId BIGINT NOT NULL,
  FullName VARCHAR(1024) NOT NULL,
  PRIMARY KEY(ShardId, UserId));
```

Normally, to efficiently access a specific row you need to specify all key columns. In the previous example, this would mean providing both a `  ShardId  ` and `  UserId  ` . However, Spanner can sometimes infer the value of the generated primary key column if it depends on a single other column and if the value of the column it depends on is fully determined. This is true if the column referenced by the generated primary key column meets one of the following conditions:

  - It's equal to a constant value or bound parameter in the `  WHERE  ` clause, or
  - It gets its value set by an `  IN  ` operator in the `  WHERE  ` clause
  - It gets its value from an equi-join condition

For example, for the following query:

### GoogleSQL

``` text
SELECT * FROM UserInfoLog
AS T WHERE T.UserId=1;
```

### PostgreSQL

``` text
SELECT * FROM UserInfoLog
AS T WHERE T.UserId=1;
```

Spanner can infer the value of `  ShardId  ` from the provided `  UserId  ` . The previous query is equivalent to the following query after query optimization:

### GoogleSQL

``` text
SELECT * FROM UserInfoLog
AS T WHERE T.ShardId = MOD(1, 2048)
AND T.UserId=1;
```

### PostgreSQL

``` text
SELECT * FROM UserInfoLog
AS T WHERE T.ShardId = MOD(1, 2048)
AND T.UserId=1;
```

The next example shows how to create the `  Students  ` table and use an expression that retrieves the `  id  ` field of the `  StudentInfo  ` JSON column and uses it as the primary key:

### GoogleSQL

``` text
CREATE TABLE Students (
  StudentId INT64 NOT NULL
  AS (INT64(StudentInfo.id)) STORED,
  StudentInfo JSON NOT NULL,
) PRIMARY KEY (StudentId);
```

### PostgreSQL

``` text
CREATE TABLE Students (
  StudentId BIGINT GENERATED ALWAYS
  AS ((StudentInfo ->> 'id')::BIGINT) STORED NOT NULL,
  StudentInfo JSONB NOT NULL,
  PRIMARY KEY(StudentId));
```

## View properties of a generated column

Spanner's `  INFORMATION_SCHEMA  ` contains information about the generated columns on your database. The following are some examples of the questions you can answer when you query the information schema.

*What generated columns are defined in my database?*

### GoogleSQL

``` text
SELECT c.TABLE_NAME, c.COLUMN_NAME, C.IS_STORED
FROM INFORMATION_SCHEMA.COLUMNS as c
WHERE c.GENERATION_EXPRESSION IS NOT NULL;
```

### PostgreSQL

``` text
SELECT c.TABLE_NAME, c.COLUMN_NAME, C.IS_STORED
FROM INFORMATION_SCHEMA.COLUMNS as c
WHERE c.GENERATION_EXPRESSION IS NOT NULL;
```

`  IS_STORED  ` is either `  YES  ` for stored generated columns, `  NO  ` for non-stored generated columns, or `  NULL  ` for non-generated columns.

*What is the current state of generated columns in table `  Users  ` ?*

If you have added a generated column to an existing table, you might want to pass `  SPANNER_STATE  ` in a query to find out the current state of the column. `  SPANNER_STATE  ` returns the following values:

  - `  COMMITTED  ` : The column is fully usable.
  - `  WRITE_ONLY  ` : The column is being backfilled. No read is allowed.

Use the following query to find the state of a column:

### GoogleSQL

``` text
SELECT c.TABLE_NAME, c.COLUMN_NAME, c.SPANNER_STATE
FROM INFORMATION_SCHEMA.COLUMNS AS c
WHERE c.TABLE_NAME="Users" AND c.GENERATION_EXPRESSION IS NOT NULL;
```

### PostgreSQL

``` text
SELECT c.TABLE_NAME, c.COLUMN_NAME, c.SPANNER_STATE
FROM INFORMATION_SCHEMA.COLUMNS AS c
WHERE c.TABLE_NAME='users' AND c.GENERATION_EXPRESSION IS NOT NULL;
```

**Note** : A generated column that's non-stored can only be accessed using the SQL query. However, if it's indexed, you can use the read API to access the value from the index.

## Performance

A `  STORED  ` generated column doesn't affect the performance of a read or query operation. However, non-stored generated columns used in a query can impact its performance due to the overhead of evaluating the generated column expression.

Performance of write operations (DML statements and mutations) is impacted when using either a `  STORED  ` generated column or a generated column that's indexed. The overhead is due to evaluating the generated column expression when the write operation inserts or modifies any of the columns referenced in the generated column expression. Since the overhead varies depending on the write workload for the application, schema design, and dataset characteristics, we recommend that you benchmark your applications before using a generated column.

## What's next

  - Learn more about Spanner's [Information schema for GoogleSQL-dialect databases](/spanner/docs/information-schema) and [Information schema for PostgreSQL-dialect databases](/spanner/docs/information-schema-pg) .

  - See more details about generated columns in the [CREATE TABLE](/spanner/docs/reference/standard-sql/data-definition-language#parameters_3) parameter details.
