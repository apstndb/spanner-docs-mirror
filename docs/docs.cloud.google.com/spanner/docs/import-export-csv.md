This page describes how to export data from Spanner to CSV files or import data from CSV files into Spanner GoogleSQL-dialect databases or PostgreSQL-dialect databases.

  - If you want to import a Spanner database that you previously exported to Avro files in Cloud Storage, see [Import Spanner Avro files](/spanner/docs/import) .
  - If you want to import Avro files from a non-Spanner database, see [Import data from non-Spanner databases](/spanner/docs/import-non-spanner) .

The process uses [Dataflow](/dataflow) . You can export data from Spanner to a [Cloud Storage](/storage) bucket, or you can import data into Spanner from a Cloud Storage bucket that contains a JSON manifest file and a set of CSV files.

**Note:** To explore Spanner using a 90-day free trial instance, see [Create a Spanner free trial instance](/spanner/docs/free-trial-quickstart) .

## Before you begin

To import or export a Spanner database, first you need to enable the Spanner, Cloud Storage, Compute Engine, and Dataflow APIs:

**Roles required to enable APIs**

To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

You also need enough quota and the required IAM permissions.

### Quota requirements

The quota requirements for import or export jobs are as follows:

  - **Spanner** : You must have enough [compute capacity](/spanner/docs/compute-capacity) to support the amount of data that you are importing. No additional compute capacity is required to import or export a database, though you might need to add more compute capacity so that your job finishes in a reasonable amount of time. See [Optimize jobs](#optimize-slow) for more details.

  - **Cloud Storage** : To import, you must have a bucket containing your previously exported files. To export, you must create a bucket for your exported files if you don't already have one. You can do this in the Google Cloud console, either through the Cloud Storage page or while creating your export through the Spanner page. You don't need to set a size for your bucket.

  - **Dataflow** : Import or export jobs are subject to the same CPU, disk usage, and IP address [Compute Engine quotas](/dataflow/quotas#compute-engine-quotas) as other Dataflow jobs.

  - **Compute Engine** : Before running your import or export job, you must [set up initial quotas](https://support.google.com/cloud/answer/6075746) for Compute Engine, which Dataflow uses. These quotas represent the *maximum* number of resources that you allow Dataflow to use for your job. Recommended starting values are:
    
      - **CPUs** : 200
      - **In-use IP addresses** : 200
      - **Standard persistent disk** : 50 TB
    
    Generally, you don't have to make any other adjustments. Dataflow provides autoscaling so that you only pay for the actual resources used during the import or export. If your job can make use of more resources, the Dataflow UI displays a warning icon. The job should finish even if there is a warning icon.

### Required roles

To get the permissions that you need to export a database, ask your administrator to grant you the following IAM roles on your Dataflow worker service account:

  - [Cloud Spanner Viewer](/iam/docs/roles-permissions/spanner#spanner.viewer) ( `  roles/spanner.viewer  ` )
  - [Dataflow Worker](/iam/docs/roles-permissions/dataflow#dataflow.worker) ( `  roles/dataflow.worker  ` )
  - [Storage Admin](/iam/docs/roles-permissions/storage#storage.admin) ( `  roles/storage.admin  ` )
  - [Spanner Database Reader](/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `  roles/spanner.databaseReader  ` )
  - [Database Admin](/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `  roles/spanner.databaseAdmin  ` )

**Note:** The Spanner Database Admin role is only required for import jobs.

## Export Spanner data to CSV files

To export data from Spanner to CSV files in Cloud Storage, follow the instructions for using the Google Cloud CLI to run a job with the [Spanner to Cloud Storage Text template](/dataflow/docs/guides/templates/provided/cloud-spanner-to-cloud-storage) .

You can also refer to the information in this page about [optimizing slow jobs](/spanner/docs/import-export-csv#optimize-slow) , and [factors affecting job performance](/spanner/docs/import-export-csv#performance-factors) .

**Note:** Neither [change stream](/spanner/docs/change-streams) data nor the values in stored [generated columns](/spanner/docs/generated-column/how-to) are exported.

## Import data from CSV files into Spanner

The process to import data from CSV files includes the following steps:

1.  Export your data to CSV files and store those files in Cloud Storage. Don't include a header line.
2.  Create a JSON manifest file and store the file along with your CSV files.
3.  Create empty target tables in your Spanner database **or** ensure that the data types for columns in your CSV files match any corresponding columns in your existing tables.
4.  Run your import job.

**Note:** The [dataflow template](/dataflow/docs/guides/templates/provided/cloud-storage-to-cloud-spanner) can't handle a CSV file with headers.

### Step 1: Export data from a non-Spanner database to CSV files

The import process brings data in from CSV files located in a Cloud Storage bucket. You can export data in CSV format from any source.

Keep the following things in mind when exporting your data:

  - Text files to be imported must be in CSV format.

  - Data must match one of the following types:

### GoogleSQL

``` googlesql
BOOL
INT64
FLOAT64
NUMERIC
STRING
DATE
TIMESTAMP
BYTES
JSON
```

### PostgreSQL

``` text
boolean
bigint
double precision
numeric
character varying, text
date
timestamp with time zone
bytea
```

  - You don't have to include or generate any metadata when you export the CSV files.

  - You don't have to follow any particular naming convention for your files.

If you don't export your files directly to Cloud Storage, you must [upload the CSV files](/storage/docs/uploading-objects) to a Cloud Storage bucket.

### Step 2: Create a JSON manifest file

You must also create a manifest file with a JSON description of files to import and place it in the same Cloud Storage bucket where you stored your CSV files. This manifest file contains a `  tables  ` array that lists the name and data file locations for each table. The file also specifies the receiving database dialect. If the dialect is omitted, it defaults to GoogleSQL.

**Note:** If a table has [generated columns](/spanner/docs/generated-column/how-to) , the manifest must include an explicit list of the non-generated columns to import for that table. Spanner uses this list to map CSV columns to the correct table columns. Generated column values automatically computed during import.

The format of the manifest file corresponds to the following message type, shown here in [protocol buffer](https://developers.google.com/protocol-buffers/docs/proto3) format:

``` text
message ImportManifest {
  // The per-table import manifest.
  message TableManifest {
    // Required. The name of the destination table.
    string table_name = 1;
    // Required. The CSV files to import. This value can be either a filepath or a glob pattern.
    repeated string file_patterns = 2;
    // The schema for a table column.
    message Column {
      // Required for each Column that you specify. The name of the column in the
      // destination table.
      string column_name = 1;
      // Required for each Column that you specify. The type of the column.
      string type_name = 2;
    }
    // Optional. The schema for the table columns.
    repeated Column columns = 3;
  }
  // Required. The TableManifest of the tables to be imported.
  repeated TableManifest tables = 1;

  enum ProtoDialect {
    GOOGLE_STANDARD_SQL = 0;
    POSTGRESQL = 1;
  }
  // Optional. The dialect of the receiving database. Defaults to GOOGLE_STANDARD_SQL.
  ProtoDialect dialect = 2;
}
```

The following example shows a manifest file for importing tables called `  Albums  ` and `  Singers  ` into a GoogleSQL-dialect database. The `  Albums  ` table uses the column schema that the job retrieves from the database, and the `  Singers  ` table uses the schema that the manifest file specifies:

``` text
{
  "tables": [
    {
      "table_name": "Albums",
      "file_patterns": [
        "gs://bucket1/Albums_1.csv",
        "gs://bucket1/Albums_2.csv"
      ]
    },
    {
      "table_name": "Singers",
      "file_patterns": [
        "gs://bucket1/Singers*.csv"
      ],
      "columns": [
        {"column_name": "SingerId", "type_name": "INT64"},
        {"column_name": "FirstName", "type_name": "STRING"},
        {"column_name": "LastName", "type_name": "STRING"}
      ]
    }
  ]
}
```

### Step 3: Create the table for your Spanner database

Before you run your import, you must create the target tables in your Spanner database. If the target Spanner table already has a schema, any columns specified in the manifest file *must have the same data types* as the corresponding columns in the target table's schema.

We recommend that you create secondary indexes, foreign keys, and change streams after you import your data into Spanner, not when you initially create the table. If your table already contains these structures, then we recommend dropping them and re-creating them after you import your data.

### Step 4: Run a Dataflow import job using gcloud

To start your import job, follow the instructions for using the Google Cloud CLI to run a job with the [Cloud Storage Text to Spanner template](/dataflow/docs/guides/templates/provided-batch#gcs_text_to_cloud_spanner) .

After you have started an import job, you can [see details about the job](/spanner/docs/import-export-csv#view-job-details) in the Google Cloud console.

After the import job is finished, add any necessary [secondary indexes](/spanner/docs/secondary-indexes) , [foreign keys](/spanner/docs/foreign-keys/overview) , and [change streams](/spanner/docs/change-streams) .

**Note:** To avoid [outbound data transfer charges](/storage/pricing#network-pricing) , [choose a region](/spanner/docs/import-export-csv#choose-region) that overlaps with your Cloud Storage bucket's location.

## Choose a region for your import job

You might want to choose a different region based on the location of your Cloud Storage bucket. To avoid [outbound data transfer charges](/storage/pricing#network-pricing) , choose a region that matches your Cloud Storage bucket's location.

  - If your Cloud Storage bucket location is a [region](/storage/docs/bucket-locations#location-r) , you can take advantage of [free network usage](/storage/pricing#network-buckets) by choosing the same region for your import job, assuming that region is available.

  - If your Cloud Storage bucket location is a [dual-region](/storage/docs/bucket-locations#location-dr) , you can take advantage of [free network usage](/storage/pricing#network-buckets) by choosing one of the two regions that make up the dual-region for your import job, assuming one of the regions is available.

<!-- end list -->

  - If a co-located region is not available for your import job, or if your Cloud Storage bucket location is a [multi-region](/storage/docs/bucket-locations#location-mr) , outbound data transfer charges apply. Refer to Cloud Storage [data transfer](/storage/pricing#network-pricing) pricing to choose a region that incurs the lowest data transfer charges.

## View or troubleshoot jobs in the Dataflow UI

After you start an import or export job, you can view details of the job, including logs, in the Dataflow section of the Google Cloud console.

### View Dataflow job details

To see details for any import or export jobs that you ran within the last week, including any jobs that are running now:

1.  Navigate to the **Database overview** page for the database.

2.  Click the **Import/Export** left pane menu item. The database **Import/Export** page displays a list of recent jobs.

3.  In the database **Import/Export** page, click the job name in the **Dataflow job name** column:
    
    The Google Cloud console displays details of the Dataflow job.

To view a job that you ran more than one week ago:

1.  Go to the Dataflow jobs page in the Google Cloud console.

2.  Find your job in the list, then click its name.
    
    The Google Cloud console displays details of the Dataflow job.

**Note:** Jobs of the same type for the same database have the same name. You can tell jobs apart by the values in their **Start time** or **End time** columns.

### View Dataflow logs for your job

To view a Dataflow job's logs, navigate to the job's details page, then click **Logs** to the right of the job's name.

If a job fails, look for errors in the logs. If there are errors, the error count displays next to **Logs** :

To view job errors:

1.  Click the error count next to **Logs** .
    
    The Google Cloud console displays the job's logs. You may need to scroll to see the errors.

2.  Locate entries with the error icon .

3.  Click an individual log entry to expand its contents.

For more information about troubleshooting Dataflow jobs, see [Troubleshoot your pipeline](/dataflow/pipelines/troubleshooting-your-pipeline#basic-troubleshooting-workflow) .

### Troubleshoot failed import or export jobs

If you see the following errors in your job logs:

``` text
com.google.cloud.spanner.SpannerException: NOT_FOUND: Session not found

--or--

com.google.cloud.spanner.SpannerException: DEADLINE_EXCEEDED: Deadline expired before operation could complete.
```

Check the *99% Read/Write latency* in the **Monitoring** tab of your Spanner database in the Google Cloud console. If it is showing high (multiple second) values, then it indicates that the instance is overloaded, causing reads/writes to timeout and fail.

One cause of high latency is that the Dataflow job is running using too many workers, putting too much load on the Spanner instance.

To specify a limit on the number of Dataflow workers:

### Console

If you are using the Dataflow console, the **Max workers** parameter is located in the **Optional parameters** section of the **Create job from template** page.

### gcloud

Run the [`  gcloud dataflow jobs run  `](/sdk/gcloud/reference/dataflow/jobs/run) command, and specify the `  max-workers  ` argument. For example:

``` text
  gcloud dataflow jobs run my-import-job \
    --gcs-location='gs://dataflow-templates/latest/GCS_Text_to_Cloud_Spanner' \
    --region=us-central1 \
    --parameters='instanceId=test-instance,databaseId=example-db,inputDir=gs://my-gcs-bucket' \
    --max-workers=10 \
    --network=network-123
```

### Troubleshoot network error

The following error might occur when you export your Spanner databases:

``` text
Workflow failed. Causes: Error: Message: Invalid value for field
'resource.properties.networkInterfaces[0].subnetwork': ''. Network interface
must specify a subnet if the network resource is in custom subnet mode.
HTTP Code: 400
```

This error occurs because Spanner assumes that you intend to use an auto mode VPC network named `  default  ` in the same project as the Dataflow job. If you don't have a default VPC network in the project, or if your VPC network is in a custom mode VPC network, then you must create a Dataflow job and [specify an alternate network or subnetwork](/dataflow/docs/guides/specifying-networks?) .

## Optimize slow running import or export jobs

If you have followed the suggestions in [initial settings](#quota) , you should generally not have to make any other adjustments. If your job is running slowly, there are a few other optimizations you can try:

  - **Optimize the job and data location** : Run your Dataflow job [in the same region](#choose-region) where your Spanner instance and Cloud Storage bucket are located.

  - **Ensure sufficient Dataflow resources** : If the [relevant Compute Engine quotas](/dataflow/quotas#compute-engine-quotas) limit your Dataflow job's resources, the job's [Dataflow page](#dataflow-job-details) in the Google Cloud console displays a warning icon and log messages:
    
    In this situation, [increasing the quotas](https://support.google.com/cloud/answer/6075746) for CPUs, in-use IP addresses, and standard persistent disk might shorten the run time of the job, but you might incur more Compute Engine charges.

  - **Check the Spanner CPU utilization** : If you see that the CPU utilization for the instance is over 65%, you can increase the [compute capacity](/spanner/docs/compute-capacity) in that instance. The capacity adds more Spanner resources and the job should speed up, but you incur more Spanner charges.

## Factors affecting import or export job performance

Several factors influence the time it takes to complete an import or export job.

  - **Spanner database size** : Processing more data takes more time and resources.

  - **Spanner database schema** , including:
    
      - The number of tables
      - The size of the rows
      - The number of secondary indexes
      - The number of foreign keys
      - The number of change streams

<!-- end list -->

  - **Data location** : Data is transferred between Spanner and Cloud Storage using Dataflow. Ideally all three components are located in the same region. If the components are not in the same region, moving the data across regions slows the job down.

  - **Number of Dataflow workers** : Optimal Dataflow workers are necessary for good performance. By using autoscaling, Dataflow chooses the number of workers for the job depending on the amount of work that needs to be done. The number of workers will, however, be capped by the quotas for CPUs, in-use IP addresses, and standard persistent disk. The Dataflow UI displays a warning icon if it encounters quota caps. In this situation, progress is slower, but the job should still complete. Autoscaling can overload Spanner leading to errors when there is a large amount of data to import.

  - **Existing load on Spanner** : An import job adds significant CPU load on a Spanner instance. An export job typically adds a light load on a Spanner instance. If the instance already has a substantial existing load, then the job runs more slowly.

  - **Amount of Spanner compute capacity** : If the CPU utilization for the instance is over 65%, then the job runs more slowly.

## Tune workers for good import performance

When starting a Spanner import job, Dataflow workers must be set to an optimal value for good performance. Too many workers overloads Spanner and too few workers results in an underwhelming import performance.

The maximum number of workers is heavily dependent on the data size, but ideally, the total Spanner CPU utilization should be between 70% to 90%. This provides a good balance between Spanner efficiency and error-free job completion.

To achieve that utilization target in the majority of schemas and scenarios, we recommend a max number of worker vCPUs between 4-6x the number of Spanner nodes.

For example, for a 10 node Spanner instance, using n1-standard-2 workers, you would set max workers to 25, giving 50 vCPUs.
