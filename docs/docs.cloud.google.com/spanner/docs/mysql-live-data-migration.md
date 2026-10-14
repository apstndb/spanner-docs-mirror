---
name: documents/docs.cloud.google.com/spanner/docs/mysql-live-data-migration
uri: https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration
title: Migrate your MySQL data
description: Perform a live migration from MySQL to Spanner with minimal downtime using Datastream and Dataflow.
data_source: docs.cloud.google.com
---

This document explains how to perform a live data migration from a source MySQL instance to Spanner with minimal downtime using Terraform to deploy Dataflow and Datastream.

Once you perform the live data migration and are confident that all your data has been transferred, you've migrated your code and dependencies, and completed testing, you can switch your application to using Spanner instead of your source MySQL database.

You can perform a live data migration after creating your target Spanner database. You need to create a compatible schema on your target database before migrating data.

## How it works

The live data migration consists of the following two phases:

- **Backfill migration** :

  - During backfill migration, Dataflow reads existing data from your source MySQL database and migrates the data to the target Spanner database. You need to use a bulk migration Dataflow template to move the data from your source MySQL instance to Spanner.
  - When the backfill migration fails to write a row to Spanner, it writes that row to a dead-letter queue (DLQ) directory in a Cloud Storage bucket. You can have the backfill migration retry writing these rows to Spanner.

- **Change data capture (CDC) migration** :

  - This phase runs concurrently with the backfill migration, capturing changes occurring in the source MySQL instance in real time. These changes are then applied to Spanner after the backfill migration is complete.
  - You need to use Datastream to capture changes occurring in your source MySQL instance in real time and write them to a Cloud Storage bucket.
  - After the backfill migration is complete, you need to use Dataflow to move the CDC from the Cloud Storage bucket to Spanner. If Dataflow fails to write a row to Spanner for any reason, it writes that row to a DLQ directory in a different Cloud Storage bucket. The CDC migration automatically retries writing the rows from the DLQ directory to Spanner.

## Plan for the live data migration

You need to configure the network infrastructure required for data to flow between your source MySQL instance, Datastream, Dataflow , Cloud Storage buckets, and the target Spanner database. We recommend configuring private network connectivity for a secure migration. Depending on your organization's compliance requirements, you might have to configure public or private network connectivity. For more information about Datastream connectivity, see [Network connectivity options](https://docs.cloud.google.com/datastream/docs/network-connectivity-options) .

To plan for the live data migration, you might need your organization's network administrator to perform the following tasks:

- Use the default VPC or create a new VPC in your project with the following requirements:
  - The source MySQL instance must be available on this VPC. You might need to create an egress firewall rule on this VPC, and an ingress firewall rule on the VPC where the source MySQL instance is located.
  - Datastream, Dataflow, Cloud Storage buckets, and the target Spanner database must be available on this VPC.
  - You must create an allowlist on your source MySQL instance to allow connections from the VPC.
- Determine and allocate an IP address range in the VPC that Datastream can use.
- Create a subnetwork in the VPC for Dataflow to use to complete the backfill migration.
- Create a subnetwork in the VPC for Dataflow to use to complete the CDC migration later.

To perform a live data migration, follow these steps:

1.  [Set up CDC migration](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#setup-cdc-migration) .
2.  [Perform the backfill migration](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#perform-backfill-migration) .
3.  [Perform the CDC migration after the backfill migration is finished](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#perform-cdc-migration) .
4.  [Monitor and resolve CDC errors](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#resolve-cdc-dlq-errors) .
5.  [Finish the CDC migration](https://docs.cloud.google.com/spanner/docs/mysql-live-data-migration#finish-cdc-migration) .

Performing the live data migration requires deploying and managing a significant number of resources. Spanner provides two sample Terraform templates for each phase of the live data migration.

The live migration template performs the CDC migration in two phases:

- Set up CDC migration to a Cloud Storage bucket using Datastream. You can use a Terraform variable to prevent the template from creating the Dataflow jobs.
- Migrate the CDC to Spanner from the Cloud Storage bucket using Dataflow. You must perform this phase only after the backfill migration Terraform template is finished with the backfill migration.

The backfill migration terraform template performs the backfill migration from your source MySQL instance to Spanner.

## Before you begin

- Ensure Terraform is installed on your local shell.

- Create a service account to run the live data migration. For more information about creating a service account, see [Create service accounts](https://docs.cloud.google.com/iam/docs/service-accounts-create) .

- To ensure that the service account has the necessary permissions to perform live migration, ask your administrator to grant the following IAM roles to the service account on your project:

  > **Important:** You must grant these roles to the service account, *not* to your user account. Failure to grant the roles to the correct principal might result in permission errors.

  - [Dataflow Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.admin) ( `roles/dataflow.admin` )
  - [Datastream Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/datastream#datastream.admin) ( `roles/datastream.admin` )
  - [Security Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/iam#iam.securityAdmin) ( `roles/iam.securityAdmin` )
  - [Service Account Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/iam#iam.serviceAccountAdmin) ( `roles/iam.serviceAccountAdmin` )
  - [Pub/Sub Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/pubsub#pubsub.admin) ( `roles/pubsub.admin` )
  - [Storage Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.admin) ( `roles/storage.admin` )
  - [Compute Network Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/compute#compute.networkAdmin) ( `roles/compute.networkAdmin` )
  - [Viewer](https://docs.cloud.google.com/iam/docs/roles-overview#basic) ( `roles/viewer` )

  For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

  These predefined roles contain the permissions required to perform live migration. To see the exact permissions that are required, expand the **Required permissions** section:

  #### Required permissions

  The following permissions are required to perform live migration:

  - `compute.globalAddresses.create`
  - `compute.globalAddresses.createInternal`
  - `compute.globalAddresses.createInternal`
  - `compute.globalAddresses.delete`
  - `compute.globalAddresses.deleteInternal`
  - `compute.globalAddresses.get`
  - `compute.globalOperations.get`
  - `compute.networks.addPeering`
  - `compute.networks.get`
  - `compute.networks.listPeeringRoutes`
  - `compute.networks.removePeering`
  - `compute.networks.use`
  - `compute.routes.get`
  - `compute.routes.list`
  - `compute.subnetworks.get`
  - `compute.subnetworks.list`
  - `dataflow.jobs.cancel`
  - `dataflow.jobs.create`
  - `dataflow.jobs.updateContents`
  - `datastream.connectionProfiles.create`
  - `datastream.connectionProfiles.delete`
  - `datastream.privateConnections.create`
  - `datastream.privateConnections.delete`
  - `datastream.streams.create`
  - `datastream.streams.delete`
  - `datastream.streams.update`
  - `iam.roles.get`
  - `iam.serviceAccounts.actAs`
  - `pubsub.subscriptions.create`
  - `pubsub.subscriptions.delete`
  - `pubsub.topics.attachSubscription`
  - `pubsub.topics.create`
  - `pubsub.topics.delete`
  - `pubsub.topics.getIamPolicy`
  - `pubsub.topics.setIamPolicy`
  - `resourcemanager.projects.setIamPolicy`
  - `storage.buckets.create`
  - `storage.buckets.delete`
  - `storage.buckets.update`
  - `storage.objects.delete`

  Your administrator might also be able to give the service account these permissions with [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

## Set up CDC migration

Spanner offers a Terraform template setting up the CDC and later, completing the CDC migration. You can use a Terraform variable to disable the template from creating the Dataflow jobs. The Terraform template deploys and manages the following resources to set up the CDC migration:

- **Datastream private connection** : a private Datastream private connection is deployed on your configured VPC.

- **Source Datastream connection profile** : a connection profile that lets Datastream connect to your source MySQL instance.

- **Cloud Storage bucket** : a Cloud Storage bucket that Datastream writes the data to.

- **Target Datastream connection profile** : this connection profile lets Datastream connect and write to the Cloud Storage bucket.

- **Datastream stream** : a Datastream stream that reads from your source MySQL instance and writes to the Cloud Storage bucket as defined in the connection profiles.

- **Pub/Sub topic and subscription** : the Cloud Storage bucket sends object notifications to the Pub/Sub topic and Dataflow consumes the Pub/Sub subscription to write data to Spanner.

- **Cloud Storage bucket notifications** : a Cloud Storage bucket notification that publishes to the Pub/Sub topic.

### Preparing the CDC Terraform configuration

You can prepare the [Terraform template](https://github.com/GoogleCloudPlatform/DataflowTemplates/tree/main/v2/datastream-to-spanner/terraform/samples/mysql-end-to-end) to include Dataflow variable configurations, but disable the creation of Dataflow jobs:

```
common_params = {
  project = "PROJECT_ID"
  region  = "GCP_REGION"
}
datastream_params = {
  mysql_host = "MYSQL_HOST_IP_ADDRESS"
  mysql_username = "MYSQL_USERNAME"
  mysql_password = "MYSQL_PASSWORD"
  mysql_port     = 3306
  mysql_database = {
    database = "DATABASE_NAME"
  }
  private_connectivity = {
    vpc_name = "VPC_NAME"
    range = "RESERVED_RANGE"
  }
}
dataflow_params = {
  skip_dataflow = false
  enable_backfill = false
  template_params = {
    spanner_database_id = "SPANNER_DATABASE_ID"
    spanner_instance_id = "SPANNER_INSTANCE_ID"
  }
  runner_params = {
    max_workers = 10
    num_workers = 4
    on_delete   = "cancel"
    network     = "VPC_NETWORK"
    subnetwork  = "SUBNETWORK_NAME"
  }
}
```

Replace the following:

- `PROJECT_ID` : the Google Cloud project ID.
- `GCP_REGION` : the Google Cloud region where you want to run the Dataflow job.
- `MYSQL_HOST_IP_ADDRESS` : your source MySQL instance IP address.
- `MYSQL_USERNAME` : your source MySQL instance username.
- `MYSQL_PASSWORD` : your source MySQL instance password.
- `DATABASE_NAME` : your source MySQL database name in the instance.
- `VPC_NAME` : the name of an existing VPC used by Datastream.
- `RESERVED_RANGE` : the IP address range on the VPC that you reserved for Datastream.
- `SPANNER_DATABASE_ID` : the target Spanner database ID.
- `SPANNER_INSTANCE_ID` : the target Spanner instance ID.
- `VPC_NETWORK` : the name of an existing VPC used by Dataflow.
- `SUBNETWORK_NAME` : the name of the designated subnetwork in the VPC where Dataflow creates workers.

### Run the CDC Terraform template

To perform the CDC migration, you need to run the Terraform template:

1.  Initialize Terraform by using the following command:

    ```
      terraform init
    ```

2.  Validate the Terraform files by using the following command:

    ```
      terraform plan --var-file=terraform_simple.tfvars
    ```

3.  Run the Terraform configuration using the following command:

    ```
      terraform apply --var-file=terraform_simple.tfvars
    ```

    The Terraform configuration produces output similar to the following:

    ```
    Outputs:
    resource_ids = {
      "datastream_source_connection_profile" = "source-mysql-thorough-wombat"
      "datastream_stream" = "mysql-stream-thorough-wombat"
      "datastream_target_connection_profile" = "target-gcs-thorough-wombat"
      "gcs_bucket" = "live-migration-thorough-wombat"
      "pubsub_subscription" = "live-migration-thorough-wombat-sub"
      "pubsub_topic" = "live-migration-thorough-wombat"
    }
    resource_urls = {
      "datastream_source_connection_profile" = "https://console.cloud.google.com/datastream/connection-profiles/locations/us-central1/instances/source-mysql-thorough-wombat?project=your-project-here"
      "datastream_stream" = "https://console.cloud.google.com/datastream/streams/locations/us-central1/instances/mysql-stream-thorough-wombat?project=your-project-here"
      "datastream_target_connection_profile" = "https://console.cloud.google.com/datastream/connection-profiles/locations/us-central1/instances/target-gcs-thorough-wombat?project=your-project-here"
      "gcs_bucket" = "https://console.cloud.google.com/storage/browser/live-migration-thorough-wombat?project=your-project-here"
      "pubsub_subscription" = "https://console.cloud.google.com/cloudpubsub/subscription/detail/live-migration-thorough-wombat-sub?project=your-project-here"
      "pubsub_topic" = "https://console.cloud.google.com/cloudpubsub/topic/detail/live-migration-thorough-wombat?project=your-project-here"
    }
    ```

Datastream is now streaming the CDC to a Cloud Storage bucket. You must perform the backfill migration and finish the CDC migration later.

## Perform the backfill migration

Spanner offers a Terraform template to perform the backfill migration. The Terraform template deploys and manages the following resource:

- **Dataflow job** : The Dataflow job that reads from the source MySQL instance and writes to the target Spanner database.

### Preparing the backfill migration Terraform configuration

```
job_name = "JOB_NAME"
project = "PROJECT_ID"
region = "GCP_REGION"
working_directory_bucket = "WORKING_DIRECTORY_BUCKET"
working_directory_prefix = "WORKING_DIRECTORY_PREFIX"
source_config_url = "SOURCE_CONFIG_URL"
username = "USERNAME"
password = "PASSWORD"
instance_id = "SPANNER_INSTANCE_ID"
database_id  = "SPANNER_DATABASE_ID"
spanner_project_id = "SPANNER_PROJECT_ID"
```

Replace the following:

- `JOB_NAME` : the name of the Dataflow job.
- `PROJECT_ID` : the Google Cloud project ID where the Dataflow job runs.
- `GCP_REGION` : the Google Cloud region where you want to run the Dataflow job.
- `WORKING_DIRECTORY_BUCKET` : the Cloud Storage bucket for uploading the session file and creating the output directory.
- `WORKING_DIRECTORY_PREFIX` : the Cloud Storage bucket prefix for the Dataflow working directory.
- `SOURCE_CONFIG_URL` : your source MySQL instance IP address.
- `USERNAME` : your source MySQL instance username.
- `PASSWORD` : your source MySQL instance password.
- `SPANNER_INSTANCE_ID` : the target Spanner instance ID.
- `SPANNER_DATABASE_ID` : the target Spanner database ID.
- `SPANNER_PROJECT_ID` : the project ID where your Spanner instance resides. This project ID can be different from the project where you run Dataflow.

### Run the backfill migration Terraform template

To perform the backfill migration, do the following:

1.  Initialize Terraform by using the following command:

    ```
      terraform init
    ```

2.  Validate the Terraform files by using the following command:

    ```
      terraform plan --var-file=terraform_simple.tfvars
    ```

3.  Run the Terraform configuration using the following command:

    ```
      terraform apply --var-file=terraform_simple.tfvars
    ```

    The Terraform configuration produces an output similar to the following:

    ```
    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    Outputs:
    dataflow_job_id = [
      "2024-06-05_00_41_11-4759981257849547781",
    ]
    dataflow_job_url = [
      "https://console.cloud.google.com/dataflow/jobs/gcp-region/2024-06-05_00_41_11-4759981257849547781",
    ]
    ```

When the backfill migration cannot write a row to Spanner, it writes that row to a dead-letter queue directory in a Cloud Storage bucket.

You can retry writing these rows from the dead-letter queue directory to Spanner before finishing the CDC migration.

To retry writing these rows from the dead-letter queue directory to Spanner before finishing the CDC migration, run the following command:

```
gcloud dataflow flex-template run JOB_NAME \
    --project=PROJECT_ID \
    --region=GCP_REGION \
    --template-file-gcs-location=gs://dataflow-templates/latest/flex/Cloud_Datastream_to_Spanner \
    --additional-experiments=use_runner_v2 \
    --parameters inputFilePattern=INPUT_FILE_PATTERN,\
      streamName="ignore",\
      datastreamSourceType=SOURCE_TYPE,\
      instanceId=INSTANCE_ID,databaseId=DATABASE_ID,sessionFilePath=SESSION_FILE_PATH,\
      deadLetterQueueDirectory=DLQ_DIRECTORY,\
      runMode="retryDLQ"
```

Replace the following:

- `JOB_NAME` : the name of the Dataflow job.
- `PROJECT_ID` : the Google Cloud project ID where the Dataflow job runs.
- `GCP_REGION` : the Google Cloud region where you want to run the Dataflow job.
- `INPUT_FILE_PATTERN` : the Cloud Storage bucket location of the input file pattern.
- `SOURCE_TYPE` : the source database type (for example, `mysql` ).
- `INSTANCE_ID` : the target Spanner instance ID.
- `DATABASE_ID` : the target Spanner database ID.
- `SESSION_FILE_PATH` : the Cloud Storage path to the session file generated by the Spanner migration tool.
- `DLQ_DIRECTORY` : the Cloud Storage path to the dead-letter queue root directory containing the error files to retry (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/` ).

## Perform the CDC migration

After the backfill migration is complete, you can use Dataflow to migrate the CDC to Spanner. The Dataflow job takes the change events from the Cloud Storage bucket and writes them to Spanner.

To perform the CDC migration, change the value of the `skip_dataflow` Terraform parameter to `false` and rerun the live migration [Terraform template](https://github.com/GoogleCloudPlatform/DataflowTemplates/tree/main/v2/datastream-to-spanner/terraform/samples/mysql-end-to-end) .

Run the Terraform configuration using the following command:

```
      terraform apply --var-file=terraform_simple.tfvars
    
```

## Monitor and resolve CDC errors

During live change data capture (CDC) replication from MySQL to Spanner, if your streaming Dataflow job can't write a change event to Spanner, it classifies the failure into one of two error categories and writes the record to a DLQ directory in the `deadLetterQueueDirectory` Cloud Storage bucket:

- **Retryable errors** : Temporary failures, which include but are not limited to the following:

  - Spanner deadline exceeded errors
  - Foreign key and interleaved table constraint violations. This can occur when a child row arrives before its parent row due to network errors or parallel processing.

  Dataflow writes these events to a `retry/` DLQ directory and retries them within the same pipeline until they succeed or exceed the maximum retry limit.

- **Severe errors** : Permanent failures that can't be resolved by automated retries, which include but are not limited to the following:

  - Data type conversion errors
  - Unique index violations
  - Exceptions inside custom transformation JARs

  Dataflow writes these events to a `severe/` DLQ directory and increments the `Total permanent errors` custom user counter.

Because severe errors cannot be resolved by automated retries, Dataflow removes those failed events from the active replication stream and saves them in the `severe/` directory. If left unhandled, your destination Spanner database will be missing those records. Regularly monitor the `Total permanent errors` counter and the `severe/` directory during live migration so you can resolve and replay failed change events before initiating application cutover.

Note that the Dataflow custom counter `Total permanent errors` is a cumulative metric that records the total number of severe errors encountered since the streaming job started. When you replay and resolve failed records using a DLQ retry job, this counter on your original streaming pipeline does not decrement. Therefore, your Cloud Storage severe DLQ directory ( `gs://.../severe/` ) is the authoritative source of truth for whether unhandled errors exist.

### Replay failed records

When failed records appear in your severe DLQ directory, inspect the JSON error files in Cloud Storage, identify the root cause, and resolve the underlying issue in Spanner (such as manually inserting missing parent records or updating schema override mappings).

To replay corrected DLQ records into Spanner, launch a new Dataflow job using one of the following DLQ retry run modes:

- **`retryDLQ` mode** : Run this mode as a batch job alongside your active streaming pipeline to process error files in the `severe/` directory without halting live CDC replication.
- **`retryAllDLQ` mode** : If your regular streaming pipeline is stopped, use this mode to process both the `retry/` and `severe/` directories. Don't run `retryAllDLQ` concurrently with an active pipeline, as it will cause file contention and processing conflicts.

To launch a DLQ retry job using the gcloud CLI CLI, run the following command:

```
gcloud dataflow flex-template run RETRY_JOB_NAME \
    --project=PROJECT_ID \
    --region=GCP_REGION \
    --template-file-gcs-location=gs://dataflow-templates/latest/flex/Cloud_Datastream_to_Spanner \
    --additional-experiments=use_runner_v2 \
    --parameters streamName="ignore",\
datastreamSourceType=SOURCE_TYPE,\
instanceId=INSTANCE_ID,databaseId=DATABASE_ID,sessionFilePath=SESSION_FILE_PATH,\
deadLetterQueueDirectory=DLQ_DIRECTORY,\
runMode="retryDLQ"
```

Replace the following:

- `RETRY_JOB_NAME` : the name of the Dataflow DLQ retry job.
- `PROJECT_ID` : the Google Cloud project ID where the Dataflow job runs.
- `GCP_REGION` : the Google Cloud region where you want to run the Dataflow job.
- `SOURCE_TYPE` : the source database type (for example, `mysql` ).
- `INSTANCE_ID` : the target Spanner instance ID.
- `DATABASE_ID` : the target Spanner database ID.
- `SESSION_FILE_PATH` : the Cloud Storage path to the session file generated by the Spanner migration tool.
- `DLQ_DIRECTORY` : the Cloud Storage path to the dead-letter queue root directory containing the error files to retry (for example, `gs://MIGRATION_BUCKET/ENV/SERVICE/cdc/DB_SCHEMA/` ).

## Finish the CDC migration

After almost all the data from the Cloud Storage bucket is written to Spanner, stop writes on the source MySQL instance to allow the remaining changes to be written to Spanner.

This causes a short downtime while Spanner catches up to the source MySQL instance. After all changes are written to Spanner, your application can start using Spanner as its database.

For detailed step-by-step instructions to pause application writes, drain in-flight data, and perform application cutover, see [Perform cutover from MySQL](https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover) .

## What's next

- [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback)
- [Perform cutover from MySQL](https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover)
- [Perform fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-fallback)
