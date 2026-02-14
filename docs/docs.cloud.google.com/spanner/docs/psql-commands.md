`  psql  ` is the command-line front end to PostgreSQL. This page describes the `  psql  ` commands that the PostgreSQL interface for Spanner supports. To learn how to connect with `  psql  ` see [Connecting psql to a PostgreSQL-dialect database](/spanner/docs/psql-connect) .

## Meta-commands

The PostgreSQL interface supports the following `  psql  ` meta-commands categories:

**Note: Enter the `  \?  ` command to see the commands under each category.**

  - General
  - Help
  - Query Buffer
  - Input/Output
  - Conditional
  - Informational (some \\d commands only)
  - Formatting
  - Operating System
  - Variables

The following categories are not supported:

  - Connection
  - Large Objects

The following informational commands are supported:

<table>
<thead>
<tr class="header">
<th>Command</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>\d</td>
<td>List tables (excluding system tables)</td>
</tr>
<tr class="even">
<td>\d table</td>
<td>List table columns</td>
</tr>
<tr class="odd">
<td>\dt</td>
<td>List tables in all schemas (detailed)</td>
</tr>
<tr class="even">
<td>\dt table</td>
<td>List table (detailed)</td>
</tr>
<tr class="odd">
<td>\dn</td>
<td>List schemas</td>
</tr>
</tbody>
</table>

## Session management statements

`  psql  ` communicates with Spanner through [PGAdapter](/spanner/docs/pgadapter) , which uses the core engine of the Spanner JDBC driver. The driver supports the session management statements described in [Session management statements](https://cloud.google.com/spanner/docs/jdbc-session-mgmt-commands-pgcompat) . Therefore, you can use these statements with `  psql  ` .

## SQL statement batching

`  psql  ` and PGAdapter support multi-statement SQL batches. To batch statements, you use the `  psql -c  ` option. This option allows for one or more SQL or session management statements, separated by semicolons (;), to be passed in as a single execution request. A batch can include any supported statements, and can mix DDL, DML, and DQL.

A multi-statement batch is executed within a single implicit transaction block. Implicit transaction blocks are automatically closed at the end of the batch. If any errors occur inside of an implicit transaction block, the entire transaction is rolled back.

Explicit `  BEGIN  ` and `  COMMIT  ` transaction controls are supported, but an explicit transaction block can't contain DDL statements.

### Examples

#### DML

The following example shows how to submit a batch of `  INSERT  ` statements.

``` text
psql -h localhost -p 5432 -c "INSERT INTO users (id, age, firstname) VALUES (1, 25, 'Alex'); \
 INSERT INTO users (id, age, firstname) VALUES (2, 31, 'Dana'); \
 INSERT INTO users (id, age, firstname) VALUES (3, 54, 'Izumi');"
```

The next example shows how to execute the SQL statements in the file `  insert_contacts.sql  ` .

``` text
psql -h localhost -c "$(cat contacts_insert.sql)"
```

#### DDL

This example submits a batch of `  ALTER TABLE  ` statements.

``` text
psql -h localhost -p 5432 test-db -c "ALTER TABLE users ADD col1 integer; \
 ALTER TABLE users ADD col2 text; ALTER TABLE users ADD col3 float8;"
```

## COPY command for importing data

Use the `  COPY FROM STDIN  ` command to import data from a text or CSV file into a PostgreSQL-dialect database. Although only STDIN is supported, you can import using `  COPY  ` by piping files into `  psql  ` .

There are two ways to execute the `  COPY  ` command:

  - Atomic `  COPY  `
    
    Data is copied in a single transaction. This is the default. Standard transaction limits of Spanner apply to the transaction. This means that at most 80,000 mutations or 100MB of data can be included in one `  COPY  ` operation.

  - Non-atomic `  COPY  `
    
    `  COPY  ` automatically splits the data over multiple transactions if the file contains more than 80,000 mutations or more than 100MB.
    
    If an error is encountered during the `  COPY  ` and the operation aborts, some rows might already be persisted to the database. No rollback occurs. The transactions are executed in parallel, so data after the row in the import file that caused the error might be imported to the database before the `  COPY  ` operation is halted.

### Enable non-atomic `     COPY    `

To enable non-atomic `  COPY  ` , submit the following command before executing the copy operation.

`  SET SPANNER.AUTOCOMMIT_DML_MODE='PARTITIONED_NON_ATOMIC'  `

### Syntax

``` text
COPY table_name [ ( column_name [, ...] ) ]
    FROM STDIN
    [ [ WITH ] ( option [, ...] ) ]

where option is one of:

    FORMAT format_name
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    HEADER [boolean]

and format_name is:
    {text|csv}

and delimiter_character is:
    [!-~] except ' " \

and null_string is:
    {a—z|A—Z|0—9|_}+

and quote_character is:
    [!-~] except ' " \

and escape_character is:
    [!-~] except ' " \

and boolean is:
    {TRUE|ON|1|FALSE|OFF|0}
```

The table must already exist. If no column list is specified, all columns of the table are copied.

The default for `  FORMAT  ` is `  text  ` .

***delimiter\_character*** must be a one-byte character. The default is the Tab character for text format and a comma for CSV format.

`  NULL  ` specifies the string that represents a null value. The default is \\N (backslash+N) in text format, and an unquoted empty string in CSV format. You might prefer an empty string even in text format for cases where you don't want to distinguish nulls from empty strings.

`  QUOTE  ` specifies the quoting character to use when a data value is quoted. The default is double quote. This must be a single one-byte character. This option is allowed only when using the CSV format.

`  ESCAPE  ` specifies the character to appear before a data character that matches the `  QUOTE  ` value. The default is the same as the `  QUOTE  ` value (so that the quoting character is doubled if it appears in the data). This must be a single one-byte character. This option is allowed only when using the CSV format.

`  HEADER  ` indicates if the first record of the input file is a header (contains column names). The default is TRUE.

### Examples

This example imports data from the text-formatted file named `  mydata.txt  ` into table `  mytable  ` . PGAdapter must be running. For more information, see [Starting PGAdapter](/spanner/docs/pgadapter-start) .

``` text
cat mydata.txt | psql -h localhost -c "COPY mytable FROM STDIN;"
```

In this next example, `  mydata.csv  ` is in CSV format and its first row is a header with comma-separated column names.

``` text
cat mydata.csv | psql -h localhost \
  -c "COPY mytable FROM STDIN WITH (FORMAT csv, ESCAPE '~', HEADER TRUE);"
```

This next example shows how to start a non-atomic `  COPY  ` operation.

``` text
cat mydata.txt | psql -h localhost \ 
  -c "SET SPANNER.AUTOCOMMIT_DML_MODE='PARTITIONED_NON_ATOMIC'" -c "COPY mytable FROM STDIN;"
```

### Troubleshooting

The following are some common errors.

#### Invalid input syntax

The following error occurs:

`  Invalid input syntax for type <type>:"<table_name>"  `

This error can occur when the input file has a header row with column names and the `  HEADER  ` option was not specified.

#### Invalid COPY data

The following error occurs:

`  Invalid COPY data: Row length mismatched. Expected <number> columns, but only found <number>  `

This error occurs when a row in the input file doesn't include a value (or null) for every column in the table. One cause could be a malformed CSV file, or a mismatch between the specified delimiter option (or default delimiter) and the actual delimiter in the file.

## What's next

  - Learn how to [connect to a PostgreSQL-dialect database with `  psql  `](/spanner/docs/psql-connect) .
  - Learn about [PGAdapter](/spanner/docs/pgadapter) .
