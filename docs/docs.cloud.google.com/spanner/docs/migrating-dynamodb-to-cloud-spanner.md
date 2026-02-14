This tutorial describes how to migrate from Amazon DynamoDB to [Spanner](/spanner) . It is primarily intended for app owners who want to move from a NoSQL system to Spanner, a fully relational, fault-tolerant, highly scalable SQL database system that supports transactions. If you have consistent Amazon DynamoDB table usage, in terms of types and layout, mapping to Spanner is straightforward. If your Amazon DynamoDB tables contain arbitrary data types and values, it might be simpler to move to other NoSQL services, such as [Datastore](/datastore) or [Firestore](https://firebase.google.com/docs/firestore) .

This tutorial assumes that you are familiar with database schemas, data types, the fundamentals of NoSQL, and relational database systems. The tutorial relies on running predefined tasks to perform an example migration. After the tutorial, you can modify the provided code and steps to [match your environment](#adjust-data-model) .

The following architectural diagram outlines the components used in the tutorial to migrate data:

## Objectives

  - Migrate data from Amazon DynamoDB to Spanner.
  - Create a Spanner database and migration table.
  - Map a NoSQL schema to a relational schema.
  - Create and export a sample dataset that uses Amazon DynamoDB.
  - Transfer data between Amazon S3 and Cloud Storage.
  - Use Dataflow to load data into Spanner.

## Costs

This tutorial uses the following billable components of Google Cloud:

  - [Pub/Sub](/pubsub/docs/overview)
  - [Cloud Storage](/storage/docs/introduction)
  - [Dataflow](https://cloud.google.com/dataflow)

Spanner charges are based on the amount of compute capacity in your instance and the amount of data stored during the monthly billing cycle. During the tutorial, you use a minimal configuration of these resources, which are [cleaned up at the end](#clean-up) . For real-world scenarios, estimate your throughput and storage requirements, and then use the [Spanner instances documentation](/spanner/docs/instances) to determine the amount of compute capacity that you need.

In addition to Google Cloud resources, this tutorial uses the following Amazon Web Services (AWS) resources:

  - AWS Lambda
  - Amazon S3
  - Amazon DynamoDB

These services are only needed during the migration process. At the end of the tutorial, follow the instructions to clean up all resources to prevent unnecessary charges. Use the [AWS pricing calculator](https://calculator.s3.amazonaws.com/index.html) to estimate these costs.

To generate a cost estimate based on your projected usage, use the [pricing calculator](https://cloud.google.com/products/calculator) .

## Before you begin

1.  Sign in to your Google Cloud account. If you're new to Google Cloud, [create an account](https://console.cloud.google.com/freetrial) to evaluate how our products perform in real-world scenarios. New customers also get $300 in free credits to run, test, and deploy workloads.

2.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

3.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

4.  Enable the Spanner, Pub/Sub, Compute Engine, and Dataflow APIs.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

5.  In the Google Cloud console, on the project selector page, select or create a Google Cloud project.
    
    **Roles required to select or create a project**
    
      - **Select a project** : Selecting a project doesn't require a specific IAM role—you can select any project that you've been granted a role on.
      - **Create a project** : To create a project, you need the Project Creator role ( `  roles/resourcemanager.projectCreator  ` ), which contains the `  resourcemanager.projects.create  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .
    
    **Note** : If you don't plan to keep the resources that you create in this procedure, create a project instead of selecting an existing project. After you finish these steps, you can delete the project, removing all resources associated with the project.

6.  [Verify that billing is enabled for your Google Cloud project](/billing/docs/how-to/verify-billing-enabled#confirm_billing_is_enabled_on_a_project) .

7.  Enable the Spanner, Pub/Sub, Compute Engine, and Dataflow APIs.
    
    **Roles required to enable APIs**
    
    To enable APIs, you need the Service Usage Admin IAM role ( `  roles/serviceusage.serviceUsageAdmin  ` ), which contains the `  serviceusage.services.enable  ` permission. [Learn how to grant roles](/iam/docs/granting-changing-revoking-access) .

When you finish the tasks that are described in this document, you can avoid continued billing by deleting the resources that you created. For more information, see [Clean up](#clean-up) .

## Prepare your environment

In this tutorial, you run commands in Cloud Shell. Cloud Shell gives you access to the command line in Google Cloud, and includes the Google Cloud CLI and other tools that you need for Google Cloud development. Cloud Shell can take several minutes to initialize.

1.  In the Google Cloud console, activate Cloud Shell.
    
    At the bottom of the Google Cloud console, a [Cloud Shell](/shell/docs/how-cloud-shell-works) session starts and displays a command-line prompt. Cloud Shell is a shell environment with the Google Cloud CLI already installed and with values already set for your current project. It can take a few seconds for the session to initialize.

2.  Set the default Compute Engine zone. For example, `  us-central1-b  ` . gcloud config set compute/zone us-central1-b

3.  Clone the GitHub repository containing the sample code. git clone https://github.com/GoogleCloudPlatform/dynamodb-spanner-migration.git

4.  Go to the cloned directory. cd dynamodb-spanner-migration

5.  Create a Python [virtual environment](https://virtualenv.pypa.io/en/stable/) . pip3 install virtualenv virtualenv env

6.  Activate the virtual environment. source env/bin/activate

7.  Install the required Python modules. pip3 install -r requirements.txt

## Configure AWS access

In this tutorial, you create and delete Amazon DynamoDB tables, Amazon S3 buckets, and other resources. To access these resources, you first need to create the required AWS Identity and Access Management (IAM) permissions. You can use a test or sandbox AWS account to avoid affecting production resources in the same account.

### Create an AWS IAM role for AWS Lambda

In this section, you create an AWS IAM role that AWS Lambda uses at a later step in the tutorial.

1.  In the AWS console, go to the **IAM** section, click **Roles** , and then select **Create role** .
2.  Under **Trusted entity type** , ensure that **AWS service** is selected.
3.  Under **Use case** , select **Lambda** , and then click **Next** .
4.  In the **Permission policies** filter box, enter `  AWSLambdaDynamoDBExecutionRole  ` and press `  Return  ` to search.
5.  Select the **AWSLambdaDynamoDBExecutionRole** checkbox, and then click **Next** .
6.  In the **Role name** box, enter `  dynamodb-spanner-lambda-role  ` , and then click **Create role** .

### Create an AWS IAM user

Follow these steps to create an AWS IAM user with programmatic access to AWS resources, which are used throughout the tutorial.

1.  While you are still in the **IAM** section of the AWS console, click **Users** , and then select **Add users** .

2.  In the **User name** box, enter `  dynamodb-spanner-migration  ` .

3.  Under **Access type** , select the checkbox to the left of **Access key - Programmatic access** .
    
    **Note:** Do not enable management console access, because you will not use this IAM user to sign in to the console.

4.  Click **Next: Permissions** .

5.  Click **Attach existing policies directly** , and using the **Search** box to filter, select the checkbox next to each of the following three policies:
    
      - `  AmazonDynamoDBFullAccess  `
      - `  AmazonS3FullAccess  `
      - `  AWSLambda_FullAccess  `

6.  Click **Next: Tags** and **Next: Review** , and then click **Create user** .

7.  Click **Show** to view the credentials. The access key ID and secret access key are displayed for the newly created user. Leave this window open for now because the credentials are needed in the following section. Safely store these credentials because with them, you can make changes to your account and affect your environment. At the end of this tutorial, you can [delete the IAM user](#delete_aws_resources) .

### Configure AWS command-line interface

1.  In Cloud Shell, configure the AWS Command Line Interface (CLI).
    
    ``` text
    aws configure
    ```
    
    The following output appears:
    
    ``` console
    AWS Access Key ID [None]: PASTE_YOUR_ACCESS_KEY_ID
    AWS Secret Access Key [None]: PASTE_YOUR_SECRET_ACCESS_KEY
    Default region name [None]: us-west-2
    Default output format [None]:
    ```
    
      - Enter the `  ACCESS KEY ID  ` and `  SECRET ACCESS KEY  ` from the AWS IAM account that you created.
      - In the **Default region name** field, enter `  us-west-2  ` . Leave other fields at their default values.

2.  Close the AWS IAM console window.

## Understand the data model

The following section outlines the similarities and differences between data types, keys, and indexes for Amazon DynamoDB and Spanner.

### Data types

Spanner uses GoogleSQL data types. The following table describes how Amazon DynamoDB [data types](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.NamingRulesDataTypes.html#HowItWorks.DataTypes) map to Spanner [data types](/spanner/docs/data-types) .

<table>
<thead>
<tr class="header">
<th><strong>Amazon DynamoDB</strong></th>
<th><strong>Spanner</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Number</td>
<td>Depending on precision or intended usage, might be mapped to INT64, FLOAT64, TIMESTAMP, or DATE.</td>
</tr>
<tr class="even">
<td>String</td>
<td>String</td>
</tr>
<tr class="odd">
<td>Boolean</td>
<td>BOOL</td>
</tr>
<tr class="even">
<td>Null</td>
<td>No explicit type. Columns can contain null values.</td>
</tr>
<tr class="odd">
<td>Binary</td>
<td>Bytes</td>
</tr>
<tr class="even">
<td>Sets</td>
<td>Array</td>
</tr>
<tr class="odd">
<td>Map and List</td>
<td>Struct if the structure is consistent and can be described by using table DDL syntax.</td>
</tr>
</tbody>
</table>

### Primary key

An Amazon DynamoDB primary key establishes uniqueness and can be either a hash key or a combination of a hash key plus a range key. This tutorial starts by demonstrating the migration of a Amazon DynamoDB table whose primary key is a hash key. This hash key becomes the primary key of your Spanner table. Later, in the [section on interleaved tables](#interleaved_indexes) , you model a situation where an Amazon DynamoDB table uses a primary key composed of a hash key and a range key.

### Secondary indexes

Both Amazon DynamoDB and Spanner support creating an index on a non-primary key attribute. Take note of any secondary indexes in your Amazon DynamoDB table so that you can create them on your Spanner table, which is covered in a [later section of this tutorial](#apply_secondary_indexes) .

### Sample table

To facilitate this tutorial, you migrate the following sample table from Amazon DynamoDB to Spanner:

<table>
<thead>
<tr class="header">
<th></th>
<th>Amazon DynamoDB</th>
<th>Spanner</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Table name</td>
<td><code dir="ltr" translate="no">       Migration      </code></td>
<td><code dir="ltr" translate="no">       Migration      </code></td>
</tr>
<tr class="even">
<td>Primary key</td>
<td><code dir="ltr" translate="no">       "Username" : String      </code></td>
<td><code dir="ltr" translate="no">       "Username" : STRING(1024)      </code></td>
</tr>
<tr class="odd">
<td>Key type</td>
<td>Hash</td>
<td>n/a</td>
</tr>
<tr class="even">
<td>Other fields</td>
<td><code dir="ltr" translate="no">       Zipcode: Number Subscribed: Boolean ReminderDate: String PointsEarned: Number      </code></td>
<td><code dir="ltr" translate="no">       Zipcode: INT64 Subscribed: BOOL ReminderDate: DATE PointsEarned: INT64      </code></td>
</tr>
</tbody>
</table>

## Prepare the Amazon DynamoDB table

In the following section, you create an Amazon DynamoDB source table and populate it with data.

1.  In Cloud Shell, create an Amazon DynamoDB table that uses the [sample table](#sample_table) attributes.
    
    ``` text
    aws dynamodb create-table --table-name Migration \
        --attribute-definitions AttributeName=Username,AttributeType=S \
        --key-schema AttributeName=Username,KeyType=HASH \
        --provisioned-throughput ReadCapacityUnits=75,WriteCapacityUnits=75
    ```

2.  Verify that the table status is `  ACTIVE  ` .
    
    ``` text
    aws dynamodb describe-table --table-name Migration \
        --query 'Table.TableStatus'
    ```

3.  Populate the table with sample data.
    
    ``` text
    python3 make-fake-data.py --table Migration --items 25000
    ```
    
    **Note:** If slightly fewer than the number of requested records are created, you can ignore the discrepancy. Fewer records might be created if duplicate sample usernames were generated.

## Create a Spanner database

You create a Spanner instance with the smallest possible compute capacity: 100 processing units. This compute capacity is sufficient for the scope of this tutorial. For a production deployment, refer to the documentation for [Spanner instances](/spanner/docs/instances#node_count) to determine the appropriate compute capacity to meet your database performance requirements.

In this example, you create a table schema at the same time as the database. It is also possible, and common, to carry out [schema updates](/spanner/docs/gcloud-spanner#update_databases) after you create the database.

**Note:** Both Amazon DynamoDB and Spanner support the use of secondary indexes. The best practices for [bulk loading](/spanner/docs/bulk-loading) Spanner recommend that you create secondary indexes *after* you load your initial data. You [set up a secondary index](#apply_secondary_indexes) later in this tutorial.

1.  Create a Spanner instance in the same region where you set the default Compute Engine zone. For example, `  us-central1  ` .
    
    ``` text
    gcloud beta spanner instances create spanner-migration \
        --config=regional-us-central1 --processing-units=100 \
        --description="Migration Demo"
    ```

2.  Create a database in the Spanner instance along with the sample table.
    
    ``` text
    gcloud spanner databases create migrationdb \
        --instance=spanner-migration \
        --ddl "CREATE TABLE Migration ( \
                Username STRING(1024) NOT NULL, \
                PointsEarned INT64, \
                ReminderDate DATE, \
                Subscribed BOOL, \
                Zipcode INT64, \
             ) PRIMARY KEY (Username)"
    ```

## Prepare the migration

The next sections show you how to export the Amazon DynamoDB source table and set Pub/Sub replication to capture any changes to the database that occur while you export it.

**Note:** If changes to your database are not [idempotent](https://wikipedia.org/wiki/Idempotence) and it is not safe to write the same data more than once, it is best to carry out the following steps during a maintenance period when you can pause app changes to the table.

### Stream changes to Pub/Sub

You use an AWS Lambda function to stream database changes to Pub/Sub.

1.  In Cloud Shell, enable Amazon DynamoDB streams on your source table.
    
    ``` text
    aws dynamodb update-table --table-name Migration \
        --stream-specification StreamEnabled=true,StreamViewType=NEW_AND_OLD_IMAGES
    ```

2.  Set up a Pub/Sub topic to receive the changes.
    
    ``` text
    gcloud pubsub topics create spanner-migration
    ```
    
    The following output appears:
    
    ``` console
    Created topic [projects/your-project/topics/spanner-migration].
    ```

3.  Create an IAM service account to push table updates to the Pub/Sub topic.
    
    ``` text
    gcloud iam service-accounts create spanner-migration \
        --display-name="Spanner Migration"
    ```
    
    The following output appears:
    
    ``` console
    Created service account [spanner-migration].
    ```

4.  Create an IAM policy binding so that the service account has permission to publish to Pub/Sub. Replace `  GOOGLE_CLOUD_PROJECT  ` with the name of your Google Cloud project.
    
    ``` text
    gcloud projects add-iam-policy-binding GOOGLE_CLOUD_PROJECT \
        --role roles/pubsub.publisher \
        --member serviceAccount:spanner-migration@GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
    ```
    
    The following output appears:
    
    ``` console
    bindings:
    (...truncated...)
    - members:
      - serviceAccount:spanner-migration@solution-z.iam.gserviceaccount.com
      role: roles/pubsub.publisher
    ```

5.  Create credentials for the service account.
    
    ``` text
    gcloud iam service-accounts keys create credentials.json \
        --iam-account spanner-migration@GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
    ```
    
    The following output appears:
    
    ``` console
    created key [5e559d9f6bd8293da31b472d85a233a3fd9b381c] of type [json] as [credentials.json] for [spanner-migration@your-project.iam.gserviceaccount.com]
    ```

6.  Prepare and package the AWS Lambda function to push Amazon DynamoDB table changes to the Pub/Sub topic.
    
    ``` text
    pip3 install --ignore-installed --target=lambda-deps google-cloud-pubsub
    
    cd lambda-deps; zip -r9 ../pubsub-lambda.zip *; cd -
    
    zip -g pubsub-lambda.zip ddbpubsub.py
    ```

7.  Create a variable to capture the Amazon Resource Name (ARN) of the Lambda execution role that you created earlier.
    
    ``` text
    LAMBDA_ROLE=$(aws iam list-roles \
        --query 'Roles[?RoleName==`dynamodb-spanner-lambda-role`].[Arn]' \
        --output text)
    ```

8.  Use the `  pubsub-lambda.zip  ` package to create the AWS Lambda function.
    
    ``` text
    aws lambda create-function --function-name dynamodb-spanner-lambda \
        --runtime python3.9 --role ${LAMBDA_ROLE} \
        --handler ddbpubsub.lambda_handler --zip fileb://pubsub-lambda.zip \
        --environment Variables="{SVCACCT=$(base64 -w 0 credentials.json),PROJECT=GOOGLE_CLOUD_PROJECT,TOPIC=spanner-migration}"
    ```
    
    The following output appears:
    
    ``` console
    {
        "FunctionName": "dynamodb-spanner-lambda",
        "LastModified": "2022-03-17T23:45:26.445+0000",
        "RevisionId": "e58e8408-cd3a-4155-a184-4efc0da80bfb",
        "MemorySize": 128,
    
    ... truncated output...
        "PackageType": "Zip",
        "Architectures": [
            "x86_64"
        ]
    }
    ```

9.  Create a variable to capture the ARN of the Amazon DynamoDB stream for your table.
    
    ``` text
    STREAMARN=$(aws dynamodb describe-table \
        --table-name Migration \
        --query "Table.LatestStreamArn" \
        --output text)
    ```

10. Attach the Lambda function to the Amazon DynamoDB table.
    
    ``` text
    aws lambda create-event-source-mapping --event-source ${STREAMARN} \
        --function-name dynamodb-spanner-lambda --enabled \
        --starting-position TRIM_HORIZON
    ```

11. To optimize responsiveness during testing, add `  --batch-size 1  ` to the end of the previous command, which triggers the function every time you create, update, or delete an item.
    
    You will see output similar to the following:
    
    ``` console
    {
        "UUID": "44e4c2bf-493a-4ba2-9859-cde0ae5c5e92",
        "StateTransitionReason": "User action",
        "LastModified": 1530662205.549,
        "BatchSize": 100,
        "EventSourceArn": "arn:aws:dynamodb:us-west-2:accountid:table/Migration/stream/2018-07-03T15:09:57.725",
        "FunctionArn": "arn:aws:lambda:us-west-2:accountid:function:dynamodb-spanner-lambda",
        "State": "Creating",
        "LastProcessingResult": "No records processed"
    ... truncated output...
    ```

### Export the Amazon DynamoDB table to Amazon S3

1.  In Cloud Shell, create a variable for a bucket name that you use in several of the following sections.
    
    ``` text
    BUCKET=${DEVSHELL_PROJECT_ID}-dynamodb-spanner-export
    ```

2.  Create an Amazon S3 bucket to receive the DynamoDB export.
    
    ``` text
    aws s3 mb s3://${BUCKET}
    ```

3.  In the AWS Management Console, go to **DynamoDB** , and click **Tables** .

4.  Click the `  Migration  ` table.

5.  Under the **Exports and stream** tab, click **Export to S3** .

6.  Enable `  point-in-time-recovery  ` (PITR) if prompted.

7.  Click **Browse S3** to choose the S3 bucket you created earlier.

8.  Click **Export** .

9.  Click the **Refresh** *icon* to update the status of the export job. The job takes several minutes to finish exporting.
    
    When the process finishes, look at the output bucket.
    
    ``` text
    aws s3 ls --recursive s3://${BUCKET}
    ```
    
    Expect this step to take about 5 minutes. After it is complete, you see output like the following:
    
    ``` console
    2022-02-17 04:41:46          0 AWSDynamoDB/01645072900758-ee1232a3/_started
    2022-02-17 04:46:04     500441 AWSDynamoDB/01645072900758-ee1232a3/data/xygt7i2gje4w7jtdw5652s43pa.json.gz
    2022-02-17 04:46:17        199 AWSDynamoDB/01645072900758-ee1232a3/manifest-files.json
    2022-02-17 04:46:17         24 AWSDynamoDB/01645072900758-ee1232a3/manifest-files.md5
    2022-02-17 04:46:17        639 AWSDynamoDB/01645072900758-ee1232a3/manifest-summary.json
    2022-02-17 04:46:18         24 AWSDynamoDB/01645072900758-ee1232a3/manifest-summary.md5
    ```

## Perform the migration

Now that the Pub/Sub delivery is in place, you can push forward any table changes that occurred after the export.

**Note:** If you paused writes to the database before exporting, it is time to reactivate writes to the database.

### Copy the exported table to Cloud Storage

1.  In Cloud Shell, create a Cloud Storage bucket to receive the exported files from Amazon S3.
    
    ``` text
    gcloud storage buckets create gs://${BUCKET}
    ```

2.  [Sync](/sdk/gcloud/reference/storage/rsync) the files from Amazon S3 into Cloud Storage. For most copy operations, the `  rsync  ` command is effective. If your export files are large (several GBs or more), use the [Cloud Storage transfer service](/storage-transfer/docs/overview) to manage the transfer in the background.
    
    ``` text
    gcloud storage rsync s3://${BUCKET} gs://${BUCKET} --recursive --delete-unmatched-destination-objects
    ```

### Batch import the data

1.  To write the data from the exported files into the Spanner table, run a Dataflow job with sample Apache Beam code.
    
    ``` text
    cd dataflow
    mvn compile
    mvn exec:java \
    -Dexec.mainClass=com.example.spanner_migration.SpannerBulkWrite \
    -Pdataflow-runner \
    -Dexec.args="--project=GOOGLE_CLOUD_PROJECT \
                 --instanceId=spanner-migration \
                 --databaseId=migrationdb \
                 --table=Migration \
                 --importBucket=$BUCKET \
                 --runner=DataflowRunner \
                 --region=us-central1"
    ```
    
    1.  To watch the progress of the import job, in the Google Cloud console, go to Dataflow.
    
    2.  While the job is running, you can watch the execution graph to examine the logs. Click the job that shows the **Status** of **Running** .

2.  Click each stage to see how many elements have been processed. The import is complete when all stages say **Succeeded** . The same number of elements that were created in your Amazon DynamoDB table display as processed at each stage.

3.  Verify that the number of records in the destination Spanner table matches the number of items in the Amazon DynamoDB table.
    
    ``` text
    aws dynamodb describe-table --table-name Migration --query Table.ItemCount
    
    gcloud spanner databases execute-sql migrationdb \
    --instance=spanner-migration --sql="select count(*) from Migration"
    ```
    
    **Note:** Since Amazon DynamoDB item count is based on table metadata that is only updated every six hours, output from this command may not immediately reflect the total number of items in the table. The Spanner item count is based on a table query and accurately reflects the number of rows in the table when you run the query.
    
    The following output appears:
    
    ``` console
    $ aws dynamodb describe-table --table-name Migration --query Table.ItemCount
    25000
    $ gcloud spanner databases execute-sql migrationdb --instance=spanner-migration --sql="select count(*) from Migration"
    25000
    ```

4.  Sample random entries in each table to make sure the data is consistent.
    
    ``` text
    gcloud spanner databases execute-sql migrationdb \
        --instance=spanner-migration \
        --sql="select * from Migration limit 1"
    ```
    
    The following output appears:
    
    ``` console
     Username: aadams4495
     PointsEarned: 5247
     ReminderDate: 2022-03-14
     Subscribed: True
     Zipcode: 58057
    ```

5.  Query the Amazon DynamoDB table with the same `  Username  ` that was returned from the Spanner query in the previous step. For example, `  aallen2538  ` . The value is specific to the sample data in your database.
    
    ``` text
    aws dynamodb get-item --table-name Migration \
        --key '{"Username": {"S": "aadams4495"}}'
    ```
    
    The values of the other fields should match those from the Spanner output. The following output appears:
    
    ``` console
    {
        "Item": {
            "Username": {
                "S": "aadams4495"
            },
            "ReminderDate": {
                "S": "2018-06-18"
            },
            "PointsEarned": {
                "N": "1606"
            },
            "Zipcode": {
                "N": "17303"
            },
            "Subscribed": {
                "BOOL": false
            }
        }
    }
    ```

### Replicate new changes

When the batch import job is complete, you set up a streaming job to write ongoing updates from the source table into Spanner. You [subscribe](/pubsub/docs/subscriber) to the events from Pub/Sub and write them to Spanner

The Lambda function you created is configured to capture changes to the source Amazon DynamoDB table and publish them to Pub/Sub.

1.  Create a subscription to the Pub/Sub topic that AWS Lambda sends events to.
    
    ``` text
    gcloud pubsub subscriptions create spanner-migration \
        --topic spanner-migration
    ```
    
    The following output appears:
    
    ``` console
    Created subscription [projects/your-project/subscriptions/spanner-migration].
    ```

2.  To stream the changes coming into Pub/Sub to write to the Spanner table, run the Dataflow job from Cloud Shell.
    
    ``` text
    mvn exec:java \
    -Dexec.mainClass=com.example.spanner_migration.SpannerStreamingWrite \
    -Pdataflow-runner \
    -Dexec.args="--project=GOOGLE_CLOUD_PROJECT \
                 --instanceId=spanner-migration \
                 --databaseId=migrationdb \
                 --table=Migration \
                 --experiments=allow_non_updatable_job \
    --subscription=projects/GOOGLE_CLOUD_PROJECT/subscriptions/spanner-migration \
    --runner=DataflowRunner \
    --region=us-central1"
    ```
    
    **Note:** To improve responsiveness during testing, you can set the [Windowing](https://beam.apache.org/documentation/programming-guide/#windowing) to `  5  ` seconds by adding `  --window 5  ` to the end of this command. The default is set to `  60  ` seconds.
    
    1.  Similar to the [batch load](#batch_import_the_data) step, to watch the progress of the job, in the Google Cloud console, go to Dataflow.
    
    2.  Click the job that has the **Status** of **Running** .
        
        The processing graph shows a similar output as before, but each processed item is counted in the status window. The system lag time is a rough estimate of how much delay to expect before changes appear in the Spanner table.

The Dataflow job that you ran in the batch loading phase was a finite set of input, also known as a *bounded* dataset. This Dataflow job uses Pub/Sub as a streaming source and is considered *unbounded* . For more information about these two types of sources, review the section on PCollections in the [Apache Beam programming guide](https://beam.apache.org/documentation/programming-guide/#pcollections) . The Dataflow job in this step is meant to stay active, so it does not terminate when finished. The streaming Dataflow job remains in the **Running** status, instead of the **Succeeded** status.

### Verify replication

You make some changes to the source table to verify that the changes are replicated to the Spanner table.

**Note:** The changes are asynchronous and might take several moments to appear in Spanner. Wait a few minutes before each step.

1.  Query a nonexistent row in Spanner.
    
    ``` text
    gcloud spanner databases execute-sql migrationdb \
        --instance=spanner-migration \
        --sql="SELECT * FROM Migration WHERE Username='my-test-username'"
    ```
    
    The operation will not return any results.

2.  Create a record in Amazon DynamoDB with the same key that you used in the Spanner query. If the command runs successfully, there is no output.
    
    ``` text
    aws dynamodb put-item \
        --table-name Migration \
        --item '{"Username" : {"S" : "my-test-username"}, "Subscribed" : {"BOOL" : false}}'
    ```

3.  Run that same query again to verify that the row is now in Spanner.
    
    ``` text
    gcloud spanner databases execute-sql migrationdb \
        --instance=spanner-migration \
        --sql="SELECT * FROM Migration WHERE Username='my-test-username'"
    ```
    
    The output shows the inserted row:
    
    ``` console
    Username: my-test-username
    PointsEarned: None
    ReminderDate: None
    Subscribed: False
    Zipcode:
    ```

4.  Change some attributes in the original item and update the Amazon DynamoDB table.
    
    ``` text
    aws dynamodb update-item \
        --table-name Migration \
        --key '{"Username": {"S":"my-test-username"}}' \
        --update-expression "SET PointsEarned = :pts, Subscribed = :sub" \
        --expression-attribute-values '{":pts": {"N":"4500"}, ":sub": {"BOOL":true}}'\
        --return-values ALL_NEW
    ```
    
    You will see output similar to the following:
    
    ``` console
    {
        "Attributes": {
            "Username": {
                "S": "my-test-username"
            },
            "PointsEarned": {
                "N": "4500"
            },
            "Subscribed": {
                "BOOL": true
            }
        }
    }
    ```

5.  Verify that the changes are propagated to the Spanner table.
    
    ``` text
    gcloud spanner databases execute-sql migrationdb \
        --instance=spanner-migration \
        --sql="SELECT * FROM Migration WHERE Username='my-test-username'"
    ```
    
    The output appears as follows:
    
    ``` console
    Username          PointsEarned  ReminderDate  Subscribed  Zipcode
    my-test-username  4500          None          True
    ```

6.  Delete the test item from the Amazon DynamoDB source table.
    
    ``` text
    aws dynamodb delete-item \
        --table-name Migration \
        --key '{"Username": {"S":"my-test-username"}}'
    ```

7.  Verify that the corresponding row is deleted from the Spanner table. When the change is propagated, the following command returns zero rows:
    
    ``` text
    gcloud spanner databases execute-sql migrationdb \
        --instance=spanner-migration \
        --sql="SELECT * FROM Migration WHERE Username='my-test-username'"
    ```

## Use interleaved tables

Spanner supports the concept of [interleaving tables](/spanner/docs/schema-and-data-model#creating_a_hierarchy_of_interleaved_tables) . This is a design model where a top-level item has several nested items that relate to that top-level item, such as a customer and their orders, or a player and their game scores. If your Amazon DynamoDB source table uses a primary key composed of a hash key and a range key, you can model an interleaved table schema as shown in the following diagram. This structure lets you efficiently query the interleaved table while joining fields in the parent table.

### Apply secondary indexes

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

It is a [best practice](/spanner/docs/bulk-loading#secondary-indexes) to apply secondary indexes to Spanner tables *after* you load the data. Now that replication is working, you set up a [secondary index](/spanner/docs/secondary-indexes) to speed up queries. Like Spanner tables, Spanner secondary indexes are fully consistent. They are *not* [eventually consistent](https://wikipedia.org/wiki/Eventual_consistency) , which is common in many NoSQL databases. This feature can help simplify your app design

Run a query that does not use any indexes. You are looking for the top *N* occurrences, given a particular column value. This is a common query in Amazon DynamoDB for database efficiency.

1.  Go to Spanner.

2.  Click **Spanner Studio** .

3.  In the **Query** field, enter the following query, and then click **Run query** .
    
    ``` text
    SELECT Username,PointsEarned
      FROM Migration
     WHERE Subscribed=true
       AND ReminderDate > DATE_SUB(DATE(current_timestamp()), INTERVAL 14 DAY)
     ORDER BY ReminderDate DESC
     LIMIT 10
    ```
    
    After the query runs, click **Explanation** and take note of the **Rows scanned** versus **Rows returned** . Without an index, Spanner scans the entire table to return a small subset of data that matches the query.

4.  If this represents a commonly occurring query, create a composite index on the Subscribed and ReminderDate columns. On the Spanner console, select **Indexes** left navigation pane, and then click **Create Index** .

5.  In the text box, enter the index definition.
    
    ``` text
    CREATE INDEX SubscribedDateDesc
    ON Migration (
      Subscribed,
      ReminderDate DESC
    )
    ```

6.  To begin building the database in the background, click **Create** .

7.  After the index is created, run the query again and add the index.
    
    ``` text
    SELECT Username,PointsEarned
      FROM Migration@{FORCE_INDEX=SubscribedDateDesc}
     WHERE Subscribed=true
       AND ReminderDate > DATE_SUB(DATE(current_timestamp()), INTERVAL 14 DAY)
     ORDER BY ReminderDate DESC
     LIMIT 10
    ```
    
    Examine the query explanation again. Notice that the number of **Rows scanned** has decreased. The **Rows returned** at each step matches the number returned by the query.

### Interleaved indexes

You can set up interleaved indexes in Spanner. The secondary indexes discussed in the [previous section](#apply_secondary_indexes) are at the root of the database hierarchy, and they use indexes the same way as a conventional database. An interleaved index is within the context of its interleaved row. See [index options](/spanner/docs/whitepapers/optimizing-schema-design#index_options) for more details about where to apply interleaved indexes.

## Adjust for your data model

In order to adapt the migration portion of this tutorial to your own situation, modify your Apache Beam source files. It is important that you do not change the source schema during the actual migration window, otherwise you can lose data.

1.  To parse incoming JSON and build mutations, use [GSON](https://github.com/google/gson/blob/master/UserGuide.md) . Adjust the JSON definition to match your data.
    
    ``` java
    public static class Record implements Serializable {
    
      private Item Item;
    
    }
    
    public static class Item implements Serializable {
    
      private Username Username;
      private PointsEarned PointsEarned;
      private Subscribed Subscribed;
      private ReminderDate ReminderDate;
      private Zipcode Zipcode;
    
    }
    
    public static class Username implements Serializable {
    
      private String S;
    
    }
    
    public static class PointsEarned implements Serializable {
    
      private String N;
    
    }
    
    public static class Subscribed implements Serializable {
    
      private String BOOL;
    
    }
    
    public static class ReminderDate implements Serializable {
    
      private String S;
    
    }
    
    public static class Zipcode implements Serializable {
    
      private String N;
    
    }
    ```

2.  Adjust the corresponding JSON mapping.
    
    ``` java
    mutation.set("Username").to(record.Item.Username.S);
    
    Optional.ofNullable(record.Item.Zipcode).ifPresent(x -> {
      mutation.set("Zipcode").to(Integer.parseInt(x.N));
    });
    
    Optional.ofNullable(record.Item.Subscribed).ifPresent(x -> {
      mutation.set("Subscribed").to(Boolean.parseBoolean(x.BOOL));
    });
    
    Optional.ofNullable(record.Item.ReminderDate).ifPresent(x -> {
      mutation.set("ReminderDate").to(Date.parseDate(x.S));
    });
    
    Optional.ofNullable(record.Item.PointsEarned).ifPresent(x -> {
      mutation.set("PointsEarned").to(Integer.parseInt(x.N));
    });
    ```

In the previous steps, you modified the Apache Beam source code for bulk import. Modify the source code for the streaming part of the pipeline in a similar manner. Finally, adjust the table creation scripts, schemas, and indexes of your Spanner target database.

## Clean up

To avoid incurring charges to your Google Cloud account for the resources used in this tutorial, either delete the project that contains the resources, or keep the project and delete the individual resources.

### Delete the project

**Caution** : Deleting a project has the following effects:

  - **Everything in the project is deleted.** If you used an existing project for the tasks in this document, when you delete it, you also delete any other work you've done in the project.
  - **Custom project IDs are lost.** When you created this project, you might have created a custom project ID that you want to use in the future. To preserve the URLs that use the project ID, such as an `  appspot.com  ` URL, delete selected resources inside the project instead of deleting the whole project.

In the Google Cloud console, go to the **Manage resources** page.

In the project list, select the project that you want to delete, and then click **Delete** .

In the dialog, type the project ID, and then click **Shut down** to delete the project.

### Delete AWS resources

If your AWS account is used outside of this tutorial, use caution when you delete the following resources:

1.  Delete the [DynamoDB table](https://console.aws.amazon.com/dynamodb/home?tables:#tables:) called **Migration** .
2.  Delete the [Amazon S3 bucket](https://s3.console.aws.amazon.com/s3/home?region=us-west-2) and [Lambda function](https://console.aws.amazon.com/lambda/home) that you created during the migration steps.
3.  Finally, delete the [AWS IAM](https://console.aws.amazon.com/iam/home#/users) user that you created during this tutorial.

## What's next

  - Read about how to [optimize your Spanner schema](/spanner/docs/whitepapers/optimizing-schema-design) .
  - Learn how to use [Dataflow](/dataflow/docs/how-to) for more complex situations.
