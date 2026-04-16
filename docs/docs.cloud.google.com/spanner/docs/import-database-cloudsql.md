This page describes how to import data from Cloud SQL for MySQL into Spanner.

The process uses Cloud Shell on Google Cloud console to run commands that configure and run a [Dataflow](https://docs.cloud.google.com/dataflow/docs) job to import a database from Cloud SQL into Spanner.

## Process overview

The import process involves the following:

1.  You complete a Google Cloud console workflow where you provide information about your source and target databases:
      - **Source database details** : Cloud SQL instance name, database name, and your credentials.
      - **Spanner details** : Your Spanner instance name, and database name. The command creates the database if it doesn't already exist.
      - **Output storage** : A Cloud Storage bucket name to store output files.
2.  Spanner opens Cloud Shell and populates a command. The command performs the following actions:
      - **Migrates the schema** : The command migrates the schema using the Spanner migration tool. This migration runs in Cloud Shell and uses a public IP address to connect to your Cloud SQL instance. Because Cloud Shell is on its own network, it needs access to Cloud SQL using the public IP address; however, you don't need to allowlist any subnets against the public IP address.
      - **Starts a data migration** : After the tool migrates the schema, the command starts a Dataflow job for data migration. The job reads from the source database directly through its private IP address and writes to Spanner. This job runs using the default Compute Engine service account. Finally, the command prints the Dataflow job URL.

## Limitations

The following limitations apply:

  - This data import only supports a single Cloud SQL for MySQL instance.
  - Schema conversion is automated; you can't make adjustments to the schema during this import.
  - This data import is a one-time bulk load; it doesn't support continuous replication.

## Before you begin

Before you import your database, complete the following prerequisites:

1.  Ensure that your Cloud SQL instance has a public IP address and a private IP address enabled. For more information, see [Configuring public IP connectivity](https://docs.cloud.google.com/sql/docs/mysql/configure-ip) and [Configure private IP](https://docs.cloud.google.com/sql/docs/mysql/configure-private-ip) .

2.  Create a user and password for your Cloud SQL instance that can be used to query the database.

3.  Store the password in Secret Manager. You need the `version ID` of the secret version. For more information, see [Create a secret](https://docs.cloud.google.com/secret-manager/docs/create-secret-quickstart) .

4.  Ensure you have a Cloud Storage bucket. Dataflow uses this bucket to store configuration files and outputs of the Dataflow jobs.

5.  Ensure that Spanner and Cloud SQL are in the same Google Cloud project.

6.  Enable the Dataflow, Cloud Storage, Spanner, Cloud SQL, and Secret Manager APIs.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `roles/serviceusage.serviceUsageAdmin` ), which contains the `serviceusage.services.enable` permission. [Learn how to grant roles](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

### Required roles

To ensure that the default Compute Engine service account has the necessary permissions to run the Dataflow job, ask your administrator to grant the following IAM roles to the default Compute Engine service account on your project:

> **Important:** You must grant these roles to the default Compute Engine service account, *not* to your user account. Failure to grant the roles to the correct principal might result in permission errors.

  - [Secret Manager Secret Accessor](https://docs.cloud.google.com/iam/docs/roles-permissions/secretmanager#secretmanager.secretAccessor) ( `roles/secretmanager.secretAccessor` )
  - [Cloud SQL Client](https://docs.cloud.google.com/iam/docs/roles-permissions/cloudsql#cloudsql.client) ( `roles/cloudsql.client` )
  - [Cloud Spanner Database Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `roles/spanner.databaseAdmin` )
  - [Storage Object Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.objectAdmin) ( `roles/storage.objectAdmin` )
  - [Dataflow Worker](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.worker) ( `roles/dataflow.worker` )

To get the permissions that you need to configure the import, ask your administrator to grant you the following IAM roles on your project:

  - [Cloud SQL Client](https://docs.cloud.google.com/iam/docs/roles-permissions/cloudsql#cloudsql.client) ( `roles/cloudsql.client` )
  - [Cloud Spanner Database Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `roles/spanner.databaseAdmin` )
  - [Secret Manager Secret Accessor](https://docs.cloud.google.com/iam/docs/roles-permissions/secretmanager#secretmanager.secretAccessor) ( `roles/secretmanager.secretAccessor` )
  - [Storage Admin](https://docs.cloud.google.com/iam/docs/roles-permissions/storage#storage.admin) ( `roles/storage.admin` )
  - [Dataflow Developer](https://docs.cloud.google.com/iam/docs/roles-permissions/dataflow#dataflow.developer) ( `roles/dataflow.developer` )
  - [Service Account User](https://docs.cloud.google.com/iam/docs/roles-permissions/iam#iam.serviceAccountUser) ( `roles/iam.serviceAccountUser` )

These predefined roles contain the permissions required to configure the import. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

The following permissions are required to configure the import:

  - `cloudsql.instances.connect`
  - `cloudsql.instances.get`
  - `cloudsql.instances.login`
  - `spanner.instances.list`
  - `spanner.instances.get`
  - `spanner.databases.create`
  - `spanner.databases.list`
  - `spanner.databases.get`
  - `spanner.databases.getDdl`
  - `spanner.databases.updateDdl`
  - `spanner.databases.read`
  - `spanner.databases.write`
  - `spanner.databases.select`
  - `secretmanager.versions.access`
  - `storage.objects.create`
  - `storage.objects.get`
  - `storage.buckets.get`
  - `dataflow.jobs.create`
  - `dataflow.jobs.get`
  - `dataflow.jobs.list`
  - `iam.serviceAccounts.actAs`

## Quota requirements

The quota requirements are as follows:

  - **Spanner** : You must have enough [compute capacity](https://docs.cloud.google.com/spanner/docs/compute-capacity) to support the amount of data that you are importing. We recommend starting with a minimum of one Spanner node. You might need to add more compute capacity so that your job finishes in a reasonable amount of time. No additional compute capacity is required to import a database schema. For more information, see [Autoscaling overview](https://docs.cloud.google.com/spanner/docs/autoscaling-overview)

  - **Dataflow** : Import jobs are subject to the same CPU, disk usage, and IP address [Compute Engine quotas](https://docs.cloud.google.com/dataflow/quotas#compute-engine-quotas) as other Dataflow jobs.

  - **Compute Engine** : Before running your import job, you must [set up initial quotas](https://support.google.com/cloud/answer/6075746) for Compute Engine, which Dataflow uses. These quotas represent the *maximum* number of resources that you allow Dataflow to use for your job. Recommended starting values are:
    
      - **CPUs** : 200
      - **In-use IP addresses** : 200
      - **Standard persistent disk** : 50 TB
    
    Generally, you don't have to make any other adjustments. Dataflow provides autoscaling so that you only pay for the actual resources used during the import. If your job can make use of more resources, the Dataflow UI displays a warning icon. The job can finish even if there is a warning icon.

## Import from Cloud SQL to Spanner

To import a Cloud SQL for MySQL database to Spanner, do the following in the Google Cloud console:

1.  Go to the Spanner **Instances** page.

2.  Click the name of the instance where the database needs to be imported.

3.  Click the **Import from Cloud SQL** button.

4.  After Spanner verifies that all required APIs are enabled, click the **Next** button.

5.  Select the Cloud SQL for MySQL instance to import, then click the **Next** button.

6.  Select the database to import, then click the **Next** button. Spanner verifies if the public IP for your Cloud SQL instance is enabled.

7.  Enter the username and secret, then click the **Next** button.

8.  Browse and select the Cloud Storage bucket, then click the **Next** button.

9.  Enter the Spanner database name, then click the **Import** button. Spanner opens Cloud Shell and populates a command.

10. Run the auto populated command to start the import:
    
        export SOURCE_PROJECT_NUMBER=$(gcloud projects describe \
            "SOURCE_PROJECT_ID" \
            --format="value(projectNumber)") && \
        export GSA_EMAIL="${SOURCE_PROJECT_NUMBER}-compute@developer.gserviceaccount.com" && \
        echo "Verifying permissions for ${GSA_EMAIL}..." && \
        export CURRENT_ROLES=$(gcloud projects get-iam-policy \
            "SOURCE_PROJECT_ID" \
            --flatten="bindings[].members" \
            --filter="bindings.members:serviceAccount:${GSA_EMAIL}" \
            --format="value(bindings.role)") && \
        ERR=0 && \
        for ROLE in roles/secretmanager.secretAccessor \
            roles/cloudsql.client roles/spanner.databaseAdmin \
            roles/storage.objectAdmin roles/dataflow.worker; do \
          if echo "${CURRENT_ROLES}" | awk -v r="$ROLE" '$1 == r {found=1} END {exit 1-found}'; then \
            echo "[OK] $ROLE"; \
          else \
            echo "[MISSING] $ROLE. Run: gcloud projects add-iam-policy-binding SOURCE_PROJECT_ID --member='serviceAccount:${GSA_EMAIL}' --role='${ROLE}'"; \
            ERR=1; \
          fi; \
        done && \
        [[ "$ERR" -eq 0 ]] && \
        export JOB_NAME="csql-to-spanner-$(date +%Y%m%d-%H%M%S)" && \
        export OUTPUT_DIR="gs://BUCKET_NAME/output/${JOB_NAME}" && \
        export SHARD_CONFIG_PATH="gs://BUCKET_NAME/config/${JOB_NAME}_shard_config.json" && \
        export WORKER_MACHINE_TYPE="n2-highmem-8" && \
        export TEMPLATE_PATH="gs://dataflow-templates/latest/flex/Sourcedb_to_Spanner_Flex" && \
        export SHARD_CONFIG_JSON='{
          "shardConfigurationBulk": {
            "dataShards": [
              {
                "host": "SOURCE_PRIVATE_IP",
                "port": "3306",
                "user": "SOURCE_DATABASE_USER",
                "secretManagerUri": "projects/PROJECT_ID/secrets/SECRET_ID/versions/VERSION",
                "databases": [
                  {
                    "dbName": "SOURCE_DATABASE_NAME",
                    "databaseId": "SOURCE_DATABASE_NAME"
                  }
                ]
              }
            ]
          }
        }' && \
        echo "${SHARD_CONFIG_JSON}" | gcloud storage cp - "${SHARD_CONFIG_PATH}" && \
        sudo apt-get update && \
        sudo apt-get install google-cloud-cli-spanner-migration-tool -y && \
        gcloud alpha spanner migrate schema \
            --source=mysql \
            --source-profile="project=SOURCE_PROJECT_ID,instance=SOURCE_INSTANCE_NAME,secretManagerUri=projects/PROJECT_ID/secrets/SECRET_ID/versions/VERSION,dbName=SOURCE_DATABASE_NAME,region=SOURCE_REGION,user=SOURCE_DATABASE_USER" \
            --target-profile="instance=SPANNER_INSTANCE_ID,project=SPANNER_PROJECT_ID,dbName=SPANNER_DATABASE_ID" && \
        JOB_OUTPUT=$(gcloud dataflow flex-template run "${JOB_NAME}" \
            --project="SOURCE_PROJECT_ID" \
            --region="SOURCE_REGION" \
            --template-file-gcs-location="${TEMPLATE_PATH}" \
            --network="NETWORK_NAME" \
            --subnetwork="https://www.googleapis.com/compute/v1/projects/SOURCE_PROJECT_ID/regions/SOURCE_REGION/subnetworks/SUBNETWORK_NAME" \
            --worker-machine-type="${WORKER_MACHINE_TYPE}" \
            --parameters "instanceId=SPANNER_INSTANCE_ID" \
            --parameters "databaseId=SPANNER_DATABASE_ID" \
            --parameters "projectId=SPANNER_PROJECT_ID" \
            --parameters "sourceConfigURL=${SHARD_CONFIG_PATH}" \
            --parameters "sourceDbDialect=MYSQL" \
            --parameters "jdbcDriverClassName=com.mysql.jdbc.Driver" \
            --parameters "outputDirectory=${OUTPUT_DIR}" \
            --format="get(job.id)") && \
        echo "--------------------------------------------------------" && \
        echo "Dataflow Job Submitted." && \
        echo "Monitor: https://console.cloud.google.com/dataflow/jobs/SOURCE_REGION/${JOB_OUTPUT}?project=SOURCE_PROJECT_ID" && \
        echo "--------------------------------------------------------"
    
    The following parameters are provided from the Google Cloud console to the command:
    
      - `  SOURCE_DATABASE_NAME  ` : the name of the source Cloud SQL database
      - `  SOURCE_DATABASE_USER  ` : the username for the source Cloud SQL database
      - `  PROJECT_ID  ` : your Google Cloud project ID
      - `  SECRET_ID  ` : the ID of the secret containing the password
      - `  VERSION  ` : the version of the secret
      - `  SOURCE_PROJECT_ID  ` : the project ID containing the source Cloud SQL instance
      - `  SOURCE_REGION  ` : the region of the source Cloud SQL instance
      - `  SOURCE_INSTANCE_NAME  ` : the name of the source Cloud SQL instance
      - `  SOURCE_PRIVATE_IP  ` : the private IP address of the Cloud SQL instance
      - `  NETWORK_NAME  ` : the network name of the source Cloud SQL instance
      - `  SUBNETWORK_NAME  ` : the subnetwork name of the source Cloud SQL instance
      - `  SPANNER_PROJECT_ID  ` : the project ID containing the target Spanner instance
      - `  SPANNER_INSTANCE_ID  ` : the ID of the target Spanner instance
      - `  SPANNER_DATABASE_ID  ` : the ID of the target Spanner database, which Spanner creates if it doesn't exist
      - `  BUCKET_NAME  ` : the name of the Cloud Storage bucket to store Dataflow output files and configuration files
    
    The command verifies that the default compute service account has the required permissions, installs the Spanner migration tool, migrates the schema, and starts the Dataflow job.
    
    After the command finishes, follow the link provided to monitor the Dataflow job in the Google Cloud console.

## What's Next

  - Learn how to [make schema updates](https://docs.cloud.google.com/spanner/docs/schema-updates) .
