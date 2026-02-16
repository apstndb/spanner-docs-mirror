This page describes how to manage Spanner database schema changes with [Liquibase](https://www.liquibase.org/) for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

Liquibase is an open-source database-independent library for tracking, managing, and applying database schema changes. It supports SQL as well as declarative formats such as XML, YAML, and JSON.

Liquibase can target Spanner databases. It supports all Spanner features, with some limitations.

  - To see general limitations, see [limitations](https://github.com/cloudspannerecosystem/liquibase-spanner/blob/master/limitations.md) .
  - To see additional information for PostgreSQL-dialect databases, such as Liquibase requirements, supported change types, and limitations, see [PGAdapter and Liquibase](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/liquibase) .

## Install Liquibase

To use Liquibase with GoogleSQL-dialect databases, you have to install the Spanner Liquibase extension. For PostgreSQL-dialect databases, Liquibase can use its built-in PostgreSQL support in conjunction with [PGAdapter](/spanner/docs/pgadapter) .

### GoogleSQL

1.  Follow the instructions in the [Liquibase documentation](https://www.liquibase.org/get-started/quickstart) to install and configure Liquibase, and to take a snapshot of your database.

2.  Navigate to the Spanner Liquibase Extension releases page on GitHub and select the [latest release](https://github.com/cloudspannerecosystem/liquibase-spanner/releases/latest) .

3.  Select and download the JAR file with the name `  liquibase-spanner-x.y.z-all.jar  ` , where x.y.z represents the extension version number. For example, `  liquibase-spanner-4.17.0-all.jar  ` .

4.  Place the downloaded JAR file in the Liquibase lib directory. The JAR file includes the extension, the Spanner SDK, and the Spanner JDBC driver driver.

In the `  liquibase.properties  ` configuration file, set the `  url  ` property as follows.

``` text
 jdbc:cloudspanner:/projects/PROJECT/instances/INSTANCE/databases/DATABASE
 
```

Your `  liquibase.properties  ` configuration file can contain only this property. Other properties are optional.

### PostgreSQL

1.  Ensure that PGAdapter is started and running on the machine where you install Liquibase. For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

2.  Follow the instructions in the [Liquibase documentation](https://www.liquibase.org/get-started/quickstart) to install and configure Liquibase, and to take a snapshot of your database.

In the `  liquibase.properties  ` configuration file, set the `  url  ` property as follows.

``` text
  jdbc:postgresql://localhost:5432/DATABASE_NAME?options=-c%20spanner.ddl_transaction_mode=AutocommitExplicitTransaction
  
```

Your `  liquibase.properties  ` configuration file can contain only this property. Other properties are optional.

The `  url  ` string must include `  options=-c%20spanner.ddl_transaction_mode=AutocommitExplicitTransaction  ` because Spanner doesn't support DDL transactions, and this ensures that DDL transactions are converted to DDL batches. For more information, see [DDL Options for PGAdapter](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/ddl.md) .

## Review the Liquibase samples

### GoogleSQL

The sample change log file [changelog.yaml](https://github.com/cloudspannerecosystem/liquibase-spanner/blob/master/example/changelog.yaml) included with the GoogleSQL Liquibase extension demonstrates many of the features of Liquibase and how to use them with Spanner.

### PostgreSQL

The sample change log file `  dbchangelog.xml  ` available in the [PGAdapter and Liquibase GitHub repository](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/liquibase) demonstrates many of the features of Liquibase and how to use them with Spanner.

## Liquibase quickstart

This quickstart shows you how to use Liquibase to add a `  Singers  ` table to a database.

### Before you begin

  - Make sure that you have completed the preceding steps to [install](#install-liq) Liquibase.

  - Create a Spanner instance.

  - Create a GoogleSQL-dialect database or PostgreSQL-dialect database.

  - For PostgreSQL-dialect databases only, ensure that PGAdapter is started and running on the same machine as your Liquibase installation. For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

  - For PostgreSQL-dialect databases only, use the [create\_database\_change\_log.sql](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/liquibase/create_database_change_log.sql) script to create the `  databasechangeloglock  ` and `  databasechangelog  ` metadata tables. You must create these tables to override the tables that Liquibase creates automatically in your database. This is to ensure that the correct PostgreSQL data types for Spanner are used in these tables.
    
    You can run the script with the following command:
    
    ``` text
    psql -h localhost -d DATABASE_NAME -f create_database_change_log.sql
    ```

  - Give the Spanner Liquibase extension temporary use of your Spanner user credentials for API access by running the following `  gcloud  ` command:
    
    ``` text
    gcloud auth application-default login
    ```

### Create a changelog.yaml

1.  Enter the following YAML into your favorite editor.
    
    ``` text
    databaseChangeLog:
      - preConditions:
         onFail: HALT
         onError: HALT
    
      - changeSet:
         id: create-singers-table
         author: spanner-examples
         changes:
           - createTable:
              tableName: Singers
              columns:
                -  column:
                    name:    SingerId
                    type:    BIGINT
                    constraints:
                      primaryKey: true
                      primaryKeyName: pk_Singers
                -  column:
                    name:    Name
                    type:    VARCHAR(255)
    ```
    
    This YAML defines a table called `  Singers  ` with a primary key `  SingerId  ` and a column called `  Name  ` to store the singer's name.
    
    For PostgreSQL-dialect databases, we recommend using all lower case for table and column names. For more information, see [PostgreSQL case sensitivity](/spanner/docs/reference/postgresql/lexical#case-sensitivity) .
    
    Note that the `  createTable  ` change set must include a primary key constraint, and the name of the primary key constraint must be pk\_ table\_name .

2.  Save your changes as `  changelog.yaml  ` .

### Run Liquibase

Apply the changeset in `  changelog.yaml  ` by executing the following command:

``` text
liquibase --changeLogFile changelog.yaml update
```

Liquibase uses the URL that you defined in the `  liquibase.properties  ` file. You can override the value in the file by adding the following argument to the preceding command:

``` text
--url URL
```

### Verify your changes

The updates in the preceding step caused the `  Singer  ` table to be added to your database. Also, the `  DATABASECHANGELOG  ` and `  DATABASECHANGELOGLOCK  ` tables were added (GoogleSQL-dialect database) or updated (PostgreSQL-dialect database).

You can verify the existence of these tables through the Google Cloud console or gcloud CLI. For example, running the SQL query `  SELECT * FROM INFORMATION_SCHEMA.TABLES  ` returns a list of all tables in your database.

``` text
gcloud spanner databases execute-sql DATABASE_NAME --instance=INSTANCE \
    --sql='SELECT * FROM INFORMATION_SCHEMA.TABLES'
```

You can see a record of the changes that were applied by querying the contents of `  DATABASECHANGELOG  ` .

## What's next

  - For more documentation, visit the [Spanner Liquibase Extension](https://github.com/cloudspannerecosystem/liquibase-spanner) GitHub repository.

  - To learn more about Liquibase, see [Getting Started with Liquibase](https://www.liquibase.org/get-started) .
