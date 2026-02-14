This document describes how to install and run the Spanner migration tool (SMT) on Cloud Shell. For more information about SMT, see [Spanner migration tool](https://googlecloudplatform.github.io/spanner-migration-tool/) .

## Before you begin

1.  Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

2.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

3.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

4.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

5.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

6.  [Install](/sdk/docs/install) the Google Cloud CLI.

7.  To initialize the Google Cloud CLI, run the `  gcloud init  ` command.

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
