A `  CHECK  ` constraint lets you specify that the values of one or more columns must satisfy a boolean expression. In this page, we describe how to add and manage this type of constraint in GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Add a check constraint to a new table

In the following `  CREATE TABLE  ` snippet, we create a table to store information about concerts. To require that the end time of a concert is later than its start time, we include a check constraint.

### GoogleSQL

``` text
CREATE TABLE Concerts (
  ConcertId INT64,
  StartTime Timestamp,
  EndTime Timestamp,
  CONSTRAINT start_before_end CHECK(StartTime < EndTime),
) PRIMARY KEY (ConcertId);
```

### PostgreSQL

``` text
CREATE TABLE Concerts (
  ConcertId BIGINT,
  StartTime TIMESTAMPTZ,
  EndTime TIMESTAMPTZ,
  CONSTRAINT start_before_end CHECK(StartTime < EndTime),
  PRIMARY KEY (ConcertId)
);
```

The constraint definition begins with the `  CONSTRAINT  ` keyword. We've explicitly named the constraint `  start_before_end  ` in this example to help you find it in error messages and whenever we need to refer to it. If no name is given, Spanner provides one, with the generated name beginning with the prefix `  CK_  ` . Constraint names are scoped to the schema, along with the names for tables and indexes, and must be unique within the schema. The check constraint definition consists of the keyword `  CHECK  ` followed by an expression in parentheses. The expression can only reference columns of this table. In this example, it references **StartTime** and **EndTime** , and the check constraint makes sure that the start time of a concert is always less than the end time.

The value of the check constraint expression is evaluated when a new row is inserted or when the `  StartTime  ` or `  EndTime  ` of an existing row are updated. If the expression evaluates to `  TRUE  ` or `  NULL  ` , the data change is allowed by the check constraint. If the expression evaluates to `  FALSE  ` , the data change is not allowed.

  - The following restrictions apply to a check constraint `  expression  ` term.
    
      - The expression can only reference columns in the same table.
    
      - The expression must reference at least one non-generated column, whether directly or through a generated column which references a non-generated column.
    
      - The expression can't reference columns that have set the `  allow_commit_timestamp  ` option.
    
      - The expression can't contain [subqueries](/spanner/docs/reference/standard-sql/subqueries) .
    
      - The expression can't contain non-deterministic functions, such as [`  CURRENT_DATE()  `](/spanner/docs/reference/standard-sql/date_functions#current_date) and [`  CURRENT_TIMESTAMP()  `](/spanner/docs/reference/standard-sql/timestamp_functions#current_timestamp) .

## Add a check constraint to an existing table

Using the following `  ALTER TABLE  ` statement, we add a constraint to make sure that all concert ids are greater than zero.

``` text
ALTER TABLE Concerts
ADD CONSTRAINT concert_id_gt_0 CHECK (ConcertId > 0);
```

Once again, we've given the constraint a name, **concert\_id\_gt\_0** . Adding a `  CHECK  ` constraint to an existing table starts the enforcement of the constraint immediately for new data and starts a long-running operation to validate that existing data conforms to the new constraint. Because this validation is performed as a long-running operation, ongoing transactions on the table are not impacted. For more information, see [Schema update performance](/spanner/docs/schema-updates#performance) . If there are any violations on existing data, the constraint is rolled back.

## Remove a check constraint

The following DDL statement drops a `  CHECK  ` constraint from the `  Concerts  ` table.

``` text
ALTER TABLE Concerts
DROP CONSTRAINT concert_id_gt_0;
```

## Modify a check constraint expression

Modifying the expression of a `  CHECK  ` constraint is not allowed. Instead, you need to drop the existing constraint and create a new constraint with the new expression.

## View properties of a check constraint

Spanner's [INFORMATION\_SCHEMA](/spanner/docs/information-schema) contains information about the check constraints on your database. The following are some examples of the questions you can answer by querying the information schema.

*What check constraints are defined in my database?*

``` text
SELECT tc.CONSTRAINT_NAME, tc.TABLE_NAME, tc.CONSTRAINT_TYPE
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS as tc
WHERE tc.CONSTRAINT_TYPE = 'CHECK';
```

*What is the current state of the check constraints in my database?*

If you have added a check constraint to an existing table, you might want to view its current state to determine, for example, whether all existing data has been validated against the constraint. If `  SPANNER_STATE  ` returns `  VALIDATING_DATA  ` in the following query, it means Spanner is still in the process of validating existing data against that constraint.

``` text
SELECT cc.CONSTRAINT_NAME, cc.SPANNER_STATE
FROM INFORMATION_SCHEMA.CHECK_CONSTRAINTS as cc;
```

## What's next

  - Learn more about Spanner's [INFORMATION SCHEMA](/spanner/docs/information-schema) .
