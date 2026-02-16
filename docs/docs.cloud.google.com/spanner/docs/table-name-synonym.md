This page describes how to rename tables and how to add, use, and drop table synonyms in GoogleSQL-dialect databases and PostgreSQL-dialect databases.

## Options for table renaming and synonyms

You can use the `  ALTER TABLE  ` statement to do the following:

  - [Rename a table and add the old name to a synonym](#rename-with-synonyms) .
  - [Swap table names](#how-renaming-works) .
  - [Rename a single table](#how-renaming-works) .
  - [Create a new table with a single synonym](#create-synonym) .
  - [Add a single synonym to a table without renaming it](#synonyms) .

## How table renaming with synonyms works

A common scenario is to rename a table and add a synonym that contains the old table name. After renaming the table, you can update applications to use the new name on your schedule. During this period, it's possible that some applications use the old name and others use the new name.

After you update all of your applications to use the new name, we recommend that you remove the synonym. While having a synonym doesn't impact performance, you can't use the old name somewhere else until the synonym is dropped.

Synonyms are stored in the schema as a `  synonym  ` object. You can only have one synonym on a table.

For more information, see [Rename a table and add a synonym](#rename-add-synonym) .

## How table name swapping works

When you need to swap names between two tables, you can chain together `  RENAME TO  ` statements to rename two tables in the same statement. This lets you link applications to a different table without interruptions.

For more information, see [Swap table names](#rename-tables) .

## How table renaming works

When you rename a table, Spanner changes the table name in the table's schema. Renaming a table interleaves any child tables with the new table name. Table renaming also changes references to the table for the following:

  - Indexes
  - Foreign keys
  - Change streams
  - Fine-grained access control (FGAC)

Spanner doesn't automatically update views to use the new table name.

**Warning:** Use care when renaming a table because it can cause an outage if there are processes that reference the table. One scenario where you might rename a table is if the table name was misspelled when you created it.

For more information, see [Rename a table](#rename-table) .

### Table renaming limitations

Table renaming has the following limitations:

  - You can't rename a table to the name of a column in that table if the table is interleaved in another table.
  - You can't rename indexes. To change the name of an index, drop it and recreate the index with a new name.
  - If the table has a view, you might want to drop the view and recreate it after renaming the table.

## How synonyms work

You can create a new table with a synonym or alter a table to add a synonym to it without renaming the table. A scenario for when you might want to do this is if you want to use a database for both a production and test environment.

For more information, see [Add a synonym to a table](#add-synonym) .

## Permissions

To rename a table or add a synonym to a table, you need the `  spanner.databases.updateDdl  ` permission. To check or edit your permissions, see [Grant permissions to principles](/spanner/docs/grant-permissions#grant_permissions_to_principals) .

## Rename a table and add a synonym

### GoogleSQL

Use [`  ALTER TABLE RENAME TO ADD SYNONYM  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) to rename a table and add a synonym.

``` text
  ALTER TABLE table_name RENAME TO new_table_name, ADD SYNONYM table_name;
```

### PostgreSQL

Use [`  ALTER TABLE RENAME WITH ADD SYNONYM  `](/spanner/docs/reference/postgresql/data-definition-language#alter_table) to rename a table and add a synonym.

``` text
  ALTER TABLE table_name RENAME WITH SYNONYM TO new_table_name;
```

The following example shows how to rename a table and add a synonym. For example, if you create a table with the following DDL:

### GoogleSQL

``` text
  CREATE TABLE Singers (
      SingerId INT64 NOT NULL,
      SingerName STRING(1024)
  ), PRIMARY KEY (SingerId);
```

### PostgreSQL

``` text
  CREATE TABLE singers (
      singer_id BIGINT,
      singer_name VARCHAR(1024),
      PRIMARY KEY (singer_id));
```

You can make the following DDL request to rename the table and move the existing name to the `  synonym  ` object.

### GoogleSQL

``` text
  ALTER TABLE Singers RENAME TO SingersNew, ADD SYNONYM Singers;
```

### PostgreSQL

``` text
  ALTER TABLE singers RENAME WITH SYNONYM TO singers_new;
```

## Swap table names

**Warning:** Renaming a table that has a reference from a process can cause an outage unless the table name is assigned to another table in the same statement.

The following DDL statement changes the names of multiple tables atomically. This is useful when swapping the names between one or more pairs of tables.

### GoogleSQL

Use [`  RENAME TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#rename_table) .

``` text
  RENAME TABLE old_name1 TO new_name1 [,old_name2 TO new_name2 ...];
```

### PostgreSQL

Use [`  ALTER TABLE RENAME TO  `](/spanner/docs/reference/postgresql/data-definition-language#alter_table) .

``` text
  ALTER TABLE [ IF EXISTS ] [ ONLY ] table_name1
        RENAME TO new_table_name1
        [, ALTER TABLE [ IF EXISTS ] [ ONLY ] table_name2
              RENAME TO new_table_name2 ...];
```

The following example shows how to swap the names of two tables. This requires that the first table is renamed to a temporary name, the second table is renamed to the first table's name, then the first table is renamed to the second table's name.

If you have created two tables as shown in the following:

### GoogleSQL

``` text
  CREATE TABLE Singers (
        SingerId INT64 NOT NULL,
        SingerName STRING(1024)
        ), PRIMARY KEY (SingerId);

  CREATE TABLE SingersNew (
        SingerId INT64 NOT NULL,
        FirstName STRING(1024),
        MiddleName STRING(1024),
        LastName STRING(1024)
        ), PRIMARY KEY (SingerId);
```

### PostgreSQL

``` text
  CREATE TABLE singers (
        singer_id BIGINT,
        singer_name VARCHAR(1024),
        PRIMARY KEY (singer_id)
        );

  CREATE TABLE singers_new (
        singer_id BIGINT,
        first_name VARCHAR(1024),
        middle_name VARCHAR(1024),
        last_name VARCHAR(1024)
        PRIMARY KEY (singer_id)
        );
```

You can use the following DDL request to swap the table names:

### GoogleSQL

``` text
  RENAME TABLE Singers TO Temp, SingersNew TO Singers, Temp TO SingersNew;
```

### PostgreSQL

``` text
  ALTER TABLE singers RENAME TO temp,
        ALTER TABLE singers_new RENAME TO singers,
        ALTER TABLE temp RENAME TO singers_new;
```

After the DDL statement is applied, the table names are swapped, as shown in the following:

### GoogleSQL

``` text
  CREATE TABLE Singers (
        SingerId INT64 NOT NULL,
        FirstName STRING(1024),
        MiddleName STRING(1024),
        LastName STRING(1024)
        ), PRIMARY KEY (SingerId);

  CREATE TABLE SingersNew (
        SingerId INT64 NOT NULL,
        SingerName STRING(1024)
        ), PRIMARY KEY (SingerId);
```

### PostgreSQL

``` text
  CREATE TABLE singers (
        singer_id BIGINT,
        first_name VARCHAR(1024),
        middle_name VARCHAR(1024),
        last_name VARCHAR(1024)
        PRIMARY KEY (singer_id)
        );

  CREATE TABLE singers_new (
        singer_id BIGINT,
        singer_name VARCHAR(1024),
        PRIMARY KEY (singer_id)
        );
```

## Rename a table

**Warning:** Renaming a table that has a reference from a process can cause an outage.

To rename a table, use the following syntax:

### GoogleSQL

Use either the [`  ALTER NAME  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) or [`  RENAME TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#rename_table) statement.

``` text
  ALTER TABLE table_name RENAME TO new_table_name;
  RENAME TABLE table_name TO new_table_name;
```

### PostgreSQL

Use the [`  ALTER TABLE RENAME TO  `](/spanner/docs/reference/postgresql/data-definition-language#alter_table) statement.

``` text
  ALTER TABLE [ IF EXISTS ] [ ONLY ] table_name
        RENAME TO new_table_name;
```

The following example shows a DDL request that renames the table:

### GoogleSQL

``` text
  RENAME TABLE Singers TO SingersNew;
```

### PostgreSQL

``` text
  ALTER TABLE singers RENAME TO singers_new;
```

## Add a synonym to a table

To add a synonym to a table:

### GoogleSQL

``` text
  ALTER TABLE table_name ADD SYNONYM synonym;
```

### PostgreSQL

``` text
  ALTER TABLE [ IF EXISTS ] [ ONLY ] table_name ADD SYNONYM synonym;
```

The following example shows a DDL request that adds a synonym to the table:

### GoogleSQL

``` text
  ALTER TABLE Singers ADD SYNONYM SingersTest;
```

### PostgreSQL

``` text
  ALTER TABLE singers ADD SYNONYM singers_test;
```

## Create a table with a synonym

To create a table with a synonym:

### GoogleSQL

Use [`  CREATE TABLE SYNONYM synonym_name  `](/spanner/docs/reference/postgresql/data-definition-language) .

``` text
  CREATE TABLE table_name (
      ...
      SYNONYM (synonym)
  ) PRIMARY KEY (primary_key);
```

### PostgreSQL

Use [`  CREATE TABLE SYNONYM synonym_name  `](/spanner/docs/reference/postgresql/data-definition-language) .

``` text
  CREATE TABLE table_name (
      ...
      SYNONYM (synonym),
      PRIMARY KEY (primary_key));
```

The following example creates a table and adds a synonym.

### GoogleSQL

``` text
  # The table's name is Singers and the synonym is Artists.
  CREATE TABLE Singers (
      SingerId INT64 NOT NULL,
      SingerName STRING(1024),
      SYNONYM (Artists)
  ) PRIMARY KEY (SingerId);
```

### PostgreSQL

``` text
  # The table's name is singers and the synonym is artists.
  CREATE TABLE singers (
      singer_id BIGINT,
      singer_name VARCHAR(1024),
      SYNONYM (artists),
      PRIMARY KEY (singer_id));
```

## Remove a synonym from a table

### GoogleSQL

Use [ALTER TABLE DROP SYNONYM](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) to remove the synonym from the table.

``` text
  ALTER TABLE table_name DROP SYNONYM synonym;
```

### PostgreSQL

Use [ALTER TABLE DROP SYNONYM](/spanner/docs/reference/postgresql/data-definition-language#alter_table) to remove the synonym from the table.

``` text
  ALTER TABLE [ IF EXISTS ] [ ONLY ] table_name DROP SYNONYM synonym;
```

The following example shows a DDL request that drops the synonym from the table:

### GoogleSQL

``` text
  ALTER TABLE Singers DROP SYNONYM SingersTest;
```

### PostgreSQL

``` text
  ALTER TABLE singers DROP SYNONYM singers_test;
```
