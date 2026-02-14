Spanner PGAdapter supports session management statements, which let you modify the state and behavior of your connection, execute transactions, and efficiently execute batches of statements. All statements that are described in this document can be used with any client or driver that connects to PGAdapter.

For more information, see a full list of supported [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql_drivers_and_orms) . The following commands apply to PostgreSQL-dialect databases.

For more information on using PGAdapter, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

## Connection statements

The following statements make changes to or display properties of the current connection.

#### SPANNER.READONLY

A boolean indicating whether or not the connection is in read-only mode. The default is `  false  ` .

``` text
SHOW [VARIABLE] SPANNER.READONLY
SET SPANNER.READONLY {TO|=} { true | false }
```

You can change the value of this property only while there is no active transaction.

##### ▶ Example: Read-only transaction (Click to expand)

The following example shows how to use this property to execute read-only transactions in Spanner.

``` text
SET SPANNER.READONLY = TRUE;
-- This transaction is a read-only transaction.
BEGIN TRANSACTION;

-- The following two queries both use the read-only transaction.
SELECT first_name, last_name
FROM singers
ORDER BY last_name;

SELECT first_name, last_name
FROM albums
ORDER BY title;

-- This shows the read timestamp that was used for the two queries.
SHOW SPANNER.READ_TIMESTAMP;

-- This marks the end of the read-only transaction. The next statement will
-- start a new read-only transaction.
COMMIT;
```

#### AUTOCOMMIT

A boolean indicating whether or not the connection is in autocommit mode. The default is `  true  ` .

**NOTE** : You don't normally need to modify the value of this variable when using a PostgreSQL driver with PGAdapter. These drivers manage transactions automatically for you by executing `  BEGIN  ` and `  COMMIT  ` when necessary. You can turn off `  autocommit  ` when using command-line tools like psql to prevent accidental data modifications from being committed automatically.

``` text
SHOW [VARIABLE] AUTOCOMMIT
SET AUTOCOMMIT {TO|=} { true | false }
```

You can change the value of this property only when there is no active transaction.

When `  AUTOCOMMIT  ` is set to false, a new transaction is initiated automatically after you execute `  COMMIT  ` or `  ROLLBACK  ` . The first statement that you execute starts the transaction.

##### ▶ Example: Autocommit (Click to expand)

The following example shows how to use the `  autocommit  ` property.

``` text
-- The default value for AUTOCOMMIT is true.
SHOW AUTOCOMMIT;

-- This insert statement is automatically committed after it is executed, as
-- the connection is in autocommit mode.
INSERT INTO T (id, col_a, col_b) VALUES (1, 100, 1);

-- Turning off autocommit means that a new transaction is automatically started
-- when the next statement is executed.
SET AUTOCOMMIT = FALSE;
-- The following statement starts a new transaction.
INSERT INTO T (id, col_a, col_b) VALUES (2, 200, 2);

-- This statement uses the same transaction as the previous statement.
INSERT INTO T (id, col_a, col_b) VALUES (3, 300, 3);

-- Commit the current transaction with the two INSERT statements.
COMMIT;

-- Transactions can also be executed in autocommit mode by executing the BEGIN
-- statement.
SET AUTOCOMMIT = TRUE;

-- Execute a transaction while in autocommit mode.
BEGIN;
INSERT INTO T (id, col_a, col_b) VALUES (4, 400, 4);
INSERT INTO T (id, col_a, col_b) VALUES (5, 500, 5);
COMMIT;
```

#### SPANNER.RETRY\_ABORTS\_INTERNALLY

A boolean indicating whether the connection automatically retries aborted transactions. The default is `  true  ` .

``` text
SHOW [VARIABLE] SPANNER.RETRY_ABORTS_INTERNALLY
SET SPANNER.RETRY_ABORTS_INTERNALLY {TO|=} { true | false }
```

You can execute this command only after a transaction is started (see [`  BEGIN [TRANSACTION | WORK]  `](#start_a_transaction) ) and before any statements are executed within the transaction.

When you enable `  SPANNER.RETRY_ABORTS_INTERNALLY  ` , the connection keeps a checksum of all data that the connection returns to the client application. This is used to retry the transaction if it is aborted by Spanner.

This setting is enabled by default. We recommend turning off this setting if your application already retries aborted transactions.

#### SPANNER.AUTOCOMMIT\_DML\_MODE

A `  STRING  ` property indicating the autocommit mode for [Data Manipulation Language (DML)](/spanner/docs/reference/postgresql/dml-syntax) statements.

``` text
SHOW [VARIABLE] SPANNER.AUTOCOMMIT_DML_MODE
SET SPANNER.AUTOCOMMIT_DML_MODE {TO|=} { 'TRANSACTIONAL' | 'PARTITIONED_NON_ATOMIC' }
```

The possible values are:

  - In `  TRANSACTIONAL  ` mode, the driver executes DML statements as separate atomic transactions. The driver creates a new transaction, executes the DML statement, and either commits the transaction upon successful execution or rolls back the transaction in the case of an error.
  - In `  PARTITIONED_NON_ATOMIC  ` mode, the driver executes DML statements as [partitioned update statements](/spanner/docs/dml-partitioned) . A partitioned update statement can run as a series of many transactions, each covering a subset of the rows impacted. The partitioned statement provides weakened semantics in exchange for better scalability and performance.

The default is `  TRANSACTIONAL  ` .

##### ▶ Example: Partitioned DML (Click to expand)

The following example shows how to execute [Partitioned DML](/spanner/docs/dml-partitioned) using PGAdapter.

``` text
-- Change autocommit DML mode to use Partitioned DML.
SET SPANNER.AUTOCOMMIT_DML_MODE = 'PARTITIONED_NON_ATOMIC';

-- Delete all singers that have been marked as inactive.
-- This statement is executed using Partitioned DML.
DELETE
FROM singers
WHERE active=false;

-- Change DML mode back to standard `TRANSACTIONAL`.
SET SPANNER.AUTOCOMMIT_DML_MODE = 'TRANSACTIONAL';
```

#### STATEMENT\_TIMEOUT

A property of type `  STRING  ` indicating the current timeout value for statements.

``` text
SHOW [VARIABLE] STATEMENT_TIMEOUT
SET STATEMENT_TIMEOUT {TO|=} { '<int8>{ s | ms | us | ns }' | <int8> | DEFAULT }
```

The `  int8  ` value is a whole number followed by a suffix indicating the time unit. A value of `  DEFAULT  ` indicates that there is no timeout value set. If a statement timeout value has been set, statements that take longer than the specified timeout value will cause a timeout error and invalidate the transaction.

The supported time units are:

  - `  s  ` : seconds
  - `  ms  ` : milliseconds
  - `  us  ` : microseconds
  - `  ns  ` : nanoseconds

The `  DEFAULT  ` is 0 seconds, which means no timeout. An `  int8  ` number without units indicates `  int8 ms  ` . For example, the following commands both set the statement timeout to 2 seconds.

``` text
SET STATEMENT_TIMEOUT TO 2000;
SET STATEMENT_TIMEOUT TO '2s';
```

A statement timeout during a transaction invalidates the transaction, all subsequent statements in the invalidated transaction (except `  ROLLBACK  ` ) fail.

#### READ\_ONLY\_STALENESS

A property of type `  STRING  ` indicating the current [read-only staleness setting](/spanner/docs/timestamp-bounds) that Spanner uses for read-only transactions and queries in `  AUTOCOMMIT  ` mode.

``` text
SHOW [VARIABLE] SPANNER.READ_ONLY_STALENESS
SET SPANNER.READ_ONLY_STALENESS {TO|=} staleness_type

staleness_type:

{ 'STRONG'
  | 'MIN_READ_TIMESTAMP timestamp'
  | 'READ_TIMESTAMP timestamp'
  | 'MAX_STALENESS <int8>{ s | ms | us | ns }'
  | 'EXACT_STALENESS <int8>{ s | ms | us | ns }' }
```

The [read-only staleness](/spanner/docs/timestamp-bounds) value applies to all subsequent read-only transactions and for all queries in `  AUTOCOMMIT  ` mode.

The default is `  STRONG  ` .

The timestamp bound options are as follows:

  - `  STRONG  ` tells Spanner to perform a [strong read](/spanner/docs/timestamp-bounds#strong) .
  - `  MAX_STALENESS  ` defines the time interval Spanner uses to perform a [bounded staleness read](/spanner/docs/timestamp-bounds#bounded_staleness) , relative to `  now()  ` .
  - `  MIN_READ_TIMESTAMP  ` defines an absolute time Spanner uses to perform a [bounded staleness read](/spanner/docs/timestamp-bounds#bounded_staleness) .
  - `  EXACT_STALENESS  ` defines the time interval Spanner uses to perform an [exact staleness read](/spanner/docs/timestamp-bounds#exact_staleness) , relative to `  now()  ` .
  - `  READ_TIMESTAMP  ` defines an absolute time Spanner uses to perform an exact staleness read.

Timestamps must use the following format:

``` text
YYYY-[M]M-[D]D [[H]H:[M]M:[S]S[.DDDDDD]][timezone]
```

The supported time units for setting `  MAX_STALENESS  ` and `  EXACT_STALENESS  ` values are:

  - `  s  ` : seconds
  - `  ms  ` : milliseconds
  - `  us  ` : microseconds
  - `  ns  ` : nanoseconds

You can modify the value of this property only while there is no active transaction.

**Note:** You can use the values `  MIN_READ_TIMESTAMP  ` and `  MAX_STALENESS  ` only for queries in `  AUTOCOMMIT  ` mode.

##### ▶ Example: Read-only staleness (Click to expand)

The following example shows how to execute queries using a custom staleness value with PGAdapter.

``` text
-- Set the read-only staleness to MAX_STALENESS 10 seconds.
SET SPANNER.READ_ONLY_STALENESS = 'MAX_STALENESS 10s';

-- Execute a query in auto-commit mode. This will return results that are up to
-- 10 seconds stale.
SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Read-only staleness can also be applied to read-only transactions.
-- MAX_STALENESS is however only allowed for queries in autocommit mode.
-- Change the staleness to EXACT_STALENESS and start a read-only transaction.
SET SPANNER.READ_ONLY_STALENESS = 'EXACT_STALENESS 10s';
BEGIN;
SET TRANSACTION READ ONLY;

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

SELECT title, singer_id
FROM albums
ORDER BY title;

COMMIT;

-- Read staleness can also be an exact timestamp.
SET SPANNER.READ_ONLY_STALENESS = 'READ_TIMESTAMP 2024-01-26T10:36:00Z';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;
```

#### SPANNER.OPTIMIZER\_VERSION

A property of type `  STRING  ` indicating the [optimizer version](/spanner/docs/query-optimizer/versions) . The version is either numeric or ' `  LATEST  ` '.

``` text
SHOW [VARIABLE] SPANNER.OPTIMIZER_VERSION
SET SPANNER.OPTIMIZER_VERSION {TO|=} { 'version'|'LATEST'|'' }
```

Sets the version of the optimizer to be used for all the following statements on the connection. Setting the optimizer version to `  ''  ` (the empty string) indicates to use the latest version. If no optimizer version is set, Spanner uses the optimizer version that is set at the database level.

The default is `  ''  ` .

##### ▶ Example: Optimizer version (Click to expand)

The following example shows how to execute queries using a specific [optimizer version](/spanner/docs/query-optimizer/versions) with PGAdapter.

``` text
-- Set the optimizer version to 5 and execute a query.
SET SPANNER.OPTIMIZER_VERSION = '5';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Execute the same query with the latest optimizer version.
SET SPANNER.OPTIMIZER_VERSION = 'LATEST';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Revert back to using the default optimizer version that has been set for the
-- database.
SET SPANNER.OPTIMIZER_VERSION = '';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;
```

#### SPANNER.OPTIMIZER\_STATISTICS\_PACKAGE

A property of type `  STRING  ` indicating the current [optimizer statistics package](/spanner/docs/query-optimizer/manage-query-optimizer) that is used by this connection.

``` text
SHOW [VARIABLE] SPANNER.OPTIMIZER_STATISTICS_PACKAGE
SET SPANNER.OPTIMIZER_STATISTICS_PACKAGE {TO|=} { 'package'|'' }
```

Sets the optimizer statistics package to use for all following statements on the connection. `  <package>  ` must be a valid package name. If no optimizer statistics package is set, Spanner uses the optimizer statistics package that is set at the database level.

The default is `  ''  ` .

##### ▶ Example: Optimizer statistics package (Click to expand)

The following example shows how to execute queries using a specific [optimizer statistics package](/spanner/docs/query-optimizer/versions) with PGAdapter.

``` text
-- Show the available optimizer statistics packages in this database.
SELECT * FROM INFORMATION_SCHEMA.SPANNER_STATISTICS;

-- Set the optimizer statistics package and execute a query.
SET SPANNER.OPTIMIZER_STATISTICS_PACKAGE = 'auto_20240124_06_47_29UTC';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Execute the same query with the default optimizer statistics package.
SET SPANNER.OPTIMIZER_VERSION = '';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;
```

#### SPANNER.RETURN\_COMMIT\_STATS

A property of type `  BOOL  ` indicating whether statistics should be returned for transactions on this connection. You can see returned statistics by executing the `  SHOW [VARIABLE] COMMIT_RESPONSE  ` command.

``` text
SHOW [VARIABLE] SPANNER.RETURN_COMMIT_STATS
SET SPANNER.RETURN_COMMIT_STATS {TO|=} { true | false }
```

The default is `  false  ` .

##### ▶ Example: Commit statistics (Click to expand)

The following example shows how to view commit statistics for a transaction with PGAdapter.

``` text
-- Enable the returning of commit stats.
SET SPANNER.RETURN_COMMIT_STATS = true;

-- Execute a transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);
COMMIT;

-- View the commit response with the transaction statistics for the last
-- transaction that was committed.
SHOW SPANNER.COMMIT_RESPONSE;
```

#### SPANNER.RPC\_PRIORITY

A property of type `  STRING  ` indicating the relative priority for Spanner requests. The priority acts as a hint to the Spanner scheduler and does not guarantee order of execution.

``` text
SHOW [VARIABLE] SPANNER.RPC_PRIORITY
SET SPANNER.RPC_PRIORITY {TO|=} {'HIGH'|'MEDIUM'|'LOW'|'NULL'}
```

`  'NULL'  ` means that no hint should be included in the request.

The default is `  'NULL'  ` .

You can also use a statement hint to specify the RPC priority:

``` text
/*@RPC_PRIORITY=PRIORITY_LOW*/ SELECT * FROM Albums
```

For more information, see [`  Priority  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions.Priority) .

## Transaction statements

The following statements manage and commit Spanner transactions.

#### TRANSACTION ISOLATION LEVEL

``` text
SHOW [ VARIABLE ] TRANSACTION ISOLATION LEVEL
```

Returns a result set with one row and one column of type `  STRING  ` . The returned value is always `  serializable  ` , as this is the only supported isolation level for Spanner PostgreSQL-dialect databases.

#### SPANNER.READ\_TIMESTAMP

``` text
SHOW [VARIABLE] SPANNER.READ_TIMESTAMP
```

Returns a result set with one row and one column of type `  TIMESTAMP  ` containing the read timestamp of the most recent read-only transaction. This statement returns a timestamp only when either a read-only transaction is still active and has executed at least one query, or immediately after a read-only transaction is committed and before a new transaction starts. Otherwise, the result is `  NULL  ` .

##### ▶ Example: Read timestamp (Click to expand)

The following example shows how to view the last read timestamp for a read-only operation with PGAdapter.

``` text
-- Execute a query in autocommit mode using the default read-only staleness
-- (strong).
SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Shows the read timestamp that was used for the previous query.
SHOW SPANNER.READ_TIMESTAMP;

-- Set a non-deterministic read-only staleness and execute the same query.
SET SPANNER.READ_ONLY_STALENESS = 'MAX_STALENESS 20s';

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Shows the read timestamp that was used for the previous query. The timestamp
-- is determined by Spanner, and is guaranteed to be no less than 20
-- seconds stale.
SHOW SPANNER.READ_TIMESTAMP;

-- The read timestamp of a read-only transaction can also be retrieved.
SET SPANNER.READ_ONLY_STALENESS = 'STRONG';
BEGIN;
SET TRANSACTION READ ONLY;

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Shows the read timestamp of the current read-only transaction. All queries in
-- this transaction will use this read timestamp.
SHOW SPANNER.READ_TIMESTAMP;

SELECT title
FROM albums
ORDER BY title;

-- The read timestamp is the same as for the previous query, as all queries in
-- the same transaction use the same read timestamp.
SHOW SPANNER.READ_TIMESTAMP;

COMMIT;
```

#### SPANNER.COMMIT\_TIMESTAMP

``` text
SHOW [VARIABLE] SPANNER.COMMIT_TIMESTAMP
```

Returns a result set with one row and one column of type `  TIMESTAMP  ` containing the commit timestamp of the last read-write transaction that Spanner committed. This statement returns a timestamp only when you execute it after you commit a read-write transaction and before you execute any subsequent `  SELECT  ` , `  DML  ` , or schema change statements. Otherwise, the result is `  NULL  ` .

##### ▶ Example: Commit timestamp (Click to expand)

The following example shows how to view the last commit timestamp for a write operation with PGAdapter.

``` text
-- Execute a DML statement.
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);

-- Show the timestamp that the statement was committed.
SHOW SPANNER.COMMIT_TIMESTAMP;
```

#### SPANNER.COMMIT\_RESPONSE

``` text
SHOW [VARIABLE] SPANNER.COMMIT_RESPONSE
```

Returns a result set with one row and two columns:

  - `  COMMIT_TIMESTAMP  ` (type= `  TIMESTAMP  ` ) Indicates when the most recent transaction was committed.
  - `  MUTATION_COUNT  ` (type= `  int8  ` ) Indicates how many mutations were applied in the committed transaction. This value is always empty when executed on the emulator.

The mutation count is available only if `  SET RETURN_COMMIT_STATS  ` was set to `  true  ` prior to the transaction commit.

##### ▶ Example: Commit response (Click to expand)

The following example shows how to view the last commit response for a write operation with PGAdapter.

``` text
-- Enable returning commit stats in addition to the commit timestamp.
SET SPANNER.RETURN_COMMIT_STATS = true;

-- Execute a DML statement.
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);

-- Show the timestamp that the statement was committed.
SHOW SPANNER.COMMIT_RESPONSE;
```

#### { START | BEGIN } \[ TRANSACTION | WORK \]

``` text
{ START | BEGIN } [ TRANSACTION | WORK ] [{ READ ONLY | READ WRITE }]
```

Starts a new transaction. The keywords `  TRANSACTION  ` and `  WORK  ` are optional, equivalent, and have no effect.

  - Use `  COMMIT  ` or `  ROLLBACK  ` to terminate a transaction.
  - If you have enabled [`  AUTOCOMMIT  ` mode](#autocommit) , this statement temporarily takes the connection out of `  AUTOCOMMIT  ` mode. The connection returns to `  AUTOCOMMIT  ` mode when the transaction ends.
  - If `  READ ONLY  ` or `  READ WRITE  ` is not specified, the transaction mode is determined by the default transaction mode of the session. This default is set by using the `  SET SESSION CHARACTERISTICS AS TRANSACTION  ` command.

You can execute this statement only while there is no active transaction.

##### ▶ Example: BEGIN TRANSACTION (Click to expand)

The following example shows how to start different types of transactions with PGAdapter.

``` text
-- This starts a transaction using the current defaults of this connection.
-- The value of SPANNER.READONLY determines whether the transaction is a
-- read/write or a read-only transaction.

BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
COMMIT;

-- Set SPANNER.READONLY to TRUE to use read-only transactions by default.
SET SPANNER.READONLY=TRUE;

-- This starts a read-only transaction.
BEGIN;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
COMMIT;

-- Use the 'READ WRITE' or 'READ ONLY' qualifier in the BEGIN statement to
-- override the current default of the connection.
SET SPANNER.READONLY=FALSE;
BEGIN READ ONLY;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
COMMIT;
```

#### COMMIT \[TRANSACTION | WORK\]

``` text
COMMIT [TRANSACTION | WORK]
```

Commits the current transaction. The keywords `  TRANSACTION  ` and `  WORK  ` are optional and equivalent, and have no effect.

  - Committing a read-write transaction makes all updates of this transaction visible to other transactions and releases all of the transaction's locks on Spanner.
  - Committing a read-only transaction ends the current read-only transaction. Any subsequent statement starts a new transaction. There is no semantic difference between `  COMMIT  ` and `  ROLLBACK  ` for a read-only transaction.

You can execute this statement only while there is an active transaction.

##### ▶ Example: COMMIT TRANSACTION (Click to expand)

The following example shows how to commit a transaction with PGAdapter.

``` text
-- Execute a regular read/write transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
COMMIT;

-- Execute a read-only transaction. Read-only transactions also need to be
-- either committed or rolled back in PGAdapter in order to mark the
-- end of the transaction.
BEGIN READ ONLY;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
COMMIT;
```

#### ROLLBACK \[TRANSACTION | WORK\]

``` text
ROLLBACK [TRANSACTION | WORK]
```

Performs a `  ROLLBACK  ` of the current transaction. The keywords `  TRANSACTION  ` and `  WORK  ` are optional and equivalent, and have no effect.

  - Performing a `  ROLLBACK  ` of a read-write transaction clears any buffered mutations, rolls back the transaction on Spanner, and releases any locks the transaction held.
  - Performing a `  ROLLBACK  ` of a read-only transaction ends the current read-only transaction. Any subsequent statements start a new transaction. There is no semantic difference between `  COMMIT  ` and `  ROLLBACK  ` for a read-only transaction on a connection.

You can execute this statement only while there is an active transaction.

##### ▶ Example: ROLLBACK TRANSACTION (Click to expand)

The following example shows how to rollback a transaction with PGAdapter.

``` text
-- Use ROLLBACK to undo the effects of a transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
-- This will ensure that the insert statement is not persisted in the database.
ROLLBACK;

-- Read-only transactions also need to be either committed or rolled back in
-- PGAdapter in order to mark the end of the transaction. There is no
-- semantic difference between rolling back or committing a read-only
-- transaction.
BEGIN READ ONLY;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
ROLLBACK;
```

#### SET TRANSACTION

``` text
SET TRANSACTION { READ ONLY | READ WRITE }
```

Sets the transaction mode for the current transaction.

You can execute this statement only when `  AUTOCOMMIT  ` is `  false  ` , or if you have started a transaction by executing `  BEGIN [TRANSACTION | WORK]  ` and have not yet executed any statements in the transaction.

This statement sets the transaction mode for the current transaction only. When the transaction commits or rolls back, the next transaction uses the default mode for the connection. (See [`  SET SESSION CHARACTERISTICS  `](#session_characteristics) .)

**Note:** You can't set the transaction mode to `  READ WRITE  ` if the connection is in `  READ ONLY  ` mode.

##### ▶ Example: SET TRANSACTION (Click to expand)

The following example shows how to set transaction characteristics with PGAdapter.

``` text
-- Start a transaction and set the transaction mode to read-only.
BEGIN;
SET TRANSACTION READ ONLY;

SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- Commit the read-only transaction to mark the end of the transaction.
COMMIT;

-- Start a transaction and set the transaction mode to read/write.
BEGIN;
SET TRANSACTION READ WRITE;

INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);

COMMIT;
```

#### SET SESSION CHARACTERISTICS

``` text
SET SESSION CHARACTERISTICS AS TRANSACTION { READ ONLY | READ WRITE }
```

Sets the default transaction mode for transactions in the session to `  READ ONLY  ` or `  READ WRITE  ` . This statement is permitted only when there is no active transaction.

The [`  SET TRANSACTION  `](#set_transaction_mode) command can override this setting.

##### ▶ Example: SET SESSION CHARACTERISTICS (Click to expand)

The following example shows how to set session characteristics with PGAdapter.

``` text
-- Set the default transaction mode to read-only.
SET SESSION CHARACTERISTICS AS TRANSACTION READ ONLY;

-- This will now start a read-only transaction.
BEGIN;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
COMMIT;

-- You can override the default transaction mode with the SET TRANSACTION
-- statement.
SET SESSION CHARACTERISTICS AS TRANSACTION READ WRITE;
BEGIN;
SET TRANSACTION READ ONLY;
SELECT first_name, last_name
FROM singers
ORDER BY last_name;
COMMIT;
```

#### SPANNER.STATEMENT\_TAG

A property of type `  STRING  ` that contains the request tag for the next statement.

``` text
SHOW [ VARIABLE ] SPANNER.STATEMENT_TAG
SET SPANNER.STATEMENT_TAG {TO|=} 'tag-name'
```

Sets the request tag for the next statement to be executed. Only one tag can be set per statement. The tag doesn't span multiple statements; it must be set on a per statement basis. A request tag can be removed by setting it to the empty string ( `  ''  ` ).

The default is `  ''  ` .

You can set both transaction tags and statement tags for the same statement.

You can also use a statement hint to add a statement tag:

``` text
/*@STATEMENT_TAG='my-tag'*/ SELECT * FROM albums
```

For more information, see [Troubleshooting with request tags and transaction tags](/spanner/docs/introspection/troubleshooting-with-tags) .

##### ▶ Example: Statement tags (Click to expand)

The following example shows how to set statement tags with PGAdapter.

``` text
-- Set the statement tag that should be included with the next statement.
SET SPANNER.STATEMENT_TAG = 'tag1';
SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- The statement tag property is cleared after each statement execution.
SHOW SPANNER.STATEMENT_TAG;
-- Set another tag for the next statement.
SET SPANNER.STATEMENT_TAG = 'tag2';
SELECT title
FROM albums
ORDER BY title;

-- Set a statement tag with a query hint.
/*@STATEMENT_TAG='tag3'*/
SELECT track_number, title
FROM tracks
WHERE album_id=1 AND singer_id=1
ORDER BY track_number;
```

#### SPANNER.TRANSACTION\_TAG

A property of type `  STRING  ` that contains the transaction tag for the next transaction.

``` text
SHOW [ VARIABLE ] SPANNER.TRANSACTION_TAG
SET SPANNER.TRANSACTION_TAG {TO|=} 'tag-name'
```

Sets the transaction tag for the current transaction to be executed. Only one tag can be set per transaction. The tag doesn't span multiple transactions; it must be set on a per transaction basis. A transaction tag can be removed by setting it to the empty string ( `  ''  ` ). The transaction tag must be set before any statements have been executed in the transaction.

The default is `  ''  ` .

You can set both transaction tags and statement tags for the same statement.

For more information, see [Troubleshooting with request tags and transaction tags](/spanner/docs/introspection/troubleshooting-with-tags) .

##### ▶ Example: Transaction tags (Click to expand)

The following example shows how to set transaction tags with PGAdapter.

``` text
BEGIN;
-- Set the transaction tag for the current transaction.
SET SPANNER.TRANSACTION_TAG = 'transaction-tag-1';

-- Set the statement tag that should be included with the next statement.
-- The statement will include both the statement tag and the transaction tag.
SET SPANNER.STATEMENT_TAG = 'select-statement';
SELECT first_name, last_name
FROM singers
ORDER BY last_name;

-- The statement tag property is cleared after each statement execution.
SHOW SPANNER.STATEMENT_TAG;

-- Set another tag for the next statement.
SET SPANNER.STATEMENT_TAG = 'insert-statement';
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);

COMMIT;

-- The transaction tag property is cleared when the transaction finishes.
SHOW SPANNER.TRANSACTION_TAG;
```

## Batch statements

The following statements manage batches of DDL statements and send those batches to Spanner.

#### START BATCH DDL

``` text
START BATCH DDL
```

Starts a batch of DDL statements on the connection. All subsequent statements during the batch must be DDL statements. The DDL statements are buffered locally and sent to Spanner as one batch when you execute `  RUN BATCH  ` . Executing multiple DDL statements as one batch is typically faster than running the statements separately.

You can execute this statement only while there is no active transaction.

##### ▶ Example: DDL batch (Click to expand)

The following example shows how to execute a DDL batch with PGAdapter.

``` text
-- Start a DDL batch. All following statements must be DDL statements.
START BATCH DDL;

-- This statement is buffered locally until RUN BATCH is executed.
CREATE TABLE singers (
  id bigint primary key,
  first_name varchar,
  last_name varchar
);

-- This statement is buffered locally until RUN BATCH is executed.
CREATE TABLE albums (
  id bigint primary key,
  title varchar,
  singer_id bigint,
  constraint fk_albums_singers foreign key (singer_id) references singers (id)
);

-- This runs the DDL statements as one batch.
RUN BATCH;
```

#### RUN BATCH

``` text
RUN BATCH
```

Sends all buffered DDL statements in the current DDL batch to the database, waits for Spanner to execute these statements, and ends the current DDL batch.

If Spanner cannot execute at least one DDL statement, `  RUN BATCH  ` returns an error for the first DDL statement that Spanner cannot execute. Otherwise, `  RUN BATCH  ` returns successfully.

**Note:** If a DDL statement in the batch returns an error, Spanner might still have applied the preceding DDL statements in the same batch to the database.

#### ABORT BATCH

Clears all buffered DDL statements in the current DDL batch and ends the batch.

You can execute this statement only when a DDL batch is active. You can use `  ABORT BATCH  ` regardless of whether or not the batch has buffered DDL statements. All preceding DDL statements in the batch will be aborted.

##### ▶ Example: Abort DDL batch (Click to expand)

The following example shows how to abort a DDL batch with PGAdapter.

``` text
-- Start a DDL batch. All following statements must be DDL statements.
START BATCH DDL;

-- The following statements are buffered locally.
CREATE TABLE singers (
  id bigint primary key,
  first_name varchar,
  last_name varchar
);
CREATE TABLE albums (
  id bigint primary key,
  title varchar,
  singer_id bigint,
  constraint fk_albums_singers foreign key (singer_id) references singers (id)
);

-- This aborts the DDL batch and removes the DDL statements from the buffer.
ABORT BATCH;
```

#### START BATCH DML

The following statements batch the two DML statements together and send these in one call to the server. A DML batch can be executed as part of a transaction or in autocommit mode.

``` text
START BATCH DML;
INSERT INTO MYTABLE (ID, NAME) VALUES (1, 'ONE');
INSERT INTO MYTABLE (ID, NAME) VALUES (2, 'TWO');
RUN BATCH;
```

##### ▶ Example: DML batch (Click to expand)

The following example shows how to execute a DML batch with PGAdapter.

``` text
-- Start a DML batch. All following statements must be a DML statement.
START BATCH DML;

-- The following statements are buffered locally.
INSERT INTO MYTABLE (ID, NAME) VALUES (1, 'ONE');
INSERT INTO MYTABLE (ID, NAME) VALUES (2, 'TWO');

-- This sends the statements to Spanner.
RUN BATCH;

-- DML batches can also be part of a read/write transaction.
BEGIN;
-- Insert a row using a single statement.
INSERT INTO MYTABLE (ID, NAME) VALUES (3, 'THREE');

-- Insert two rows using a batch.
START BATCH DML;
INSERT INTO MYTABLE (ID, NAME) VALUES (4, 'FOUR');
INSERT INTO MYTABLE (ID, NAME) VALUES (5, 'FIVE');
RUN BATCH;

-- Rollback the current transaction. This rolls back both the single DML
-- statement and the DML batch.
ROLLBACK;
```

## Savepoint commands

Savepoints in PGAdapter are emulated. Rolling back to a savepoint rolls back the entire transaction and retries it up to the point where the savepoint was set. This operation will fail with an `  AbortedDueToConcurrentModificationException  ` error if the underlying data that has been used by the transaction up to the savepoint has changed.

Creating and releasing savepoints always succeeds when savepoint support is enabled.

The following statements enable and disable emulated savepoints in transactions.

#### SPANNER.SAVEPOINT\_SUPPORT

``` text
SHOW [VARIABLE] SPANNER.SAVEPOINT_SUPPORT
SET SPANNER.SAVEPOINT_SUPPORT = { 'DISABLED' | 'FAIL_AFTER_ROLLBACK' | 'ENABLED' }
```

A property of type `  STRING  ` indicating the current `  SAVEPOINT_SUPPORT  ` configuration. Possible values are:

  - `  DISABLED  ` : All savepoint commands are disabled and will fail.
  - `  FAIL_AFTER_ROLLBACK  ` : Savepoint commands are enabled. Rolling back to a savepoint rolls back the entire transaction. The operation fails if you try to use the transaction after rolling back to a savepoint.
  - `  ENABLED  ` : All savepoint commands are enabled. Rolling back to a savepoint rolls back the transaction and retry is performed to the savepoint. This operation fails with an `  AbortedDueToConcurrentModificationException  ` error if the underlying data that has been used by the transaction up to the savepoint has changed.

The default value is `  ENABLED  ` .

You can execute this statement only while there is no active transaction.

#### SAVEPOINT savepoint\_name

``` text
SAVEPOINT savepoint-name;
```

`  SAVEPOINT  ` creates a new savepoint within the current transaction. A transaction can be rolled back to a savepoint to undo all operations that have been executed since the savepoint was created.

##### ▶ Example: Savepoint (Click to expand)

The following example shows how to savepoints with PGAdapter.

``` text
-- Start a transaction and execute an insert statement.
BEGIN;
INSERT INTO T (id, col_a, col_b) VALUES (1, 100, 1);

-- Set a savepoint and then execute another insert statement.
SAVEPOINT one_row_inserted;
INSERT INTO T (id, col_a, col_b) VALUES (2, 200, 2);

-- Roll back to the savepoint. This will undo all statements that have been
-- executed after the savepoint.
ROLLBACK TO one_row_inserted;

-- This only commits the first insert statement.
COMMIT;
```

#### ROLLBACK TO savepoint\_name

``` text
ROLLBACK TO savepoint_name
```

Rolls back the current transaction to the savepoint with the specified name.

Rolling back to a savepoint is not guaranteed to be successful in all cases. Rolling back to a savepoint rolls back the entire transaction and retries it up to the point where the savepoint was set. This operation will fail with an `  AbortedDueToConcurrentModificationException  ` if the underlying data that has been used by the transaction up to the savepoint has changed.

#### RELEASE \[SAVEPOINT\] savepoint\_name

``` text
RELEASE savepoint_name
```

Removes the savepoint from the current transaction. It can no longer be used to execute a `  ROLLBACK TO savepoint_name  ` statement.

## Prepared Statements

The following statements create and execute prepared statements.

#### PREPARE

``` text
PREPARE statement_name [(data_type, ...)] AS statement
```

Prepares a statement on this connection. The statement is parsed and validated by Spanner and stored in memory in PGAdapter.

##### ▶ Example: Prepared statements (Click to expand)

The following example shows how to create and execute prepared statements with PGAdapter.

``` text
-- Create a prepared statement that can be used to insert a single row.
PREPARE insert_t AS INSERT INTO T (id, col_a, col_b) VALUES ($1, $2, $3);

-- The prepared statement can be used to insert rows both in autocommit, in a
-- transaction, and in DML batches.

-- Execute in autocommit.
EXECUTE insert_t (1, 100, 1);

-- Execute in transaction.
BEGIN;
EXECUTE insert_t (2, 200, 2);
EXECUTE insert_t (3, 300, 3);
COMMIT;

-- Execute in a DML batch.
START BATCH DML;
EXECUTE insert_t (4, 400, 4);
EXECUTE insert_t (5, 500, 5);
RUN BATCH;

-- Prepared statements can be removed with the DEALLOCATE command.
DEALLOCATE insert_t;
```

#### EXECUTE

``` text
EXECUTE statement_name [(value, ...)]
```

Executes a statement that has been created on this connection with `  PREPARE  ` .

##### ▶ Example: Execute (Click to expand)

The following example shows how to prepare and execute statements with PGAdapter.

``` text
-- Create a prepared statement.
PREPARE my_statement AS insert into my_table (id, value) values ($1, $2);

-- Execute the statement twice with different parameter values.
EXECUTE my_statement (1, 'One');
EXECUTE my_statement (2, 'Two');
```

#### DEALLOCATE

``` text
DEALLOCATE statement_name
```

Removes a prepared statement from this connection.

## Copy

PGAdapter supports a subset of the PostgreSQL `  COPY  ` command.

#### COPY table\_name FROM STDIN

``` text
COPY table_name FROM STDIN [BINARY]
```

Copies data from `  stdin  ` to Spanner. It is more efficient to use `  COPY  ` to import a large dataset to Spanner than to execute `  INSERT  ` statements.

`  COPY  ` can be combined with `  SPANNER.AUTOCOMMIT_DML_MODE  ` to execute a non-atomic transaction. This allows the transaction to execute more mutations than the standard transaction mutation limit.

##### ▶ Example: Copy (Click to expand)

The following examples show how to copy data to and from Spanner with PGAdapter.

``` text
create table numbers (number bigint not null primary key, name varchar);
```

Execute an atomic `  COPY  ` operation:

``` text
cat numbers.txt | psql -h /tmp -d test-db -c "copy numbers from stdin;"
```

Execute a non-atomic `  COPY  ` operation:

``` text
cat numbers.txt | psql -h /tmp -d test-db \
  -c "set spanner.autocommit_dml_mode='partitioned_non_atomic'; copy numbers from stdin;"
```

Copy data from PostgreSQL to Spanner:

``` text
psql -h localhost -p 5432 -d my-local-db \
  -c "copy (select i, to_char(i, 'fm000') from generate_series(1, 1000000) s(i)) to stdout binary" \
  | psql -h localhost -p 5433 -d my-spanner-db \
  -c "set spanner.autocommit_dml_mode='partitioned_non_atomic'; copy numbers from stdin binary;"
```

This example assumes that PostgreSQL runs on port 5432 and PGAdapter runs on port 5433.

For more examples, see [PGAdapter - COPY support](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/copy.md) .

#### COPY table\_name TO STDOUT \[BINARY\]

``` text
COPY table_name TO STDOUT [BINARY]
```

Copies the data in a table or from a query to `  stdout  ` .

### Data Boost and partitioned query statements

[Data Boost](/spanner/docs/databoost/databoost-overview) lets you execute analytics queries and data exports with near-zero impact to existing workloads on the provisioned Spanner instance. Data boost is only supported for [partitioned queries](/spanner/docs/reads#read_data_in_parallel) .

You can enable Data Boost with the `  SET SPANNER.DATA_BOOST_ENABLED  ` statement.

PGAdapter supports three alternatives for executing partitioned queries:

  - `  SET SPANNER.AUTO_PARTITION_MODE = true  `
  - `  RUN PARTITIONED QUERY sql  `
  - `  PARTITION sql  ` followed by multiple `  RUN PARTITION 'partition-token'  `

Each of these methods are described in the following sections.

#### SPANNER.DATA\_BOOST\_ENABLED

``` text
SHOW SPANNER.DATA_BOOST_ENABLED
SET SPANNER.DATA_BOOST_ENABLED {TO|=} { true | false }
```

Sets whether this connection should use [Data Boost](/spanner/docs/databoost/databoost-overview) for partitioned queries.

The default is `  false  ` .

##### ▶ Example: Execute a query using Data Boost (Click to expand)

The following example shows how to a query using Data Boost with PGAdapter.

``` text
-- Enable Data Boost on this connection.
SET SPANNER.DATA_BOOST_ENABLED = true;

-- Execute a partitioned query. Data Boost is only used for partitioned queries.
RUN PARTITIONED QUERY SELECT FirstName, LastName FROM Singers;
```

#### SPANNER.AUTO\_PARTITION\_MODE

``` text
SHOW SPANNER.AUTO_PARTITION_MODE
SET SPANNER.AUTO_PARTITION_MODE {TO|=} { true | false}
```

A property of type `  BOOL  ` indicating whether the connection automatically uses partitioned queries for all queries that are executed.

  - Set this variable to `  true  ` if you want the connection to use partitioned query for all queries that are executed.
  - Also set `  SPANNER.DATA_BOOST_ENABLED  ` to `  true  ` if you want the connection to use [Data Boost](/spanner/docs/databoost/databoost-overview) for all queries.

The default is `  false  ` .

##### ▶ Example: Execute (Click to expand)

This example executes two queries with PGAdapter using [Data Boost](/spanner/docs/databoost/databoost-overview)

``` text
SET SPANNER.AUTO_PARTITION_MODE = true
SET SPANNER.DATA_BOOST_ENABLED = true
SELECT first_name, last_name FROM singers
SELECT singer_id, title FROM albums
```

#### RUN PARTITIONED QUERY

``` text
RUN PARTITIONED QUERY <sql>
```

Executes a query as a partitioned query on Spanner. Ensure that `  SPANNER.DATA_BOOST_ENABLED  ` is set to `  true  ` to execute the query with [Data Boost](/spanner/docs/databoost/databoost-overview) :

``` text
SET SPANNER.DATA_BOOST_ENABLED = true
RUN PARTITIONED QUERY SELECT FirstName, LastName FROM Singers
```

PGAdapter internally partitions the query and executes partitions in parallel. The results are merged into one result set and returned to the application. The number of worker threads executing partitions can be set with the variable `  SPANNER.MAX_PARTITIONED_PARALLELISM  ` .

#### PARTITION \<SQL\>

``` text
PARTITION <sql>
```

Creates a list of partitions to execute a query against Spanner and returns these a list of partition tokens. Each partition token can be executed on a separate connection on the same or another PGAdapter instance using the `  RUN PARTITION 'partition-token'  ` command.

##### ▶ Example: Partition query (Click to expand)

The following example shows how to partition a query and then execute each partition separately using PGAdapter.

``` text
-- Partition a query. This returns a list of partition tokens that can be
-- executed either on this connection or on any other connection to the same
-- database.
PARTITION SELECT FirstName, LastName FROM Singers;

-- Run the partitions that were returned from the previous statement.
RUN PARTITION 'partition-token-1';
RUN PARTITION 'partition-token-2';
```

#### RUN PARTITION 'partition-token'

``` text
RUN PARTITION 'partition-token'
```

Executes a query partition that has previously been returned by the `  PARTITION  ` command. The command can be executed on any connection that is connected to the same database as the database that created the partition tokens.

#### SPANNER.MAX\_PARTITIONED\_PARALLELISM

A property of type `  bigint  ` indicating the number of worker threads PGAdapter uses to execute partitions. This value is used for:

  - `  SPANNER.AUTO_PARTITION_MODE = true  `
  - `  RUN PARTITIONED QUERY sql  `

<!-- end list -->

``` text
SHOW SPANNER.MAX_PARTITIONED_PARALLELISM
SET SPANNER.MAX_PARTITIONED_PARALLELISM {TO|=} <bigint>
```

Sets the maximum number of worker threads that PGAdapter can use to execute partitions. Setting this value to `  0  ` instructs PGAdapter to use the number of CPU cores on the client machine as the maximum.

The default is `  0  ` .

## What's next

  - Learn more about how to [Start PGAdapter](/spanner/docs/pgadapter-start) .
  - See a full list of supported [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql_drivers_and_orms) .
