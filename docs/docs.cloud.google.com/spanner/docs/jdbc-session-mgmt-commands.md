The Spanner JDBC driver (Java Database Connectivity) supports session management statements, which let you modify the state of your connection, execute transactions, and efficiently execute batches of statements.

The following commands apply to GoogleSQL-dialect databases.

**Note:** You can use session management statements only with the Spanner JDBC driver. Session management statements don't work in the client libraries, the Google Cloud CLI, or the JDBC driver created by Simba.

### Connection statements

The following statements make changes to or display properties of the current connection.

#### READONLY

A boolean indicating whether or not the connection is in read-only mode. The default is `  false  ` .

``` text
SHOW VARIABLE READONLY
SET READONLY = { true | false }
```

You can change the value of this property only while there is no active transaction.

##### Example: Read-only transaction (Click to expand)

The following example shows how to use this property to execute read-only transactions in Spanner.

``` text
SET READONLY = TRUE;
-- This transaction is a read-only transaction.
BEGIN TRANSACTION;

-- The following two queries both use the read-only transaction.
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

SELECT Title
FROM Albums
ORDER BY Title;

-- This shows the read timestamp that was used for the two queries.
SHOW VARIABLE READ_TIMESTAMP;

-- This marks the end of the read-only transaction. The next statement starts
-- a new read-only transaction.
COMMIT;
```

#### AUTOCOMMIT

A boolean indicating whether or not the connection is in autocommit mode. The default is `  true  ` .

``` text
SHOW VARIABLE AUTOCOMMIT
SET AUTOCOMMIT = { true | false }
```

You can change the value of this property only when there is no active transaction.

When `  AUTOCOMMIT  ` is set to false, a new transaction is initiated automatically after you execute `  COMMIT  ` or `  ROLLBACK  ` . The first statement that you execute starts the transaction.

##### Example: Autocommit (Click to expand)

The following example shows how to use the \`autocommit\` property.

``` text
-- The default value for AUTOCOMMIT is true.
SHOW VARIABLE AUTOCOMMIT;

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

#### RETRY\_ABORTS\_INTERNALLY

A boolean indicating whether the connection automatically retries aborted transactions. The default is `  true  ` .

``` text
SHOW VARIABLE RETRY_ABORTS_INTERNALLY
SET RETRY_ABORTS_INTERNALLY = { true | false }
```

You can change the value of this property only after a transaction has started (see [`  BEGIN TRANSACTION  `](#start_a_transaction) ) and before any statements are executed within the transaction.

When you set `  RETRY_ABORTS_INTERNALLY  ` to true, the connection keeps a checksum of all data that the connection returns to the client application. This is used to retry the transaction if it is aborted by Spanner.

The default value is `  true  ` . We recommend setting this value to `  false  ` if your application already retries aborted transactions.

#### AUTOCOMMIT\_DML\_MODE

A `  STRING  ` property indicating the autocommit mode for [Data Manipulation Language (DML)](/spanner/docs/reference/standard-sql/dml-syntax) statements.

``` text
SHOW VARIABLE AUTOCOMMIT_DML_MODE
SET AUTOCOMMIT_DML_MODE = { 'TRANSACTIONAL' | 'PARTITIONED_NON_ATOMIC' }
```

The possible values are:

  - In `  TRANSACTIONAL  ` mode, the driver executes DML statements as separate atomic transactions. The driver creates a new transaction, executes the DML statement, and either commits the transaction upon successful execution or rolls back the transaction in the case of an error.
  - In `  PARTITIONED_NON_ATOMIC  ` mode, the driver executes DML statements as [partitioned update statements](/spanner/docs/dml-partitioned) . A partitioned update statement can run as a series of many transactions, each covering a subset of the rows impacted. The partitioned statement provides weakened semantics in exchange for better scalability and performance.

The default is `  TRANSACTIONAL  ` .

##### Example: Partitioned DML (Click to expand)

The following example shows how to execute \[Partitioned DML\](/spanner/docs/dml-partitioned) using the Spanner JDBC driver.

``` text
-- Change autocommit DML mode to use Partitioned DML.
SET AUTOCOMMIT_DML_MODE = 'PARTITIONED_NON_ATOMIC';

-- Delete all singers that have been marked as inactive.
-- This statement is executed using Partitioned DML.
DELETE
FROM singers
WHERE active=false;

-- Change DML mode back to standard `TRANSACTIONAL`.
SET AUTOCOMMIT_DML_MODE = 'TRANSACTIONAL';
```

#### STATEMENT\_TIMEOUT

A property of type `  STRING  ` indicating the current timeout value for statements.

``` text
SHOW VARIABLE STATEMENT_TIMEOUT
SET STATEMENT_TIMEOUT = { '<INT64>{ s | ms | us | ns }' | NULL }
```

The `  INT64  ` value is a whole number followed by a suffix indicating the time unit. A value of `  NULL  ` indicates that there is no timeout value set. If a statement timeout value has been set, statements that take longer than the specified timeout value will cause a `  java.sql.SQLTimeoutException  ` error and invalidate the transaction.

The supported time units are:

  - `  s  ` : seconds
  - `  ms  ` : milliseconds
  - `  us  ` : microseconds
  - `  ns  ` : nanoseconds

The default is `  NULL  ` , which means no timeout value is set.

A statement timeout during a transaction invalidates the transaction, all subsequent statements in the invalidated transaction (except `  ROLLBACK  ` ) fail, and the Spanner JDBC driver throws a `  java.sql.SQLTimeoutException  ` .

#### READ\_ONLY\_STALENESS

A property of type `  STRING  ` indicating the current [read-only staleness setting](/spanner/docs/timestamp-bounds) that Spanner uses for read-only transactions and queries in `  AUTOCOMMIT  ` mode.

``` text
SHOW VARIABLE READ_ONLY_STALENESS
SET READ_ONLY_STALENESS = staleness_type

staleness_type:

{ 'STRONG'
  | 'MIN_READ_TIMESTAMP timestamp'
  | 'READ_TIMESTAMP timestamp'
  | 'MAX_STALENESS <INT64>{ s | ms | us | ns }'
  | 'EXACT_STALENESS <INT64>{ s | ms | us | ns }' }
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
YYYY-[M]M-[D]DT[[H]H:[M]M:[S]S[.DDDDDD]][timezone]
```

The supported time units for setting `  MAX_STALENESS  ` and `  EXACT_STALENESS  ` values are:

  - `  s  ` : seconds
  - `  ms  ` : milliseconds
  - `  us  ` : microseconds
  - `  ns  ` : nanoseconds

You can modify the value of this property only while there is no active transaction.

**Note:** You can use the values `  MIN_READ_TIMESTAMP  ` and `  MAX_STALENESS  ` only for queries in `  AUTOCOMMIT  ` mode.

##### Example: Read-only staleness (Click to expand)

The following example shows how to execute queries using a custom staleness value with the Spanner JDBC driver.

``` text
-- Set the read-only staleness to MAX_STALENESS 10 seconds.
SET READ_ONLY_STALENESS = 'MAX_STALENESS 10s';

-- Execute a query in auto-commit mode. This returns results that are up to
-- 10 seconds stale.
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Read-only staleness can also be applied to read-only transactions.
-- MAX_STALENESS is only allowed for queries in autocommit mode.
-- Change the staleness to EXACT_STALENESS and start a read-only transaction.
SET READ_ONLY_STALENESS = 'EXACT_STALENESS 10s';
BEGIN;
SET TRANSACTION READ ONLY;

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

SELECT Title, SingerId
FROM Albums
ORDER BY Title;

COMMIT;

-- Set the read staleness to an exact timestamp.
SET READ_ONLY_STALENESS = 'READ_TIMESTAMP 2024-01-26T10:36:00Z';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
```

#### OPTIMIZER\_VERSION

A property of type `  STRING  ` indicating the [optimizer version](/spanner/docs/query-optimizer/versions) . The version is either an integer or ' `  LATEST  ` '.

``` text
SHOW VARIABLE OPTIMIZER_VERSION
SET OPTIMIZER_VERSION = { 'version'|'LATEST'|'' }
```

Sets the version of the optimizer to be used for all the following statements on the connection. If the optimizer version is set to `  ''  ` (the empty string), then Spanner uses the latest version. If no optimizer version is set, Spanner uses the optimizer version that is set at the database level.

The default is `  ''  ` .

##### Example: Optimizer version (Click to expand)

The following example shows how to execute queries using a specific \[optimizer version\](/spanner/docs/query-optimizer/versions) with the Spanner JDBC driver.

``` text
-- Set the optimizer version to 5 and execute a query.
SET OPTIMIZER_VERSION = '5';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Execute the same query with the latest optimizer version.
SET OPTIMIZER_VERSION = 'LATEST';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Revert back to using the default optimizer version that has been set for the
-- database.
SET OPTIMIZER_VERSION = '';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
```

#### OPTIMIZER\_STATISTICS\_PACKAGE

A property of type `  STRING  ` indicating the current [optimizer statistics package](/spanner/docs/query-optimizer/manage-query-optimizer) that is used by this connection.

``` text
SHOW VARIABLE OPTIMIZER_STATISTICS_PACKAGE
SET OPTIMIZER_STATISTICS_PACKAGE = { 'package'|'' }
```

Sets the optimizer statistics package to use for all following statements on the connection. `  <package>  ` must be a valid package name. If no optimizer statistics package is set, Spanner uses the optimizer statistics package that is set at the database level.

The default is `  ''  ` .

##### Example: Optimizer statistics package (Click to expand)

The following example shows how to execute queries using a specific \[optimizer statistics package\](/spanner/docs/query-optimizer/versions) with the Spanner JDBC driver.

``` text
-- Show the available optimizer statistics packages in this database.
SELECT * FROM INFORMATION_SCHEMA.SPANNER_STATISTICS;

-- Set the optimizer statistics package and execute a query.
SET OPTIMIZER_STATISTICS_PACKAGE = 'auto_20240124_06_47_29UTC';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Execute the same query with the default optimizer statistics package.
SET OPTIMIZER_STATISTICS_PACKAGE = '';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
```

#### RETURN\_COMMIT\_STATS

A property of type `  BOOL  ` indicating whether statistics should be returned for transactions on this connection. You can see returned statistics by executing the `  SHOW VARIABLE COMMIT_RESPONSE  ` command.

``` text
SHOW VARIABLE RETURN_COMMIT_STATS
SET RETURN_COMMIT_STATS = { true | false }
```

The default is `  false  ` .

##### Example: Commit statistics (Click to expand)

The following example shows how to view commit statistics for a transaction with the Spanner JDBC driver.

``` text
-- Enable the returning of commit stats.
SET RETURN_COMMIT_STATS = true;

-- Execute a transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);
COMMIT;

-- View the commit response with the transaction statistics for the last
-- transaction that was committed.
SHOW VARIABLE COMMIT_RESPONSE;
```

#### RPC\_PRIORITY

A property of type `  STRING  ` indicating the relative priority for Spanner requests. The priority acts as a hint to the Spanner scheduler and doesn't guarantee order of execution.

``` text
SHOW VARIABLE RPC_PRIORITY
SET RPC_PRIORITY = {'HIGH'|'MEDIUM'|'LOW'|'NULL'}
```

`  'NULL'  ` means that no hint should be included in the request.

The default is `  'NULL'  ` .

You can also use a statement hint to specify the RPC priority:

``` text
@{RPC_PRIORITY=PRIORITY_LOW} SELECT * FROM Albums
```

For more information, see [`  Priority  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.RequestOptions.Priority) .

### Tags

The following statements manage [request and transaction tags](/spanner/docs/introspection/troubleshooting-with-tags) .

#### STATEMENT\_TAG

A property of type `  STRING  ` that contains the request tag for the next statement.

``` text
SHOW VARIABLE STATEMENT_TAG
SET STATEMENT_TAG = 'tag-name'
```

Sets the request tag for the next statement to be executed. Only one tag can be set per statement. The tag doesn't span multiple statements; it must be set on a per statement basis. A request tag can be removed by setting it to the empty string ( `  ''  ` ).

The default is `  ''  ` .

You can set both transaction tags and statement tags for the same statement.

You can also use a statement hint to add a statement tag:

``` text
@{STATEMENT_TAG='my-tag'} SELECT * FROM Albums
```

For more information, see [Troubleshoot with request tags and transaction tags](/spanner/docs/introspection/troubleshooting-with-tags) .

##### Example: Statement tags (Click to expand)

The following example shows how to set statement tags with the Spanner JDBC driver.

``` text
-- Set the statement tag that should be included with the next statement.
SET STATEMENT_TAG = 'tag1';
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- The statement tag property is cleared after each statement execution.
SHOW VARIABLE STATEMENT_TAG;
-- Set another tag for the next statement.
SET STATEMENT_TAG = 'tag2';
SELECT Title
FROM Albums
ORDER BY Title;

-- Set a statement tag with a query hint.
@{STATEMENT_TAG = 'tag3'}
SELECT TrackNumber, Title
FROM Tracks
WHERE AlbumId=1 AND SingerId=1
ORDER BY TrackNumber;
```

#### TRANSACTION\_TAG

A property of type `  STRING  ` that contains the transaction tag for the next transaction.

``` text
SHOW VARIABLE TRANSACTION_TAG
SET TRANSACTION_TAG = 'tag-name'
```

Sets the transaction tag for the current transaction to be executed. Only one tag can be set per transaction. The tag doesn't span multiple transactions; it must be set on a per transaction basis. A transaction tag can be removed by setting it to the empty string ( `  ''  ` ). The transaction tag must be set before any statements have been executed in the transaction.

The default is `  ''  ` .

You can set both transaction tags and statement tags for the same statement.

For more information, see [Troubleshoot with request tags and transaction tags](/spanner/docs/introspection/troubleshooting-with-tags) .

##### Example: Transaction tags (Click to expand)

The following example shows how to set transaction tags with the Spanner JDBC driver.

``` text
BEGIN;
-- Set the transaction tag for the current transaction.
SET TRANSACTION_TAG = 'transaction-tag-1';

-- Set the statement tag that should be included with the next statement.
-- The statement will include both the statement tag and the transaction tag.
SET STATEMENT_TAG = 'select-statement';
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- The statement tag property is cleared after each statement execution.
SHOW VARIABLE STATEMENT_TAG;

-- Set another tag for the next statement.
SET STATEMENT_TAG = 'insert-statement';
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);

COMMIT;

-- The transaction tag property is cleared when the transaction finishes.
SHOW VARIABLE TRANSACTION_TAG;
```

### Transaction statements

The following statements manage and commit Spanner transactions.

#### READ\_TIMESTAMP

``` text
SHOW VARIABLE READ_TIMESTAMP
```

Returns a result set with one row and one column of type `  TIMESTAMP  ` containing the read timestamp of the most recent read-only transaction. This statement returns a timestamp only when either a read-only transaction is still active and has executed at least one query, or immediately after a read-only transaction is committed and before a new transaction starts. Otherwise, the result is `  NULL  ` .

##### Example: Read timestamp (Click to expand)

The following example shows how to view the last read timestamp for a read-only operation with the Spanner JDBC driver.

``` text
-- Execute a query in autocommit mode using the default read-only staleness
-- (strong).
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Show the read timestamp that was used for the previous query.
SHOW VARIABLE READ_TIMESTAMP;

-- Set a non-deterministic read-only staleness and execute the same query.
SET READ_ONLY_STALENESS = 'MAX_STALENESS 20s';

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Show the read timestamp that was used for the previous query. The timestamp
-- is determined by Spanner, and is guaranteed to be no less than
-- 20 seconds stale.
SHOW VARIABLE READ_TIMESTAMP;

-- The read timestamp of a read-only transaction can also be retrieved.
SET READ_ONLY_STALENESS = 'STRONG';
BEGIN;
SET TRANSACTION READ ONLY;

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Show the read timestamp of the current read-only transaction. All queries in
-- this transaction will use this read timestamp.
SHOW VARIABLE READ_TIMESTAMP;

SELECT Title
FROM Albums
ORDER BY Title;

-- The read timestamp is the same as for the previous query, as all queries in
-- the same transaction use the same read timestamp.
SHOW VARIABLE READ_TIMESTAMP;

COMMIT;
```

#### COMMIT\_TIMESTAMP

``` text
SHOW VARIABLE COMMIT_TIMESTAMP
```

Returns a result set with one row and one column of type `  TIMESTAMP  ` containing the commit timestamp of the last read-write transaction that Spanner committed. This statement returns a timestamp only when you execute it after you commit a read-write transaction and before you execute any subsequent `  SELECT  ` , `  DML  ` , or schema change statements. Otherwise, the result is `  NULL  ` .

##### Example: Commit timestamp (Click to expand)

The following example shows how to view the last commit timestamp for a write operation with the Spanner JDBC driver.

``` text
-- Execute a DML statement.
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);

-- Show the timestamp that the statement was committed.
SHOW VARIABLE COMMIT_TIMESTAMP;
```

#### COMMIT\_RESPONSE

``` text
SHOW VARIABLE COMMIT_RESPONSE
```

Returns a result set with one row and two columns:

  - `  COMMIT_TIMESTAMP  ` (type= `  TIMESTAMP  ` ) indicates when the most recent transaction was committed.
  - `  MUTATION_COUNT  ` (type= `  INT64  ` ) indicates how many mutations were applied in the committed transaction. This value is always empty when executed on the emulator.

The mutation count is available only if `  SET RETURN_COMMIT_STATS  ` was set to `  true  ` prior to the transaction commit.

##### Example: Commit response (Click to expand)

The following example shows how to view the last commit response for a write operation with the Spanner JDBC driver.

``` text
-- Enable returning commit stats in addition to the commit timestamp.
SET RETURN_COMMIT_STATS = true;

-- Execute a DML statement.
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1), (2, 200, 2), (3, 300, 3);

-- Show the timestamp that the statement was committed.
SHOW VARIABLE COMMIT_RESPONSE;
```

#### BEGIN \[TRANSACTION\]

``` text
BEGIN [TRANSACTION]
```

Starts a new transaction. The keyword `  TRANSACTION  ` is optional.

  - Use `  COMMIT  ` or `  ROLLBACK  ` to terminate a transaction.
  - If you have enabled [`  AUTOCOMMIT  ` mode](#autocommit) , this statement temporarily takes the connection out of `  AUTOCOMMIT  ` mode. The connection returns to `  AUTOCOMMIT  ` mode when the transaction ends.
  - The transaction mode is determined by the current `  READONLY  ` setting for this connection. This value is set by using the `  SET READONLY = {TRUE | FALSE}  ` command.
  - The transaction mode can be changed by executing `  SET TRANSACTION READ ONLY  ` or `  SET TRANSACTION READ WRITE  ` directly after executing `  BEGIN [TRANSACTION]  ` .

You can execute this statement only while there is no active transaction.

##### Example: BEGIN TRANSACTION (Click to expand)

The following example shows how to start different types of transactions with the Spanner JDBC driver.

``` text
-- This starts a transaction using the current defaults of this connection.
-- The value of READONLY determines whether the transaction is a
-- read-write or a read-only transaction.

BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
COMMIT;

-- Set READONLY to TRUE to use read-only transactions by default.
SET READONLY=TRUE;

-- This starts a read-only transaction.
BEGIN;
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
COMMIT;

-- Execute 'SET TRANSACTION READ WRITE' or 'SET TRANSACTION READ ONLY' directly
-- after the BEGIN statement to override the current default of the connection.
SET READONLY=FALSE;
BEGIN;
SET TRANSACTION READ ONLY;
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
COMMIT;
```

#### COMMIT \[TRANSACTION\]

``` text
COMMIT [TRANSACTION]
```

Commits the current transaction. The keyword `  TRANSACTION  ` is optional.

  - Committing a read-write transaction makes all updates of this transaction visible to other transactions and releases all of the transaction's locks on Spanner.
  - Committing a read-only transaction ends the current read-only transaction. Any subsequent statement starts a new transaction. There is no semantic difference between `  COMMIT  ` and `  ROLLBACK  ` for a read-only transaction.

You can execute this statement only while there is an active transaction.

##### Example: COMMIT TRANSACTION (Click to expand)

The following example shows how to commit a transaction with the Spanner JDBC driver.

``` text
-- Execute a regular read-write transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
COMMIT;

-- Execute a read-only transaction. Read-only transactions also need to be
-- either committed or rolled back in the Spanner JDBC driver in order
-- to mark the end of the transaction.
BEGIN;
SET TRANSACTION READ ONLY;
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
COMMIT;
```

#### ROLLBACK \[TRANSACTION\]

``` text
ROLLBACK [TRANSACTION]
```

Performs a `  ROLLBACK  ` of the current transaction. The keywords `  TRANSACTION  ` is optional.

  - Performing a `  ROLLBACK  ` of a read-write transaction clears any buffered mutations, rolls back the transaction on Spanner, and releases any locks the transaction held.
  - Performing a `  ROLLBACK  ` of a read-only transaction ends the current read-only transaction. Any subsequent statements start a new transaction. There is no semantic difference between `  COMMIT  ` and `  ROLLBACK  ` for a read-only transaction on a connection.

You can execute this statement only while there is an active transaction.

##### Example: ROLLBACK TRANSACTION (Click to expand)

The following example shows how to rollback a transaction with the Spanner JDBC driver.

``` text
-- Use ROLLBACK to undo the effects of a transaction.
BEGIN;
INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);
-- This ensures that the insert statement is not persisted in the database.
ROLLBACK;

-- Read-only transactions also need to be either committed or rolled back in the
-- Spanner JDBC driver in order to mark the end of the transaction.
-- There is no semantic difference between rolling back or committing a
-- read-only transaction.
BEGIN;
SET TRANSACTION READ ONLY;
SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;
ROLLBACK;
```

#### SET TRANSACTION

``` text
SET TRANSACTION { READ ONLY | READ WRITE }
```

Sets the transaction mode for the current transaction.

You can execute this statement only when `  AUTOCOMMIT  ` is `  false  ` , or if you have started a transaction by executing `  BEGIN [TRANSACTION]  ` and have not yet executed any statements in the transaction.

This statement sets the transaction mode for the current transaction only. When the transaction commits or rolls back, the next transaction uses the default mode for the connection (see [`  SET READONLY  `](#readonly) ).

**Note:** You can't set the transaction mode to `  READ WRITE  ` if the connection is in `  READ ONLY  ` mode.

##### Example: SET TRANSACTION (Click to expand)

The following example shows how to set transaction characteristics with the Spanner JDBC driver.

``` text
-- Start a transaction and set the transaction mode to read-only.
BEGIN;
SET TRANSACTION READ ONLY;

SELECT FirstName, LastName
FROM Singers
ORDER BY LastName;

-- Commit the read-only transaction to mark the end of the transaction.
COMMIT;

-- Start a transaction and set the transaction mode to read-write.
BEGIN;
SET TRANSACTION READ WRITE;

INSERT INTO T (id, col_a, col_b)
VALUES (1, 100, 1);

COMMIT;
```

## Batch statements

The following statements manage batches of DDL statements and send those batches to Spanner.

#### START BATCH DDL

``` text
START BATCH DDL
```

Starts a batch of DDL statements on the connection. All subsequent statements during the batch must be DDL statements. The DDL statements are buffered locally and sent to Spanner as one batch when you execute `  RUN BATCH  ` . Executing multiple DDL statements as one batch is typically faster than running the statements separately.

You can execute this statement only while there is no active transaction.

##### Example: DDL batch (Click to expand)

The following example shows how to execute a DDL batch with the Spanner JDBC driver.

``` text
-- Start a DDL batch. All following statements must be DDL statements.
START BATCH DDL;

-- This statement is buffered locally until RUN BATCH is executed.
CREATE TABLE Singers (
  SingerId  INT64 NOT NULL,
  FirstName STRING(MAX),
  LastName  STRING(MAX)
) PRIMARY KEY (SingerId);

-- This statement is buffered locally until RUN BATCH is executed.
CREATE TABLE Albums (
  AlbumId  INT64 NOT NULL,
  Title    STRING(MAX),
  SingerId INT64,
  CONSTRAINT fk_albums_singers FOREIGN KEY (SingerId) REFERENCES Singers (SingerId)
) PRIMARY KEY (AlbumId);

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

#### ABORT BATCH \[TRANSACTION\]

Clears all buffered DDL statements in the current DDL batch and ends the batch.

You can execute this statement only when a DDL batch is active. You can use `  ABORT BATCH  ` regardless of whether or not the batch has buffered DDL statements. All preceding DDL statements in the batch will be aborted.

##### Example: Abort DDL batch (Click to expand)

The following example shows how to abort a DDL batch with the Spanner JDBC driver.

``` text
-- Start a DDL batch. All following statements must be DDL statements.
START BATCH DDL;

-- The following statements are buffered locally.
CREATE TABLE Singers (
  SingerId  INT64 NOT NULL,
  FirstName STRING(MAX),
  LastName  STRING(MAX)
) PRIMARY KEY (SingerId);

CREATE TABLE Albums (
  AlbumId  INT64 NOT NULL,
  Title    STRING(MAX),
  SingerId INT64,
  CONSTRAINT fk_albums_singers FOREIGN KEY (SingerId) REFERENCES Singers (SingerId)
) PRIMARY KEY (AlbumId);

-- This aborts the DDL batch and removes the DDL statements from the buffer.
ABORT BATCH;
```

#### START BATCH DML and RUN Batch

The following statements batch the two DML statements together and send these in one call to the server. A DML batch can be executed as part of a transaction or in autocommit mode.

``` text
START BATCH DML;
INSERT INTO MYTABLE (ID, NAME) VALUES (1, 'ONE');
INSERT INTO MYTABLE (ID, NAME) VALUES (2, 'TWO');
RUN BATCH;
```

##### Example: DML batch (Click to expand)

The following example shows how to execute a DML batch with the Spanner JDBC driver.

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

## Data Boost and partitioned query statements

The [`  partitionQuery  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) API divides a query into smaller pieces, or partitions, and uses multiple machines to fetch the partitions in parallel. Each partition is identified by a partition token. The PartitionQuery API has higher latency than the standard query API, because it is only intended for bulk operations such as exporting or scanning the whole database.

[Data Boost for Spanner](/spanner/docs/databoost/databoost-overview) lets you execute analytics queries and data exports with near-zero impact to existing workloads on the provisioned Spanner instance. Data Boost only supports [partitioned queries](/spanner/docs/reads#read_data_in_parallel) .

You can enable Data Boost with the `  SET DATA_BOOST_ENABLED  ` statement.

The Spanner JDBC driver supports three alternatives for executing partitioned queries:

  - `  SET AUTO_PARTITION_MODE = true  `
  - `  RUN PARTITIONED QUERY sql  `
  - `  PARTITION sql  ` followed by multiple `  RUN PARTITION 'partition-token'  `

Each of these methods are described in the following sections.

#### DATA\_BOOST\_ENABLED

A property of type `  BOOL  ` indicating whether this connection should use [Data Boost](/spanner/docs/databoost/databoost-overview) for partitioned queries. The default is `  false  ` .

``` text
SHOW VARIABLE DATA_BOOST_ENABLED
SET DATA_BOOST_ENABLED = { true | false }
```

##### Example: Execute a query using Data Boost (Click to expand)

The following example shows how to a query using Data Boost with the Spanner JDBC driver.

``` text
-- Enable Data Boost on this connection.
SET DATA_BOOST_ENABLED = true;

-- Execute a partitioned query. Data Boost is only used for partitioned queries.
RUN PARTITIONED QUERY SELECT FirstName, LastName FROM Singers;
```

For a full example, see [DataBoostExample.](https://github.com/GoogleCloudPlatform/java-docs-samples/blob/-/spanner/jdbc/src/main/java/com/example/spanner/jdbc/DataBoostExample.java)

#### AUTO\_PARTITION\_MODE

A property of type `  BOOL  ` indicating whether the connection automatically uses partitioned queries for all queries that are executed.

``` text
SHOW VARIABLE AUTO_PARTITION_MODE
SET AUTO_PARTITION_MODE = { true | false}
```

  - Set this variable to `  true  ` if you want the connection to use partitioned query for all queries that are executed.
  - Also set `  DATA_BOOST_ENABLED  ` to `  true  ` if you want the connection to use [Data Boost](/spanner/docs/databoost/databoost-overview) for all queries.

The default is `  false  ` .

##### Example: Execute (Click to expand)

This example executes two queries with the Spanner JDBC driver using \[Data Boost\](/spanner/docs/databoost/databoost-overview)

``` text
SET AUTO_PARTITION_MODE = true
SET DATA_BOOST_ENABLED = true
SELECT FirstName, LastName FROM Singers
SELECT SingerId, Title FROM Albums
```

For a full example, see [AutoPartitionModeExample.](https://github.com/GoogleCloudPlatform/java-docs-samples/blob/-/spanner/jdbc/src/main/java/com/example/spanner/jdbc/AutoPartitionModeExample.java)

#### RUN PARTITIONED QUERY

``` text
RUN PARTITIONED QUERY <sql>
```

Executes a query as a partitioned query on Spanner. Ensure that `  DATA_BOOST_ENABLED  ` is set to `  true  ` to execute the query with [Data Boost](/spanner/docs/databoost/databoost-overview) :

``` text
SET DATA_BOOST_ENABLED = true
RUN PARTITIONED QUERY SELECT FirstName, LastName FROM Singers
```

The Spanner JDBC driver internally partitions the query and executes partitions in parallel. The results are merged into one result set and returned to the application. The number of worker threads executing partitions can be set with the variable `  MAX_PARTITIONED_PARALLELISM  ` .

For a full example, see [RunPartitionedQueryExample.](https://github.com/GoogleCloudPlatform/java-docs-samples/blob/-/spanner/jdbc/src/main/java/com/example/spanner/jdbc/RunPartitionedQueryExample.java)

#### PARTITION \<SQL\>

``` text
PARTITION <sql>
```

Creates a list of partitions to execute a query against Spanner and returns these a list of partition tokens. Each partition token can be executed on a separate connection on the same or another client using the `  RUN PARTITION 'partition-token'  ` command.

##### Example: Partition query (Click to expand)

The following example shows how to partition a query and then execute each partition separately using the Spanner JDBC driver.

``` text
-- Partition a query. This returns a list of partition tokens that can be
-- executed either on this connection or on any other connection to the same
-- database.
PARTITION SELECT FirstName, LastName FROM Singers;

-- Run the partitions that were returned from the previous statement.
RUN PARTITION 'partition-token-1';
RUN PARTITION 'partition-token-2';
```

For a full example, see [PartitionQueryExample.](https://github.com/GoogleCloudPlatform/java-docs-samples/blob/-/spanner/jdbc/src/main/java/com/example/spanner/jdbc/PartitionQueryExample.java)

#### RUN PARTITION 'partition-token'

``` text
RUN PARTITION 'partition-token'
```

Executes a query partition that has previously been returned by the `  PARTITION  ` command. The command can be executed on any connection that is connected to the same database as the database that created the partition tokens.

#### MAX\_PARTITIONED\_PARALLELISM

A property of type `  INT64  ` indicating the number of worker threads the Spanner JDBC driver uses to execute partitions. This value is used for:

  - `  AUTO_PARTITION_MODE = true  `
  - `  RUN PARTITIONED QUERY sql  `

<!-- end list -->

``` text
SHOW VARIABLE MAX_PARTITIONED_PARALLELISM
SET MAX_PARTITIONED_PARALLELISM = <INT64>
```

Sets the maximum number of worker threads that the Spanner JDBC driver can use to execute partitions. Setting this value to `  0  ` instructs the Spanner JDBC driver to use the number of CPU cores on the client machine as the maximum.

The default is `  0  ` .

## Directed read statements

A property of type `  STRING  ` that sets the directed read option for the following statements.

``` text
SHOW VARIABLE DIRECTED_READ
SET DIRECTED_READ='{"includeReplicas":{"replicaSelections":[{"location":"<location-name>"}]}}'
```

For more information, see [Directed reads](/spanner/docs/directed-reads) .

## Savepoint commands

The following statements enable and disable emulated [savepoints](https://docs.oracle.com/javase/8/docs/api/java/sql/Savepoint.html) in transactions. You can create a savepoint by calling the [`  java.sql.Connection#setSavepoint()  `](https://docs.oracle.com/javase/8/docs/api/java/sql/Connection.html#setSavepoint--) method.

The Spanner JDBC driver emulates savepoints to support frameworks that rely on these for nested transactions. Savepoints are emulated by keeping track of a running checksum for the results that have been returned by statements in the transaction. When rolling back to a savepoint, the Spanner JDBC driver rolls back the transaction, and then retries the transaction up to the point where the savepoint was set. The checksum of the retry is compared to the checksum of the initial transaction to verify that the same results were returned.

#### SAVEPOINT\_SUPPORT

``` text
SHOW VARIABLE SAVEPOINT_SUPPORT
SET SAVEPOINT_SUPPORT = { 'DISABLED' | 'FAIL_AFTER_ROLLBACK' | 'ENABLED' }
```

A property of type `  STRING  ` indicating the current `  SAVEPOINT_SUPPORT  ` configuration. Possible values are:

  - `  DISABLED  ` : All savepoint commands are disabled and will fail.
  - `  FAIL_AFTER_ROLLBACK  ` : Savepoint commands are enabled. Rolling back to a savepoint rolls back the entire transaction. The transaction fails if you try to use the transaction after rolling back to a savepoint.
  - `  ENABLED  ` : All savepoint commands are enabled. Rolling back to a savepoint will rollback the transaction and retry is performed to the savepoint. This operation fails with an `  AbortedDueToConcurrentModificationException  ` error if the underlying data that has been used by the transaction up to the savepoint has changed.

The default value is `  FAIL_AFTER_ROLLBACK  ` .

You can change the value of this variable only while there is no active transaction.

##### Example: Savepoint support (Click to expand)

The following example shows how to enable and disable savepoints support in the Spanner JDBC driver.

``` text
try (Connection connection =
    DriverManager.getConnection(
        String.format(
            "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
            "my-project", "my-instance", "my-database"))) {
  // Savepoints can only be used when AutoCommit=false.
  connection.setAutoCommit(false);

  // Disables setting a savepoint.
  connection.createStatement().execute("SET SAVEPOINT_SUPPORT='DISABLED'");
  // The following statement fails because savepoints have been disabled.
  connection.setSavepoint("my_savepoint1");

  // Enables setting a savepoint and releasing a savepoint.
  // Rolling back to a savepoint is disabled.
  connection.createStatement().execute("SET SAVEPOINT_SUPPORT='FAIL_AFTER_ROLLBACK'");
  Savepoint mySavepoint2 = connection.setSavepoint("my_savepoint2");
  connection.createStatement().execute("insert into my_table (id, value) values (1, 'One')");
  connection.releaseSavepoint(mySavepoint2);
  connection.commit();

  // Enables setting, releasing and rolling back to a savepoint.
  connection.createStatement().execute("SET SAVEPOINT_SUPPORT='ENABLED'");
  Savepoint mySavepoint3 = connection.setSavepoint("my_savepoint3");
  connection.createStatement().execute("insert into my_table (id, value) values (2, 'Two')");
  connection.rollback(mySavepoint3);
}
```

## What's next

Learn how to [connect JDBC to a GoogleSQL-dialect database](/spanner/docs/use-oss-jdbc) .
