**PostgreSQL interface note:** For PostgreSQL-dialect databases, you can use the [psql command-line tool](/spanner/docs/psql-commands) . The examples in this document are intended for GoogleSQL-dialect databases.

This page summarizes the Spanner CLI supported commands.

**Command syntax**

**Description**

`  SHOW DATABASES;  `

List databases.

`  USE database_name  ` \[ROLE `  role_name  ` \];

Switch database. The role you set is used for [fine-grained access control](/spanner/docs/fgac-about) .

`  CREATE DATABASE database_name  ` ;

Create database.

`  DROP DATABASE database_name  ` ;

Delete database.

`  SHOW TABLES [ schema  ` \];

List tables. If you don't provide a schema, Spanner uses the default schema.

`  SHOW CREATE TABLE table_name  ` ;

Show table schema.

`  SHOW COLUMNS FROM table_name  ` ;

Show columns.

`  SHOW INDEX FROM table_name  ` ;

Show indexes.

`  CREATE TABLE ...;  `

Create table.

`  ALTER TABLE ...;  `

Change table schema.

`  DROP TABLE ...;  `

Delete table.

`  TRUNCATE TABLE table_name  ` ;

Truncate table. Only delete rows. This command is non-atomic because it's executed as a [partitioned DML statement](/spanner/docs/dml-partitioned) .

`  CREATE INDEX ...;  `

Create index.

`  DROP INDEX ...;  `

Delete index.

`  CREATE ROLE ...;  `

Create role. For more information, see Spanner [IAM overview](/spanner/docs/iam) .

`  DROP ROLE ...;  `

Delete role.

`  GRANT ...;  `

Grant permission to a role.

`  REVOKE ...;  `

Revoke permission from a role.

`  SELECT ...;  `

Run a query.

`  { INSERT|UPDATE|DELETE } ...;  `

Execute a DML statement.

`  PARTITIONED { UPDATE|DELETE } ...;  `

Execute a partitioned DML statement. This command is non-atomic.

`  EXPLAIN SELECT ...;  `

Show a query execution plan. For more information, see [Query execution plans](/spanner/docs/query-execution-plans) .

`  EXPLAIN {INSERT|UPDATE|DELETE} ...;  `

Show the DML execution plan.

`  EXPLAIN ANALYZE SELECT ...;  `

Show query execution plan with optimizer statistics. For more information, see [Optimizer statistics packages](/spanner/docs/query-optimizer/manage-query-optimizer#list-statistics-packages) .

`  EXPLAIN ANALYZE {INSERT|UPDATE|DELETE} ...;  `

Show the DML execution plan with optimizer statistics. For more information, see [Optimizer statistics packages](/spanner/docs/query-optimizer/manage-query-optimizer#list-statistics-packages) .

`  DESCRIBE SELECT ...;  `

Show query result shape.

`  DESCRIBE {INSERT|UPDATE|DELETE} ... THEN RETURN ...;  `

Show DML result shape.

`  ANALYZE;  `

Start a new query optimizer statistics package construction.

`  START BATCH DDL;  `

Start a DDL batch.

`  RUN BATCH;  `

Run batch commands.

`  ABORT BATCH;  `

Abort batch commands.

`  BEGIN [RW] [ISOLATION LEVEL {SERIALIZABLE|REPEATABLE READ}] [PRIORITY {HIGH|MEDIUM|LOW}] [TAG tag_name  ` \];

Start a read-write transaction. For more information, see [Transaction commands](#transaction-commands) .

`  COMMIT;  `

Commit a read-write transaction.

`  ROLLBACK;  `

Rollback (undo) a read-write transaction.

`  BEGIN RO [{ seconds  ` | `  RFC 3339-formatted_time  ` }\] \[PRIORITY {HIGH|MEDIUM|LOW}\] \[TAG `  tag_name  ` \];

Start a read-only transaction. seconds and RFC 3339-formatted\_time are used for stale reads. For more information, see [Transaction commands](#transaction-commands) .

`  CLOSE;  `

End a read-only transaction.

`  EXIT;  `

Exit Spanner CLI.

### `     BATCH    ` commands

The Spanner CLI lets you perform DDL operations in batch mode, which groups multiple DDL statements into a single operation and speeds up schema changes. The Spanner CLI supports the following `  BATCH  ` commands:

**`  START BATCH DDL;  `**

This command initiates a DDL batch. All subsequent DDL statements (for example, `  CREATE TABLE  ` , `  ALTER TABLE  ` , `  DROP INDEX  ` ) that you execute within this session remain in a pending state and aren't applied to the database immediately.

**`  RUN BATCH;  `**

After executing `  START BATCH DDL  ` and subsequent DDL statements, use the `  RUN BATCH  ` command to send all the pending DDL operations as a single request to Spanner. This command reduces the overhead associated with individual DDL statements, leading to faster schema modifications.

**`  ABORT BATCH;  `**

If you decide not to apply the pending DDL changes, use the `  ABORT BATCH  ` command. This command discards all DDL statements collected since you issued the `  START BATCH DDL  ` command, effectively rolling back the batch and leaving the database schema unchanged.

### Transaction commands

The Spanner CLI supports the following transaction SQL commands:

**`  BEGIN [TRANSACTION] [RO] [ seconds | RFC 3339-formatted_time ] [ISOLATION LEVEL {SERIALIZABLE|REPEATABLE READ}] [PRIORITY {HIGH|MEDIUM|LOW}] [TAG tag_name ];  `**

Start a transaction. You can configure these options:

  - Transaction type: Start a read-write (no parameter needed) or read-only ( `  RO  ` ) transaction.

  - Stale reads time: Set the time, in seconds or RFC 3339-formatted, to read data from a specific timestamp.

  - Isolation level: Set the isolation level for read-write transactions. By default, serializable isolation is used. For more information, see [Isolation levels overview](/spanner/docs/isolation-levels) .

  - Priority: Set the request priority for the transaction. Medium priority is set by default.

  - Tag: Set transaction tags using the `  BEGIN  ` command.
    
      - In a read-write transaction, add a tag with `  BEGIN TAG tag  ` . The Spanner CLI adds the tag as a transaction tag. The tag is also used as a request tag within the transaction.
      - In a read-only transaction, add a tag with `  BEGIN RO TAG tag  ` . Because read-only transactions don't support transaction tags, Spanner adds the tag as a request tag.

**`  COMMIT;  `**

Finalize and make permanent all changes in a read-write transaction.

**`  CLOSE;  `**

Close a read-only transaction.

**`  ROLLBACK;  `**

Rollback (undo) a read-write transaction.

## What's next

  - Learn more about the [Spanner CLI](/spanner/docs/spanner-cli) .
