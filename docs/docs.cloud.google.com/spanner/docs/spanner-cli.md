**PostgreSQL interface note:** For PostgreSQL-dialect databases, you can use the [psql command-line tool](/spanner/docs/psql-commands) . The examples in this document are intended for GoogleSQL-dialect databases.

The Spanner CLI is a command-line interface (CLI) within the gcloud CLI that lets you connect to and interact with your Spanner databases. For example, you can use the Spanner CLI to run [GoogleSQL](/spanner/docs/reference/standard-sql/overview) statements and automate tasks. This document describes how to set up and use the Spanner CLI.

The Spanner CLI is based on the open source [spanner-cli](https://github.com/cloudspannerecosystem/spanner-cli) project.

For a list of all supported Spanner CLI commands, see [`  gcloud spanner cli  ` commands](/sdk/gcloud/reference/spanner/cli) .

## Key benefits

You can use the Spanner CLI to perform the following actions:

  - Run DDL, DML, and DQL SQL commands.
  - Write and execute SQL statements across multiple lines.
  - Use [meta-commands](#supported-meta-commands) to help with system tasks such as executing a system shell command and executing SQL from a file.
  - Automate SQL executions by [writing a series of SQL statements into a script file](#use-file-based) , and then instructing Spanner CLI to execute the script. In addition, you can redirect the output to an output file.
  - [Start an interactive Spanner CLI session](#start-interactive-session) , which lets you directly type SQL statements and meta-commands and see results in the CLI.

## Before you begin

Before using the Spanner CLI, ensure that you have the required role and have installed the CLI.

### Required roles

To get the permissions that you need to install the Spanner, ask your administrator to grant you the [Cloud Spanner Admin](/iam/docs/roles-permissions/spanner#spanner.admin) ( `  roles/spanner.admin  ` ) IAM role on Spanner. For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

### Install the Spanner CLI

The Spanner CLI is available in the gcloud CLI. When you run the `  gcloud spanner cli  ` command for the first time, gcloud CLI automatically installs the Spanner CLI component.

To install the Spanner CLI manually, run the following command:

``` text
gcloud components install spanner-cli
```

**Note:** If you use the open source spanner-cli, you must manually install the Spanner CLI because the open source version isn't automatically upgraded.

If installation using the Google Cloud CLI command is unsuccessful or not feasible in your shell environment, Spanner provides standalone Debian (.deb) and RPM (.rpm) packages. You can use these packages to manually install on compatible systems. To install, run the following command:

``` text
apt-get install google-cloud-cli-spanner-cli
```

### Configuration options

The Spanner CLI supports the following configurable options:

  - The project option is retrieved through the `  core/project  ` property; alternatively, you can specify the project using the `  --project  ` option.
  - The instance option is retrieved through the `  core/instance  ` property; alternatively, you can specify the instance using the `  --instance  ` option.
  - The API endpoint is retrieved through the `  api_endpoint_overrides/spanner  ` property; alternatively, you can specify the endpoint using the `  --host  ` and `  --port  ` options. The default Spanner endpoint is used if no endpoint is specified.

## Use the Spanner CLI

1.  [Set up a Google Cloud project](/spanner/docs/getting-started/set-up#set_up_a_project) .

2.  [Set up authentication using the gcloud CLI](/spanner/docs/getting-started/set-up#gcloud) .

3.  [Create an instance](/spanner/docs/getting-started/gcloud#create-instance) .

4.  [Create a database](/spanner/docs/getting-started/gcloud#create-database) .

5.  Run the following command to start the Spanner CLI and interact with your Spanner database:
    
    ``` text
    gcloud spanner cli DATABASE_ID --instance=INSTANCE_ID
    ```
    
    Replace the following:
    
      - `  DATABASE_ID  ` : the ID of the Spanner database. This is the name you used in the previous Create a database step. You can use the [`  gcloud spanner databases list  `](/sdk/gcloud/reference/spanner/databases/list) command to list the Spanner databases contained within the given instance.
      - `  INSTANCE_ID  ` : the ID of the Spanner instance. This is the name you used in the previous Create an instance step. You can use the [`  gcloud spanner instances list  `](/sdk/gcloud/reference/spanner/instances/list) command to list the Spanner instances contained within the given project.

## Execute SQL

You can execute SQL statements in the Spanner CLI by using the [`  execute  ` option](#use-execute-option) or using a [file-based input and output method](#use-file-based) . Your SQL statements can consist of DDL, DML, or DQL.

### Use the `     execute    ` flag

To use the `  execute  ` flag to execute SQL, run the following [`  gcloud spanner cli  `](/sdk/gcloud/reference/spanner/cli) command:

``` text
gcloud spanner cli DATABASE_ID --instance INSTANCE_ID \
    --execute "SQL"
```

Replace the following:

  - `  DATABASE_ID  ` : the ID of the Spanner database that you want to connect to.
  - `  INSTANCE_ID  ` : the ID of the Spanner instance that you want to connect to.
  - `  SQL  ` : the SQL that you want to execute.

For example, to execute a DDL statement:

``` text
gcloud spanner cli test-database --instance test-instance \
    --execute "CREATE TABLE Singers ( \
        SingerId   INT64 NOT NULL, \
        FirstName  STRING(1024), \
        LastName   STRING(1024), \
        SingerInfo STRING(1024), \
        BirthDate  DATE \
      ) PRIMARY KEY(SingerId);"
```

To execute a DML statement:

``` text
gcloud spanner cli test-database --instance test-instance \
    --execute "INSERT INTO Singers (SingerId, FirstName, LastName, SingerInfo) \
        VALUES(1, 'Marc', 'Richards', 'nationality: USA'), \
              (2, 'Catalina', 'Smith', 'nationality: Brazil'), \
              (3, 'Andrew', 'Duneskipper', NULL);"
```

### Use a file-based input and output

If you use the file-based input and output method, Spanner reads its input from a file and writes its output to another file. To use the file-based input and output method to execute SQL, run the following command:

``` text
gcloud spanner cli DATABASE_ID --instance INSTANCE_ID \
    --source INPUT_FILE_PATH --tee OUTPUT_FILE_PATH
```

You can also use the file-based redirection input and output method:

``` text
gcloud spanner cli DATABASE_ID --instance INSTANCE_ID \
    < INPUT_FILE_PATH > OUTPUT_FILE_PATH
```

Replace the following:

  - `  DATABASE_ID  ` : the ID of the Spanner database that you want to connect to.
  - `  INSTANCE_ID  ` : the ID of the Spanner instance that you want to connect to.
  - `  SOURCE_FILE_PATH  ` : the file that contains the SQL that you want to execute.
  - `  OUTPUT_FILE_PATH  ` : the named file to append a copy of the SQL output.

## Start an interactive session

You can start an interactive Spanner CLI session, which lets you directly type SQL statements and meta-commands and see results in the CLI. To do so, run the following command:

``` text
gcloud spanner cli DATABASE_ID --instance=INSTANCE_ID
```

Upon successful connection between the CLI and your database, you will see a prompt (for example, `  spanner-cli>  ` ) where you can do the following:

  - Directly type GoogleSQL statements:
      - [GoogleSQL DDL reference](/spanner/docs/reference/standard-sql/data-definition-language)
      - [GoogleSQL DML reference](/spanner/docs/reference/standard-sql/dml-syntax)
      - [GoogleSQL query syntax](/spanner/docs/reference/standard-sql/query-syntax)
  - Execute transactions
  - Use supported [meta-commands](#supported-meta-commands)

After pressing the `  ENTER  ` key, the statement or command is sent to the appropriate Spanner database. Spanner then executes the statement or command.

In the following example, you start an interactive session in `  test-database  ` and then execute `  SELECT 1;  ` :

``` text
gcloud spanner cli test-database --instance test-instance

Welcome to Spanner-Cli Client.
Type 'help;' or '\h' for help.
Type 'exit;' or 'quit;' or '\q' to exit.

spanner-cli> SELECT 1;
+---+
|   |
+---+
| 1 |
+---+

1 rows in set (1.11 msecs)
```

### Execute DDL statement

To execute a DDL statement, you can run the following:

``` text
spanner-cli> CREATE TABLE Singers (
          ->         SingerId   INT64 NOT NULL,
          ->         FirstName  STRING(1024),
          ->         LastName   STRING(1024),
          ->         SingerInfo STRING(1024),
          ->         BirthDate  DATE
          -> ) PRIMARY KEY(SingerId);

Query OK, 0 rows affected (17.08 sec)
```

### Execute DML statement

To execute a DML statement, you can run the following:

``` text
spanner-cli> INSERT INTO Singers (SingerId, FirstName, LastName, SingerInfo)
          -> VALUES(1, 'Marc', 'Richards', 'nationality: USA'),
          -> (2, 'Catalina', 'Smith', 'nationality: Brazil'),
          -> (3, 'Andrew', 'Duneskipper', NULL);

Query OK, 3 rows affected (0.32 sec)
```

### Execute partitioned DML statement

In the Spanner CLI, you can use the `  PARTITIONED  ` keyword with the `  UPDATE  ` and `  DELETE  ` commands to run efficient, large-scale [partitioned DML](/spanner/docs/dml-partitioned) statements. When the Spanner CLI encounters `  PARTITIONED UPDATE  ` or `  PARTITIONED DELETE  ` , it recognizes them as partitioned DML operations. These keywords are useful for operations that affect a significant portion of a table without locking the entire table for an extended period. Spanner doesn't apply the partitioned DML statements atomically across the entire table. It applies partitioned DML statements atomically across each partition.

To execute a partitioned DML statement, you can run the following:

``` text
-- Update all rows in the 'Products' table by multiplying the price by 2
spanner-cli> PARTITIONED UPDATE Products SET Price = Price * 2 WHERE Price > 100;

-- Delete all rows in the 'Products' table with price less than 500
spanner-cli> PARTITIONED DELETE FROM Products WHERE Price < 500;
```

## Supported meta-commands

The Spanner CLI supports utility meta-commands, which are commands that operate on the client, in this case the Spanner CLI. The following meta-commands are supported in the Spanner CLI:

**Command**

**Syntax**

**Description**

?

`  \?  `

Displays help information. Same as `  \h  ` .

Delimiter

`  \d  `

Sets the statement delimiter. The default delimiter is a semi-colon.

Exit

`  \q  `

Exits the Spanner CLI. Same as quit.

Go

`  \g  `

Sends and runs SQL statement in Spanner.

Help

`  \h  `

Displays help information. Same as `  \?  ` .

Notee

`  \t  `

Turns off writing to the output file set by the `  \T  ` .

Prompt

`  \R  `

Changes your prompt to a user prompt string.

Quit

`  \q  `

Quits Spanner CLI. Same as exit.

Source

`  \.  `

Executes SQL from an input file. Takes \[filename\] as an argument.

System

`  \!  `

Executes a system shell command.

Tee

`  \T  `

Appends command output to a specified \[filename\] along with the standard output.

Use

`  \u  `

Connects to another database. Takes the new database name as an argument.

## Additional supported commands

The Spanner CLI supports additional commands. For more information, see [Spanner CLI commands](/spanner/docs/spanner-cli-commands) .

## Get support

To report an issue with the Spanner CLI, create a new [issue](https://issuetracker.google.com/issues/new?component=190851&template=844402) .

## What's next

  - See a list of all supported [Spanner CLI commands](/spanner/docs/spanner-cli-commands) .
  - See a list of all supported [`  gcloud spanner cli  ` commands](/sdk/gcloud/reference/spanner/cli) .
