This page describes how to create and manage Spanner views for GoogleSQL-dialect databases and PostgreSQL-dialect databases. For more information about Spanner views, see [Views overview](/spanner/docs/views) .

## Permissions

To create, grant, and revoke access to a view, you must have the [`  spanner.database.updateDdl  `](/spanner/docs/iam#databases) permission.

## Create a view

To create a view, use the DDL statement [`  CREATE VIEW  `](/spanner/docs/reference/standard-sql/data-definition-language#create-view) to name the view and provide the query that defines it. This statement has two forms:

  - `  CREATE VIEW  ` defines a new view in the current database. If a view named `  view_name  ` already exists, the `  CREATE VIEW  ` statement fails.

  - `  CREATE OR REPLACE VIEW  ` defines a new view in the current database. If a view named `  view_name  ` already exists, its definition is replaced.

The syntax for the CREATE VIEW statement is:

``` text
{CREATE | CREATE OR REPLACE } VIEW  view_name
SQL SECURITY { INVOKER | DEFINER }
AS query
```

Because a view is a virtual table, the `  query  ` that you specify must provide names for all the columns in that virtual table.

Additionally, Spanner checks the `  query  ` you specify using *strict name resolution* , meaning that all schema object names used in the query must be qualified such that they unambiguously identify a single schema object. For example, in the examples that follow the `  SingerId  ` column in the `  Singers  ` table must be qualified as `  Singers.SingerId  ` .

You must specify the `  SQL SECURITY  ` as either `  INVOKER  ` or `  DEFINER  ` in the `  CREATE VIEW  ` or `  CREATE OR REPLACE VIEW  ` statement. For more information about the difference between the two security types, see [Views overview](/spanner/docs/views) .

For example, assume the `  Singers  ` table is defined as shown in the following:

### GoogleSQL

``` text
CREATE TABLE Singers (
  SingerId   INT64 NOT NULL,
  FirstName  STRING(1024),
  LastName   STRING(1024),
  SingerInfo BYTES(MAX)
) PRIMARY KEY (SingerId);
```

### PostgreSQL

``` text
CREATE TABLE Singers (
  SingerId   BIGINT PRIMARY KEY,
  FirstName  VARCHAR(1024),
  LastName   VARCHAR(1024),
  SingerInfo BYTEA
);
```

You can define the `  SingerNames  ` view with invoker's rights as shown in the following:

``` text
CREATE VIEW SingerNames
SQL SECURITY INVOKER
AS SELECT
   Singers.SingerId AS SingerId,
   Singers.FirstName || ' ' || Singers.LastName AS Name
FROM Singers;
```

The virtual table created when the `  SingerNames  ` view is used in a query has two columns, `  SingerId  ` and `  Name  ` .

While this definition of the `  SingerNames  ` view is valid, it does not abide by the best practice of casting data types to ensure stability across schema changes, as described in the next section.

### Best practices when creating views

To minimize the need to update a view's definition, explicitly cast the data type of all table columns in the query that defines the view. When you do so, the view's definition can remain valid across schema changes to a column's type.

For example, the following definition of the `  SingerNames  ` view might become invalid as the result of changing a column's data type in the `  Singers  ` table.

``` text
CREATE VIEW SingerNames
SQL SECURITY INVOKER
AS SELECT
   Singers.SingerId AS SingerId,
   Singers.FirstName || ' ' || Singers.LastName AS Name
FROM Singers;
```

You can avoid the view becoming invalid by explicitly casting the columns to the needed data types, as shown in the following example:

### GoogleSQL

``` text
CREATE OR REPLACE VIEW SingerNames
SQL SECURITY INVOKER
AS SELECT
 CAST(Singers.SingerId AS INT64) AS SingerId,
 CAST(Singers.FirstName AS STRING) || " " || CAST(Singers.LastName AS STRING) AS Name
FROM Singers;
```

### PostgreSQL

``` text
CREATE OR REPLACE VIEW SingerNames
SQL SECURITY INVOKER
AS SELECT
 CAST(Singers.SingerId AS bigint) AS SingerId,
 CAST(Singers.FirstName AS varchar) || ' ' || CAST(Singers.LastName AS varchar) AS Name
FROM Singers;
```

## Grant and revoke access to a view

**Note:** If you create a definer's rights view, use [fine-grained access control](/spanner/docs/fgac-about) alongside the view, otherwise the definer's rights view doesn't add any additional access control.

As a fine-grained access control user, you must have the `  SELECT  ` privilege on a view. To grant `  SELECT  ` privilege on a view to a database role:

### GoogleSQL

``` text
GRANT SELECT ON VIEW SingerNames TO ROLE Analyst;
```

### PostgreSQL

``` text
GRANT SELECT ON TABLE SingerNames TO Analyst;
```

To revoke `  SELECT  ` privilege on a view from a database role:

### GoogleSQL

``` text
REVOKE SELECT ON VIEW SingerNames FROM ROLE Analyst;
```

### PostgreSQL

``` text
REVOKE SELECT ON TABLE SingerNames FROM Analyst;
```

## Query a view

The way to query an invoker's rights or a definer's rights view is the same. However, depending on the security type of the view, Spanner may or may not need to check the schema objects referenced in the view against the database role of the principal who invoked the query.

### Query an invoker's rights view

If a view has invoker's rights, the user must have privileges on all underlying schema objects of the view in order to query it.

For example, if a database role has access to all objects referenced by the `  SingerNames  ` view, they can query the `  SingerNames  ` view:

``` text
SELECT COUNT(SingerID) as SingerCount
FROM SingerNames;
```

### Query a definer's rights view

If a view has definer's rights, a user can query the view without needing privileges on the underlying objects as long as you grant the required role the `  SELECT  ` privilege on the view.

In the following example, a user with the Analyst database role wants to query the `  SingerNames  ` view. However, the user is denied access because `  SingerNames  ` is an invoker's rights view and the Analyst role does not have access to all the underlying objects. In this case, if you decide to provide the Analyst with access to the view, but don't want to provide them access to the `  Singers  ` table, you can [replace the security type of the view](#replace-view) to definer's rights. After you replace the security type of the view, grant the Analyst role access to the view. The user can now query the `  SingerNames  ` view even though they don't have access to the `  Singers  ` table.

``` text
SELECT COUNT(SingerID) as SingerCount
FROM SingerNames;
```

## Replace a view

You can replace a view by using the `  CREATE OR REPLACE VIEW  ` statement to change the view definition or the security type of the view.

Replacing a view is similar to dropping and recreating the view. Any access grants given to the initial view has to be granted again after replacing the view.

To replace an invoker's rights view with a definer's rights view:

``` text
CREATE OR REPLACE VIEW SingerNames
SQL SECURITY DEFINER
AS SELECT
   Singers.SingerId AS SingerId,
   Singers.FirstName || ' ' || Singers.LastName AS Name
FROM Singers;
```

## Delete a view

After a view is dropped, database roles with privileges on it no longer have access. To delete a view, use the `  DROP VIEW  ` statement.

``` text
DROP VIEW SingerNames;
```

## Get information about a view

You can get information about views in a database by querying tables in its `  INFORMATION_SCHEMA  ` schema.

  - The `  INFORMATION_SCHEMA.TABLES  ` table provides the names of all defined views.

  - The `  INFORMATION_SCHEMA.VIEWS  ` table provides the names, view definition, security type, and query text of all defined views. FGAC users who have `  SELECT  ` privilege on the view can get information about the view from the `  INFORMATION_SCHEMA.VIEWS  ` table. Other FGAC users need the `  spanner_info_reader  ` role if they don't have `  SELECT  ` privilege for the view.

To check the view definition and security type of a view called `  ProductSoldLastWeek  ` :

``` text
  SELECT *
  FROM INFORMATION_SCHEMA.VIEWS
  WHERE TABLE_NAME = 'ProductSoldLastWeek';
```
