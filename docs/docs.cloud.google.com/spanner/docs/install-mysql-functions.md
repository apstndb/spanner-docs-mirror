This document describes how to install a predefined library of MySQL functions in a Spanner database. You can add these functions to your database using the Google Cloud console or the Google Cloud CLI.

Installing these MySQL functions extends Spanner's capabilities, allowing you to perform operations that are common in MySQL environments directly within Spanner.

For more information about the MySQL functions that Spanner supports, see [MySQL functions](https://docs.cloud.google.com/spanner/docs/reference/mysql/user_defined_functions_all) .

## Required roles

To get the permissions that you need to install the MySQL functions, ask your administrator to grant you the [Cloud Spanner Database Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `roles/spanner.databaseAdmin` ) IAM role on your project. For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## Install the MySQL user-defined functions

You can install the MySQL user-defined functions in a Spanner database in the following ways:

### Google Cloud console

1.  In the Google Cloud console, go to the Spanner **Instances** page.

2.  Click the instance containing the database.

3.  Click the database.

4.  Click the **Write DDL** button to open Spanner Studio.

5.  Copy the entire content of the DDL file in the [`mysql_udfs.sql`](https://github.com/googleapis/spanner-sql-udf/blob/main/mysql/mysql_udfs.sql) file.

6.  In a SQL editor tab, paste the copied content from the DDL file,

7.  Click **Run** .

### gcloud

You can use the [`gcloud spanner database ddl update`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/databases/ddl/update) command to install the MySQL UDFs.

Before using any of the command data below, make the following replacements:

  - DATABASE\_ID : the ID of the database to add the MySQL UDFs.
  - INSTANCE\_ID : the ID of the instance where the database is located.
  - DDL\_FILE\_PATH : path to a file containing all the `CREATE OR REPLACE FUNCTION` statements in the [mysql\_udfs.sql](https://github.com/googleapis/spanner-sql-udf/blob/main/mysql/mysql_udfs.sql) file.

Execute the following command:

#### Linux, macOS, or Cloud Shell

> **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](https://docs.cloud.google.com/sdk/gcloud/reference/init) ; or [gcloud auth login](https://docs.cloud.google.com/sdk/gcloud/reference/auth/login) and [gcloud config set project](https://docs.cloud.google.com/sdk/gcloud/reference/config/set) .

    gcloud spanner databases ddl update DATABASE_ID \
    --instance=INSTANCE_ID \
    --ddl-file=DDL_FILE_PATH

#### Windows (PowerShell)

> **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](https://docs.cloud.google.com/sdk/gcloud/reference/init) ; or [gcloud auth login](https://docs.cloud.google.com/sdk/gcloud/reference/auth/login) and [gcloud config set project](https://docs.cloud.google.com/sdk/gcloud/reference/config/set) .

    gcloud spanner databases ddl update DATABASE_ID `
    --instance=INSTANCE_ID `
    --ddl-file=DDL_FILE_PATH

#### Windows (cmd.exe)

> **Note:** Ensure you have initialized the Google Cloud CLI with authentication and a project by running either [gcloud init](https://docs.cloud.google.com/sdk/gcloud/reference/init) ; or [gcloud auth login](https://docs.cloud.google.com/sdk/gcloud/reference/auth/login) and [gcloud config set project](https://docs.cloud.google.com/sdk/gcloud/reference/config/set) .

    gcloud spanner databases ddl update DATABASE_ID ^
    --instance=INSTANCE_ID ^
    --ddl-file=DDL_FILE_PATH

## What's next

  - Learn about all the Spanner supported [MySQL user-defined functions](https://docs.cloud.google.com/spanner/docs/reference/mysql/user_defined_functions_all) .
