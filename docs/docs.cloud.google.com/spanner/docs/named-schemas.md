This page describes how to create, use, and drop named schemas, and how to apply fine-grained access control to objects in named schemas. For overview information about named schemas, see [Named schemas](/spanner/docs/schema-and-data-model#named-schemas) .

**Note:** To reference a database object in a non-default named schema within Spanner, you must use its fully qualified name. Using an unqualified name with a configured named schema search path isn't supported.

## Before you begin

To perform procedures on this page, you need the following:

  - The [Database Admin roles/spanner.databaseAdmin](/spanner/docs/iam#spanner.databaseAdmin) role on your user account.
  - Understand how [fine-grained access control works](/spanner/docs/fgac-about) .

## Create a named schema

The `  CREATE SCHEMA  ` command ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create_schema) and [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create_schema) ) is used to create a named schema.

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

6.  In the **Editor** tab, enter your DDL.
    
    ### GoogleSQL
    
    Run the `  CREATE SCHEMA  ` statement to create the named schema, for example:
    
    ``` text
    CREATE SCHEMA sch1;
    ```
    
    Add database objects in the named schema, for example:
    
    ``` text
    CREATE SEQUENCE sch1.sequence OPTIONS (
      sequence_kind = 'bit_reversed_positive'
    );
    
    CREATE TABLE Singers (
      SingerId INT64 NOT NULL,
      FirstName STRING(1024),
      LastName STRING(1024),
      SingerInfo BYTES(MAX),
    ) PRIMARY KEY(SingerId);
    
    CREATE INDEX indexOnSingers ON Singers(FirstName);
    
    CREATE TABLE Albums (
      SingerId INT64 NOT NULL,
      AlbumId INT64 NOT NULL,
      AlbumTitle STRING(MAX),
    ) PRIMARY KEY(SingerId, AlbumId),
      INTERLEAVE IN PARENT Singers ON DELETE CASCADE;
    
    CREATE TABLE Songs (
      SingerId INT64 NOT NULL,
      AlbumId INT64 NOT NULL,
      TrackId INT64 NOT NULL,
      SongName STRING(MAX),
    ) PRIMARY KEY(SingerId, AlbumId, TrackId),
      INTERLEAVE IN PARENT Albums ON DELETE CASCADE;
    
    CREATE TABLE sch1.Singers (
      SingerId INT64 NOT NULL,
      FirstName STRING(1024),
      LastName STRING(1024),
      SingerInfo BYTES(MAX),
    ) PRIMARY KEY(SingerId);
    
    CREATE INDEX sch1.indexOnSingers ON sch1.Singers(FirstName);
    
    CREATE TABLE sch1.Albums (
      SingerId INT64 NOT NULL,
      AlbumId INT64 NOT NULL,
      AlbumTitle STRING(MAX),
    ) PRIMARY KEY(SingerId, AlbumId),
      INTERLEAVE IN PARENT sch1.Singers ON DELETE CASCADE;
    
    CREATE TABLE sch1.Songs (
      SingerId INT64 NOT NULL,
      AlbumId INT64 NOT NULL,
      TrackId INT64 NOT NULL,
      SongName STRING(MAX),
    ) PRIMARY KEY(SingerId, AlbumId, TrackId),
      INTERLEAVE IN PARENT sch1.Albums ON DELETE CASCADE;
    
    CREATE VIEW sch1.SingerView SQL SECURITY INVOKER
      AS Select s.FirstName, s.LastName, s.SingerInfo
      FROM sch1.Singers AS s WHERE s.SingerId = 123456;
    
    CREATE VIEW SingerView SQL SECURITY INVOKER
      AS Select s.FirstName, s.LastName, s.SingerInfo
      FROM Singers AS s WHERE s.SingerId = 123456;
    ```
    
    Spanner only lets you create an index that uses the same schema as the table that uses the index. We need to make sure that the index and table schema names are the same.
    
    ### PostgreSQL
    
    Run the `  CREATE SCHEMA  ` statement to create the named schema, for example:
    
    ``` text
    CREATE SCHEMA sch1;
    ```
    
    Add database objects in the named schema, for example:
    
    ``` text
    CREATE SEQUENCE sch1.sequence BIT_REVERSED_POSITIVE
    CREATE TABLE sch1.singers(
      singer_id bigint primary key, album_id bigint default(nextval('sch1.sequence')))
    CREATE TABLE sch1.albums(k bigint default(nextval('sch1.sequence'))primary key, album_id bigint)
    
    CREATE VIEW sch1.singer_view SQL SECURITY INVOKER
      AS SELECT * FROM sch1.singers
    CREATE INDEX index_singers ON TABLE sch1.singers(album_id)
    ```
    
    Spanner only allows index creation in the same schema. In Spanner, PostgreSQL statements do this by default. You don't need to use fully qualified names to create indexes on named schemas.

7.  View the named schemas and related objects in the **Explorer** pane.

## Add fine-grained access control to a named schema

The following DDL statements add fine-grained access control to a named schema:

  - The `  USAGE  ` privilege grants privileges to the schema object. The `  USAGE  ` privilege is granted, by default, to the default schema. However, you can revoke the `  USAGE  ` privilege for the default schema. Use caution when revoking access because users and roles that are revoked lose all access to objects in the default schema.
  - The `  ALL  ` statement performs bulk grant privileges on ALL objects of a type in the schema.
  - The `  DEFAULT  ` keyword refers to the default schema in FGAC DDL statements.

To access an object in a named schema, you must have usage permission on the named schema and corresponding permissions on database objects that use that schema. For example, the following statements grant permissions to read from a table:

  - `  GRANT SELECT ON TABLE TABLE_NAME IN SCHEMA SCHEMA_NAME TO ROLE ROLE_NAME  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#grant_statement) and [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#grant_statement) ) grants the role permission to read from a specific table within a schema. Alternatively, you can grant the role permission to read from all tables within the schema using the `  ALL  ` keyword. For example, `  GRANT SELECT ON ALL TABLES IN SCHEMA SCHEMA_NAME TO ROLE ROLE_NAME  ` . The preceding `  ALL  ` statement applies to only tables present at the time of executing the statement.
  - `  GRANT USAGE ON SCHEMA SCHEMA_NAME TO ROLE ROLE_NAME  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#grant_statement) and [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#grant_statement) ) gives the role permission to access objects contained in the schema if the contained object's permissions are also satisfied. For example, for a table in a schema, you would need 'USAGE' on the schema, and 'SELECT' on the table to read from the table.

<!-- end list -->

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

6.  In the **Editor** tab, enter your DDL.
    
    ### GoogleSQL
    
    Create your custom role for the named schema. In the following example, we use `  role1  ` and `  role2  ` .
    
    ``` text
    CREATE ROLE role1
    CREATE ROLE role2
    ```
    
    Grant the role to the tables that use the named schema using `  GRANT ALL  ` . In the following example, we use `  sch1  ` for the named schema and `  role1  ` for the role.
    
    ``` text
    GRANT SELECT ON ALL TABLES IN SCHEMA sch1 TO ROLE role1
    ```
    
    Grant usage on the schema for the roles you created. In the following example, we grant usage on `  sch1  ` to `  role1  ` and `  role2  ` .
    
    ``` text
    GRANT USAGE ON SCHEMA sch1 TO ROLE role1, role2
    ```
    
    ### PostgreSQL
    
    Create your custom role for the named schema. In the following example, we use `  role1  ` and `  role2  ` .
    
    ``` text
    CREATE ROLE role1
    CREATE ROLE role2
    ```
    
    Grant the role to the tables that use the named schema using `  GRANT ALL  ` . In the following example, we use `  sch1  ` for the named schema and `  role1  ` for the role.
    
    ``` text
    GRANT SELECT ON ALL TABLES IN SCHEMA sch1 TO role1
    ```
    
    Grant usage on the schema for the roles you created. In the following example, we grant usage on `  sch1  ` to `  role1  ` and `  role2  ` .
    
    ``` text
    GRANT USAGE ON SCHEMA sch1 TO role1, role2
    ```

## Add and revoke fine-grained access control to a default schema

When you have named schemas, the default schema is called `  default  ` . You need to use the `  default  ` schema name when adding or revoking fine-grained access control.

### Add fine-grained access control to a default schema

By default, all users and roles have the `  USAGE  ` permission on the default schema.

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

6.  In the **Editor** tab, enter your DDL.
    
    ### GoogleSQL
    
    In the following example, we grant access to all tables to `  role1  ` .
    
    ``` text
    GRANT SELECT ON ALL TABLES IN SCHEMA default TO ROLE role1
    ```
    
    ### PostgreSQL
    
    In the following example, we grant access to all tables to `  role1  ` .
    
    ``` text
    GRANT SELECT ON ALL TABLES IN SCHEMA default TO role1
    ```

### Revoke fine-grained access control to a default schema

You can revoke the default fine-grained access control permissions on the default schema using the `  REVOKE USAGE  ` command.

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

6.  In the **Editor** tab, enter your DDL.
    
    ### GoogleSQL
    
    ``` text
    REVOKE USAGE ON SCHEMA DEFAULT FROM ROLE public
    ```
    
    ### PostgreSQL
    
    ``` text
    REVOKE USAGE ON SCHEMA public FROM public
    ```

7.  After the previous command is run, we must explicitly grant permissions to the roles that need to access the default schema. In the following example, we grant permissions to `  role1  ` .
    
    ### GoogleSQL
    
    In the following example, we grant access to the default schema to `  role1  ` .
    
    ``` text
    GRANT USAGE ON SCHEMA default to ROLE role1
    ```
    
    ### PostgreSQL
    
    In the following example, we grant access to the default schema to `  role1  ` .
    
    ``` text
    GRANT USAGE ON SCHEMA public To role1
    ```

## View named schemas

1.  Select an instance from the list.
2.  Select a database.
3.  In the navigation menu, click **Spanner Studio** .
4.  In the **Explorer** pane, click the arrow\_right toggle node to expand the *Schemas* drop-down list.

Alternatively, you can use SQL to view all schemas in the `  information_schema.schemata  ` table.

The following example shows how view names schemas and their owners:

``` text
SELECT schema_name, schema_owner
  FROM information_schema.schemata
  ORDER BY schema_owner
```

This statement provides a list of schemas and owners, similar to the following:

``` text
public spanner_admin
products  spanner_admin
analytics  spanner_admin
logs  spanner_admin
pg_catalog spanner_system
information_schema spanner_system
spanner_sys   spanner_system
```

## Drop a named schema

**Note:** Spanner doesn't support a cascading drop for named schemas. You must drop database objects that use the named schema before dropping the schema.

The `  DROP SCHEMA  ` command ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#drop_schema) and [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#drop_schema) ) is used to drop a named schema.

1.  In the Google Cloud console, open the **Spanner** page.

2.  Select an instance from the list.

3.  Select a database.

4.  In the navigation menu, click **Spanner Studio** .

5.  Open a new tab by clicking add **New SQL editor tab** or add **New tab** .

6.  In the **Editor** tab, enter your DDL.
    
    ### GoogleSQL
    
    In the following example, we drop `  sch1  ` .
    
    ``` text
    DROP SCHEMA IF EXISTS sch1;
    ```
    
    ### PostgreSQL
    
    In the following example, we drop `  sch1  ` .
    
    ``` text
    DROP SCHEMA IF EXISTS sch1;
    ```

## What's next

  - Learn about [Schema best practices](/spanner/docs/schema-design) .
  - Learn about [Views](/spanner/docs/views) .
  - Learn about [Secondary indexes](/spanner/docs/secondary-indexes) .
  - Learn about [Foreign keys](/spanner/docs/foreign-keys/overview) .
