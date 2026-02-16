This page describes how to import Spanner databases into Spanner using the Google Cloud console. To import Avro files from another source, see [Import data from non-Spanner databases](/spanner/docs/import-non-spanner) .

The process uses [Dataflow](/dataflow) ; it imports data from a [Cloud Storage](/storage) bucket folder that contains a set of [Avro files](https://en.wikipedia.org/wiki/Apache_Avro) and JSON manifest files. The import process supports only Avro files exported from Spanner.

To import a Spanner database using the REST API or the `  gcloud CLI  ` , complete the steps in the [Before you begin](#before-you-begin) section on this page, then see the detailed instructions in [Cloud Storage Avro to Spanner](/dataflow/docs/guides/templates/provided-batch#gcsavrotocloudspanner) .

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

## Optional: Find your database folder in Cloud Storage

To find the folder that contains your exported database in the Google Cloud console, navigate to the Cloud Storage browser and click on the bucket that contains the exported folder.

The name of the folder that contains your exported data begins with your instance's ID, database name, and the timestamp of your export job. The folder contains:

  - A `  spanner-export.json  ` file.

  - A `  TableName -manifest.json  ` file for each table in the database you exported.

  - One or more `  TableName .avro- ##### -of- #####  ` files. The first number in the extension `  .avro- ##### -of- #####  ` represents the index of the Avro file, starting at zero, and the second represents the number of Avro files generated for each table.
    
    For example, `  Songs.avro-00001-of-00002  ` is the second of two files that contain the data for the `  Songs  ` table.

  - A `  ChangeStreamName -manifest.json  ` file for each [change stream](/spanner/docs/change-streams) in the database you exported.

  - A `  ChangeStreamName .avro-00000-of-00001  ` file for each change stream. This file contains empty data with only the Avro schema of the change stream.

## Import a database

To import your Spanner database from Cloud Storage to your instance, follow these steps.

1.  Go to the Spanner **Instances** page.

2.  Click the name of the instance that will contain the imported database.

3.  Click the **Import/Export** menu item in the left pane and then click the **Import** button.

4.  Under **Choose a source folder** , click **Browse** .

5.  Find the bucket that contains your export in the initial list, or click **Search** to filter the list and find the bucket. Double-click the bucket to see the folders it contains.

6.  Find the folder with your exported files and click to select it.
    
    **Note:** Be sure to select the folder created by the export job and not a higher-level folder that contains the exported folder.

7.  Click **Select** .

8.  Enter a name for the new database, which Spanner creates during the import process. The database name cannot already exist in your instance.

9.  Choose the dialect for the new database (GoogleSQL or PostgreSQL).

10. (Optional) To protect the new database with a [customer-managed encryption key](/spanner/docs/cmek) , click **Show encryption options** and select **Use a customer-managed encryption key (CMEK)** . Then, select a key from the drop-down list.

11. Select a region in the **Choose a region for the import job** drop-down menu.
    
    **Note:** To avoid [outbound data transfer charges](/storage/pricing#network-pricing) , choose a region that overlaps with your Cloud Storage bucket's location. See [Choose a region](#choose-region) below for more information.

12. (Optional) To [encrypt the Dataflow pipeline state](/dataflow/docs/guides/customer-managed-encryption-keys) with a customer-managed encryption key, click **Show encryption options** and select **Use a customer-managed encryption key (CMEK)** . Then, select a key from the drop-down list.

13. Select the checkbox under **Confirm charges** to acknowledge that there are charges in addition to those incurred by your existing Spanner instance.

14. Click **Import** .
    
    The Google Cloud console displays the **Database details** page, which now shows a box describing your import job, including the job's elapsed time:

When the job finishes or terminates, the Google Cloud console displays a message on the **Database details** page. If the job succeeds, a success message appears:

**Note:** After the Dataflow import job successfully finishes, Spanner creates indexes and foreign keys for your imported database. While index creation is in progress, the Google Cloud console shows an in- progress icon to indicate that a long-running operation is occurring. The icon is next to the index's name, in the database hierarchy to the left of the **Database details** page. When the in-progress icon changes to the index icon , creation of that index is complete. You can query `  SPANNER_STATE  ` on the [INFORMATION\_SCHEMA.REFERENTIAL\_CONSTRAINTS](/spanner/docs/information-schema#information_schemareferential_constraints) view to see the creation progress of foreign keys.

If the job does not succeed, a failure message appears:

If your job fails, [check the job's Dataflow logs](#dataflow-job-logs) for error details and see [Troubleshoot failed import jobs](#failing-import) .

### A note on importing generated columns and change streams

Spanner uses the definition of each [generated column](/spanner/docs/generated-column/how-to) in the Avro schema to recreate that column. Spanner computes generated column values automatically during import.

Similarly, Spanner uses the definition of each [change stream](/spanner/docs/change-streams) in the Avro schema to recreate it during import. Change stream data is neither exported nor imported through Avro, so all change streams associated with a freshly imported database will have no change data records.

### A note on importing sequences

Each sequence ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#create-sequence) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#create_sequence) ) that Spanner exports uses the `  GET_INTERNAL_SEQUENCE_STATE()  ` ( [GoogleSQL](/spanner/docs/reference/standard-sql/sequence_functions#get_internal_sequence_state) , [PostgreSQL](/spanner/docs/reference/postgresql/functions-and-operators#sequence) ) function to capture its current state. Spanner adds a buffer of 1000 to the counter, and writes the new counter value to the properties of the record field. Note that this is only a best effort approach to avoid duplicate value errors that might happen after import. Adjust the actual sequence counter if there are more writes to the source database during data export.

At import, the sequence starts from this new counter instead of the counter found in the schema. If you need to, you can use the ALTER SEQUENCE ( [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#alter-sequence) , [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#alter_sequence) ) statement to update to a new counter.

### A note on importing interleaved tables and foreign keys

The Dataflow job can import interleaved tables, letting you to retain parent-child relationships from your source file. However, foreign key constraints aren't enforced during the data load. The Dataflow job creates all the necessary foreign keys after the data load is completed.

If you have foreign key constraints on the Spanner database before the import starts, you might encounter write errors due to referential integrity violations. To avoid write errors, consider dropping any existing foreign keys before initiating the import process.

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

Note that index and foreign key creation continues after the Dataflow import job completes. Change streams are created before the import job completes, but after all the data is imported.

  - **Data location** : Data is transferred between Spanner and Cloud Storage using Dataflow. Ideally all three components are located in the same region. If the components are not in the same region, moving the data across regions slows the job down.

  - **Number of Dataflow workers** : Optimal Dataflow workers are necessary for good performance. By using autoscaling, Dataflow chooses the number of workers for the job depending on the amount of work that needs to be done. The number of workers will, however, be capped by the quotas for CPUs, in-use IP addresses, and standard persistent disk. The Dataflow UI displays a warning icon if it encounters quota caps. In this situation, progress is slower, but the job should still complete. Autoscaling can overload Spanner leading to errors when there is a large amount of data to import.

  - **Existing load on Spanner** : An import job adds significant CPU load on a Spanner instance. If the instance already has a substantial existing load, then the job runs more slowly.

  - **Amount of Spanner compute capacity** : If the CPU utilization for the instance is over 65%, then the job runs more slowly.

## Tune workers for good import performance

When starting a Spanner import job, Dataflow workers must be set to an optimal value for good performance. Too many workers overloads Spanner and too few workers results in an underwhelming import performance.

The maximum number of workers is heavily dependent on the data size, but ideally, the total Spanner CPU utilization should be between 70% to 90%. This provides a good balance between Spanner efficiency and error-free job completion.

To achieve that utilization target in the majority of schemas and scenarios, we recommend a max number of worker vCPUs between 4-6x the number of Spanner nodes.

For example, for a 10 node Spanner instance, using n1-standard-2 workers, you would set max workers to 25, giving 50 vCPUs.
