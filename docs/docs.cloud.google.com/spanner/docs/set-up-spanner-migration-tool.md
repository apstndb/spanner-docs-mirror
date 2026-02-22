This document describes how to install and run the Spanner migration tool (SMT) on Cloud Shell. For more information about SMT, see [Spanner migration tool](https://googlecloudplatform.github.io/spanner-migration-tool/) .

## Before you begin

1.  [Install](/sdk/docs/install) the Google Cloud CLI.
2.  To initialize the Google Cloud CLI, run the `  gcloud init  ` command.

## Install Spanner migration tool

You can install SMT using a Linux shell command or Google Cloud CLI:

### Linux shell

To install SMT, run the following command:

``` text
  sudo apt-get install google-cloud-sdk-spanner-migration-tool
```

### gcloud

You can install SMT by using the [`  gcloud components install  `](/sdk/gcloud/reference/components/install) command:

``` text
  gcloud components install spanner-migration-tool
```

For more information on installing SMT, see [Installing Spanner migration tool](https://googlecloudplatform.github.io/spanner-migration-tool/install.html#installing-spanner-migration-tool) .

## Access Spanner migration tool

You can use the [`  gcloud alpha spanner migrate  `](/sdk/gcloud/reference/alpha/spanner/migrate) command to access and use SMT.

To launch the SMT web UI, you can run the following command:

``` text
  gcloud alpha spanner migrate web
```

You need to provide your Google Cloud credentials to allow SMT to access resources. Click `  http://localhost:8080  ` on the response to the previous command to open the web UI.

**Note:** Keep the Google Cloud CLI tab open while you view the SMT web UI.

## Connect to Spanner

To connect to Spanner using the SMT web UI, do the following:

1.  Click the edit button to configure the connection to Spanner.

2.  Specify the following information to connect to Spanner:
    
      - **Project ID** : the project ID where your Spanner instance is.
      - **Instance ID** : the Spanner instance ID.

3.  Click **Save** .

## Connect to your source database

You need to connect to the source database using the SMT web UI by providing the following information:

  - **Database Engine** : specify whether your source database is MySQL, SQL Server, Oracle, or PostgreSQL.
  - **Hostname** : the IP address of your source database.
  - **Port** : the port where your source database is accessible.
  - **User name** : the username of the source database.
  - **Password** : the password of the source database.
  - **Spanner Dialect** : specify whether you want to use [GoogleSQL](/spanner/docs/reference/standard-sql/overview) or [PostgreSQL](/spanner/docs/reference/postgresql/overview) .

After entering the information you can connect to the source database by clicking **Test Connection** , and then clicking **Connect** .

## What's next?

  - [Migrate schema from MySQL](/spanner/docs/migrate-mysql-schema) .
  - [Use Spanner migration tool for MySQL schema migration](/spanner/docs/migrate-mysql-schema) .
