This page describes how to prepare Avro files that you exported from non- Spanner databases and then import those files into Spanner. These procedures include information for both GoogleSQL-dialect databases and PostgreSQL-dialect databases. If you want to import a Spanner database that you previously exported, see [Import Spanner Avro files](/spanner/docs/import) .

The process uses [Dataflow](/dataflow) ; it imports data from a [Cloud Storage](/storage) bucket that contains a set of [Avro files](https://en.wikipedia.org/wiki/Apache_Avro) and a [JSON manifest file](/spanner/docs/import-export-csv#create-json-manifest) that specifies the destination tables and Avro files that populate each table.

## Before you begin

To import a Spanner database, first you need to enable the Spanner, Cloud Storage, Compute Engine, and Dataflow APIs:

**Roles required to enable APIs**

To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

You also need enough quota and the required IAM permissions.

### Quota requirements

The quota requirements for import jobs are as follows:

  - **Spanner** : You must have enough [compute capacity](/spanner/docs/compute-capacity) to support the amount of data that you are importing. No additional compute capacity is required to import a database, though you might need to add more compute capacity so that your job finishes in a reasonable amount of time. See [Optimize jobs](#optimize-slow) for more details.

  - **Cloud Storage** : To import, you must have a bucket containing your previously exported files. You don't need to set a size for your bucket.

  - **Dataflow** : Import jobs are subject to the same CPU, disk usage, and IP address [Compute Engine quotas](/dataflow/quotas#compute-engine-quotas) as other Dataflow jobs.

  - **Compute Engine** : Before running your import job, you must [set up initial quotas](https://support.google.com/cloud/answer/6075746) for Compute Engine, which Dataflow uses. These quotas represent the *maximum* number of resources that you allow Dataflow to use for your job. Recommended starting values are:
    
      - **CPUs** : 200
      - **In-use IP addresses** : 200
      - **Standard persistent disk** : 50 TB
    
    Generally, you don't have to make any other adjustments. Dataflow provides autoscaling so that you only pay for the actual resources used during the import. If your job can make use of more resources, the Dataflow UI displays a warning icon. The job should finish even if there is a warning icon.

### Required roles

To get the permissions that you need to export a database, ask your administrator to grant you the following IAM roles on your Dataflow worker service account:

  - [Cloud Spanner Viewer](/iam/docs/roles-permissions/spanner#spanner.viewer) ( `  roles/spanner.viewer  ` )
  - [Dataflow Worker](/iam/docs/roles-permissions/dataflow#dataflow.worker) ( `  roles/dataflow.worker  ` )
  - [Storage Admin](/iam/docs/roles-permissions/storage#storage.admin) ( `  roles/storage.admin  ` )
  - [Spanner Database Reader](/iam/docs/roles-permissions/spanner#spanner.databaseReader) ( `  roles/spanner.databaseReader  ` )
  - [Database Admin](/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `  roles/spanner.databaseAdmin  ` )

**Note:** The Spanner Database Admin role is only required for import jobs.

## Export data from a non-Spanner database to Avro files

The import process brings data in from Avro files located in a Cloud Storage bucket. You can export data in Avro format from any source and can use any available method to do so.

To export data from a non-Spanner database to Avro files, follow these steps:

Keep the following things in mind when exporting your data:

  - You can export using any of the Avro [primitive types](https://avro.apache.org/docs/current/spec.html#schema_primitive) as well as the [array](https://avro.apache.org/docs/current/spec.html#Arrays) complex type.

  - Each column in your Avro files must use one of the following column types:
    
      - `  ARRAY  `
      - `  BOOL  `
      - `  BYTES  ` <sup>\*</sup>
      - `  DOUBLE  `
      - `  FLOAT  `
      - `  INT  `
      - `  LONG  ` <sup>†</sup>
      - `  STRING  ` <sup>‡</sup>
    
    <sup>\*</sup> A column of type `  BYTES  ` is used to import a Spanner `  NUMERIC  ` ; see the following [recommended mappings](#recommended-map) section for details.
    
    <sup>†,‡</sup> You can import a `  LONG  ` storing a timestamp or a `  STRING  ` storing a timestamp as a Spanner `  TIMESTAMP  ` ; see the following [recommended mappings](#recommended-map) section for details.

  - You don't have to include or generate any metadata when you export the Avro files.

  - You don't have to follow any particular naming convention for your files.

If you don't export your files directly to Cloud Storage, you must upload the Avro files to a Cloud Storage bucket. For detailed instructions, see [Upload objects](/storage/docs/uploading-objects) to your Cloud Storage.

## Import Avro files from non-Spanner databases to Spanner

To import Avro files from a non-Spanner database to Spanner, follow these steps:

1.  Create target tables and define the schema for your Spanner database.
2.  Create a `  spanner-export.json  ` file in your Cloud Storage bucket.
3.  Run a Dataflow import job using gcloud CLI.

### Step 1: Create the schema for your Spanner database

Before you run your import, you must create the target table in Spanner and define its schema.

You must create a schema that uses the appropriate column type for each column in the Avro files.

#### Recommended mappings

### GoogleSQL

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Avro column type</th>
<th>Spanner column type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         ARRAY        </code></td>
<td><code dir="ltr" translate="no">         ARRAY        </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         BOOL        </code></td>
<td><code dir="ltr" translate="no">         BOOL        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         BYTES        </code></td>
<td><p><code dir="ltr" translate="no">          BYTES         </code></p>
<p><code dir="ltr" translate="no">          NUMERIC         </code> (when the column type is <code dir="ltr" translate="no">          BYTES         </code> and <code dir="ltr" translate="no">          logicalType=decimal         </code> , <code dir="ltr" translate="no">          precision=38         </code> , and <code dir="ltr" translate="no">          scale=9         </code> . If these exact specifications are omitted, the field is treated as a Spanner <code dir="ltr" translate="no">          BYTES         </code> value. For more information, see the <a href="https://avro.apache.org/docs/current/spec.html#Decimal">Avro decimal logical type</a> documentation.)</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         DOUBLE        </code></td>
<td><code dir="ltr" translate="no">         FLOAT64        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         FLOAT        </code></td>
<td><code dir="ltr" translate="no">         FLOAT64        </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         INT        </code></td>
<td><code dir="ltr" translate="no">         INT64        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         LONG        </code></td>
<td><p><code dir="ltr" translate="no">          INT64         </code></p>
<p><code dir="ltr" translate="no">          TIMESTAMP         </code> when <code dir="ltr" translate="no">          LONG         </code> represents a timestamp of the number of microseconds since 1970-01-01 00:00:00 UTC</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td><p><code dir="ltr" translate="no">          STRING         </code></p>
<p><code dir="ltr" translate="no">          TIMESTAMP         </code> when <code dir="ltr" translate="no">          STRING         </code> represents a timestamp in the <a href="/spanner/docs/reference/standard-sql/data-types#canonical-format_1">canonical format for SQL queries</a></p></td>
</tr>
</tbody>
</table>

### PostgreSQL

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Avro column type</th>
<th>Spanner column type</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         ARRAY        </code></td>
<td><code dir="ltr" translate="no">         ARRAY        </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         BOOL        </code></td>
<td><code dir="ltr" translate="no">         BOOLEAN        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         BYTES        </code></td>
<td><p><code dir="ltr" translate="no">          BYTEA         </code></p>
<p><code dir="ltr" translate="no">          NUMERIC         </code> (when the column type is <code dir="ltr" translate="no">          BYTEA         </code> and <code dir="ltr" translate="no">          logicalType=decimal         </code> , <code dir="ltr" translate="no">          precision=147455         </code> , and <code dir="ltr" translate="no">          scale=16383         </code> . If these exact specifications are omitted, the field is treated as a <code dir="ltr" translate="no">          BYTEA         </code> value. For more information, see the <a href="https://avro.apache.org/docs/current/spec.html#Decimal">Avro decimal logical type</a> documentation.)</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         DOUBLE        </code></td>
<td><code dir="ltr" translate="no">         DOUBLE PRECISION        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         FLOAT        </code></td>
<td><code dir="ltr" translate="no">         DOUBLE PRECISION        </code></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         INT        </code></td>
<td><code dir="ltr" translate="no">         BIGINT        </code></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">         LONG        </code></td>
<td><p><code dir="ltr" translate="no">          BIGINT         </code></p>
<p><code dir="ltr" translate="no">          TIMESTAMP         </code> when <code dir="ltr" translate="no">          LONG         </code> represents a timestamp of the number of microseconds since 1970-01-01 00:00:00 UTC</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">         STRING        </code></td>
<td><p><code dir="ltr" translate="no">          CHARACTER VARYING         </code></p>
<p><code dir="ltr" translate="no">          TIMESTAMP         </code> when <code dir="ltr" translate="no">          STRING         </code> represents a timestamp in the canonical format for SQL queries, for example '2022-05-28T07:08:21.123456789Z' or '2021-12-19T16:39:57-08:00'.</p></td>
</tr>
</tbody>
</table>

**Note:** If a column in your Avro data contains `  NULL  ` values, you must ensure that you make the corresponding column in your Spanner table nullable.

### Step 2: Create a spanner-export.json file

You must also create a file named `  spanner-export.json  ` in your Cloud Storage bucket. This file specifies the database dialect and contains a `  tables  ` array that lists the name and data file locations for each table.

The contents of the file have the following format:

``` text
{
  "tables": [
   {
    "name": "TABLE1",
    "dataFiles": [
      "RELATIVE/PATH/TO/TABLE1_FILE1",
      "RELATIVE/PATH/TO/TABLE1_FILE2"
    ]
   },
   {
    "name": "TABLE2",
    "dataFiles": ["RELATIVE/PATH/TO/TABLE2_FILE1"]
   }
  ],
  "dialect":"DATABASE_DIALECT"
}
```

Where DATABASE\_DIALECT = { `  GOOGLE_STANDARD_SQL  ` | `  POSTGRESQL  ` }

If the dialect element is omitted, the dialect defaults to `  GOOGLE_STANDARD_SQL  ` .

**Note:** Wildcards aren't supported; you must write out all filenames in full.

### Step 3: Run a Dataflow import job using gcloud CLI

To start your import job, follow the instructions for using the Google Cloud CLI to run a job with the [Avro to Spanner template](/dataflow/docs/guides/templates/provided-batch#gcsavrotocloudspanner) .

After you have started an import job, you can [see details about the job](/spanner/docs/import-non-spanner#view-dataflow-ui) in the Google Cloud console.

After the import job is finished, add any necessary [secondary indexes](/spanner/docs/secondary-indexes) and [foreign-keys](/spanner/docs/foreign-keys/overview) .

**Note:** To avoid [outbound data transfer charges](/storage/pricing#network-pricing) , choose a region that overlaps with your Cloud Storage bucket's location. For more information, see [Choose a region](/spanner/docs/import-non-spanner#choose-region) .

**Note:** The Dataflow job doesn't resolve interleaved tables or foreign key constraints, so you might face write errors due to referential integrity violations. Remove all interleaving relations or foreign keys before initiating the Dataflow job.

## Choose a region for your import job

You might want to choose a different region based on the location of your Cloud Storage bucket. To avoid [outbound data transfer charges](/storage/pricing#network-pricing) , choose a region that matches your Cloud Storage bucket's location.

  - If your Cloud Storage bucket location is a [region](/storage/docs/bucket-locations#location-r) , you can take advantage of [free network usage](/storage/pricing#network-buckets) by choosing the same region for your import job, assuming that region is available.

  - If your Cloud Storage bucket location is a [dual-region](/storage/docs/bucket-locations#location-dr) , you can take advantage of [free network usage](/storage/pricing#network-buckets) by choosing one of the two regions that make up the dual-region for your import job, assuming one of the regions is available.

<!-- end list -->

  - If a co-located region is not available for your import job, or if your Cloud Storage bucket location is a [multi-region](/storage/docs/bucket-locations#location-mr) , outbound data transfer charges apply. Refer to Cloud Storage [data transfer](/storage/pricing#network-pricing) pricing to choose a region that incurs the lowest data transfer charges.

## View or troubleshoot jobs in the Dataflow UI

After you start an import job, you can view details of the job, including logs, in the Dataflow section of the Google Cloud console.

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

### Troubleshoot failed import jobs

If you see the following errors in your job logs:

``` text
com.google.cloud.spanner.SpannerException: NOT_FOUND: Session not found

--or--

com.google.cloud.spanner.SpannerException: DEADLINE_EXCEEDED: Deadline expired before operation could complete.
```

Check the *99% Write latency* in the **Monitoring** tab of your Spanner database in the Google Cloud console. If it is showing high (multiple second) values, then it indicates that the instance is overloaded, causing writes to timeout and fail.

One cause of high latency is that the Dataflow job is running using too many workers, putting too much load on the Spanner instance.

To specify a limit on the number of Dataflow workers, instead of using the Import/Export tab in the instance details page of your Spanner database in the Google Cloud console, you must start the import using the Dataflow [Cloud Storage Avro to Spanner template](/dataflow/docs/guides/templates/provided-batch#gcs_avro_to_cloud_spanner) and specify the maximum number of workers as described:

### Console

If you are using the Dataflow console, the **Max workers** parameter is located in the **Optional parameters** section of the **Create job from template** page.

### gcloud

Run the [`  gcloud dataflow jobs run  `](/sdk/gcloud/reference/dataflow/jobs/run) command, and specify the `  max-workers  ` argument. For example:

``` text
  gcloud dataflow jobs run my-import-job \
    --gcs-location='gs://dataflow-templates/latest/GCS_Avro_to_Cloud_Spanner' \
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

## Optimize slow running import jobs

If you have followed the suggestions in [initial settings](#quota) , you should generally not have to make any other adjustments. If your job is running slowly, there are a few other optimizations you can try:

  - **Optimize the job and data location** : Run your Dataflow job [in the same region](#choose-region) where your Spanner instance and Cloud Storage bucket are located.

  - **Ensure sufficient Dataflow resources** : If the [relevant Compute Engine quotas](/dataflow/quotas#compute-engine-quotas) limit your Dataflow job's resources, the job's [Dataflow page](#dataflow-job-details) in the Google Cloud console displays a warning icon and log messages:
    
    In this situation, [increasing the quotas](https://support.google.com/cloud/answer/6075746) for CPUs, in-use IP addresses, and standard persistent disk might shorten the run time of the job, but you might incur more Compute Engine charges.

  - **Check the Spanner CPU utilization** : If you see that the CPU utilization for the instance is over 65%, you can increase the [compute capacity](/spanner/docs/compute-capacity) in that instance. The capacity adds more Spanner resources and the job should speed up, but you incur more Spanner charges.

## Factors affecting import job performance

Several factors influence the time it takes to complete an import job.

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

  - **Existing load on Spanner** : An import job adds significant CPU load on a Spanner instance. If the instance already has a substantial existing load, then the job runs more slowly.

  - **Amount of Spanner compute capacity** : If the CPU utilization for the instance is over 65%, then the job runs more slowly.

## Tune workers for good import performance

When starting a Spanner import job, Dataflow workers must be set to an optimal value for good performance. Too many workers overloads Spanner and too few workers results in an underwhelming import performance.

The maximum number of workers is heavily dependent on the data size, but ideally, the total Spanner CPU utilization should be between 70% to 90%. This provides a good balance between Spanner efficiency and error-free job completion.

To achieve that utilization target in the majority of schemas and scenarios, we recommend a max number of worker vCPUs between 4-6x the number of Spanner nodes.

For example, for a 10 node Spanner instance, using n1-standard-2 workers, you would set max workers to 25, giving 50 vCPUs.
