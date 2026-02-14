This page describes how to manage the lifecycle of a Spanner *long-running operation* using `  gcloud spanner operations  ` commands and the [operations REST API](/spanner/docs/reference/rest) . Some procedures can also be done in the Google Cloud console.

*Long-running operations* are method calls that might take a substantial amount of time to complete. Spanner creates long-running operations for several instance, database, and backup actions. An example is the method to restore a database, [`  projects.instances.databases.restore  `](/spanner/docs/reference/rest/v1/projects.instances.databases/restore#try-it) . When you restore a database, the Spanner service creates a long-running operation to track the restore progress. If the operation is taking longer than you expected, you can use `  gcloud  ` to check the progress of the operation. If the operation isn't responding, you can use `  gcloud  ` to cancel the operation.

Spanner provides operation APIs that let you check the progress of long-running operations. You can also list and cancel long-running operations, and delete long-running instance operations.

You can check and manage long-running operations with the following:

  - [Spanner client libraries](/spanner/docs/reference/libraries)
  - The [`  gcloud  `](/sdk/gcloud/reference/spanner) command-line tool
  - The [Google Cloud console](https://console.cloud.google.com/spanner)

## REST API commands for operation management

Manage your Spanner long-running operations using the following REST methods:

<table>
<thead>
<tr class="header">
<th>Action</th>
<th>Long-running database operations</th>
<th>Long-running instance operations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Cancel a long-running operation</td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases.operations/cancel#try-it"><code dir="ltr" translate="no">        cancel       </code></a></td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.operations/cancel#try-it"><code dir="ltr" translate="no">        cancel       </code></a></td>
</tr>
<tr class="even">
<td>Delete a long-running operation</td>
<td>Unsupported</td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.operations/delete#try-it"><code dir="ltr" translate="no">        delete       </code></a></td>
</tr>
<tr class="odd">
<td>Check the progress of a long-running operation</td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases.operations/get#try-it"><code dir="ltr" translate="no">        get       </code></a></td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.operations/get#try-it"><code dir="ltr" translate="no">        get       </code></a></td>
</tr>
<tr class="even">
<td>List long-running operations</td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.databases.operations/list#try-it"><code dir="ltr" translate="no">        list       </code></a></td>
<td><a href="/spanner/docs/reference/rest/v1/projects.instances.operations/list#try-it"><code dir="ltr" translate="no">        list       </code></a></td>
</tr>
</tbody>
</table>

For information about using REST with Spanner, see [Getting started with Spanner using REST](/spanner/docs/getting-started/rest#update_the_database_schema) .

## Instance operations

The following are long-running instance operations.

  - [`  projects.instances.create  `](/spanner/docs/reference/rest/v1/projects.instances/create#try-it)
  - [`  projects.instances.patch  `](/spanner/docs/reference/rest/v1/projects.instances/patch#try-it)

### Check the progress of a long-running instance operation

Use [`  projects.instances.operations.get  `](/spanner/docs/reference/rest/v1/projects.instances.operations/get#try-it) to check the progress of a long-running instance operation.

As an example, this is a response from [`  projects.instances.create  `](/spanner/docs/reference/rest/v1/projects.instances/create#try-it) :

``` text
  {
    "name": "projects/test01/instances/test-instance/operations/_auto_1492721321097206",
    "metadata": {
      "@type": "type.googleapis.com/google.spanner.admin.instance.v1.CreateInstanceMetadata",
      "instance": {
        "name": "projects/<VAR>PROJECT-ID</VAR>/instances/test-instance",
        "config": "projects/<VAR>PROJECT-ID</VAR>/instanceConfigs/regional-us-central1",
        "displayName": "Test Instance",
        "nodeCount": 1,
        "state": "READY"
      },
      "startTime": "2017-04-24T22:45:41.130854Z"
    }
  }
```

The `  name  ` value at the top of the response shows the Spanner service created a long-running instance operation named `  projects/test01/instances/test-instance/operations/_auto_1492721321097206  ` .

To Check the progress of the long-running instance operation:

1.  Navigate to [`  projects.instances.operations.get  `](/spanner/docs/reference/rest/v1/projects.instances.operations/get#try-it) .

2.  For **name** , enter the long-running instance operation name as shown in the response to `  projects.instances.create  ` or `  projects.instances.patch  ` . For example:
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/operations/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can retrieve the instance operation name by [listing long-running instance operations](#list_long-running_instance_operations) .

3.  Click **Execute** . When an operation is done, the `  done  ` field is set to `  true  ` .

To get continuous updates, repeatedly invoke the `  projects.instances.databases.operations.get  ` method until the operation is done. Use a backoff between each request. For example, make a request every 10 seconds.

### List long-running instance operations

Use [`  projects.instances.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.operations/list#try-it) to list long-running instance operations.

1.  Navigate to [`  projects.instances.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.operations/list#try-it) .

2.  For **name** , enter:
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/operationsOPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.

3.  Click **Execute** . The response contains a list of long-running operations.

### Cancel a long-running instance operation

Use [`  projects.instances.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.operations/cancel#try-it) to cancel a long-running instance operation.

1.  Navigate to [`  projects.instances.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.operations/cancel#try-it) .

2.  For **name** , enter the long-running instance operation name as shown in the long-running instance operation response.
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/operations/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the instance operation name by [listing long-running instance operations](#list_long-running_instance_operations) .

3.  Click **Execute** .

### Delete a long-running instance operation

Use [`  projects.instances.operations.delete  `](/spanner/docs/reference/rest/v1/projects.instances.operations/delete#try-it) to delete a long-running instance operation.

1.  Click [`  projects.instances.operations.delete  `](/spanner/docs/reference/rest/v1/projects.instances.operations/delete#try-it) .

2.  For **name** , enter the long-running instance operation name as shown in the long-running instance operation response.
    
    ``` text
    projects/<VAR>PROJECT-ID</VAR>/instances/<VAR>INSTANCE-NAME</VAR>/operations/<VAR>OPERATION-ID</VAR>
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the instance operation name by [listing long-running instance operations](#list_long-running_instance_operations) .

3.  Navigate to **Execute** . The operation is deleted.

## Database operations

The following are long-running database operations.

  - [`  projects.instances.databases.create  `](/spanner/docs/reference/rest/v1/projects.instances.databases/create#try-it)
  - [`  projects.instances.databases.restore  `](/spanner/docs/reference/rest/v1/projects.instances.databases/restore#try-it)
  - [`  projects.instances.databases.updateDdl  `](/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it)
  - [`  projects.instances.databaseOperations.list  `](/spanner/docs/reference/rest/v1/projects.instances.databaseOperations/list#try-it)

### Check the progress of a long-running database operation

Use [`  projects.instances.databases.operations.get  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/get#try-it) to check the progress of a long-running database operation.

For example, the following is a response from [`  projects.instances.databases.create  `](/spanner/docs/reference/rest/v1/projects.instances.databases/create#try-it) :

``` text
{
  "name": "projects/test01/instances/test-instance/databases/example-db/operations/_auto_1492721321097206",
  "metadata": {
    "@type": "type.googleapis.com/google.spanner.admin.database.v1.CreateDatabaseMetadata",
    "database": "projects/test01/instances/test-instance/databases/example-db"
  }
}
```

The `  name  ` value at the top of the response shows that the Spanner service created a long-running database operation called `  projects/test01/instances/test-instance/databases/example-db/operations/_auto_1492721321097206  ` .

To check the progress of the long-running database operation:

1.  Navigate to [`  projects.instances.databases.operations.get  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/get#try-it) .

2.  For **name** , enter the long-running database operation name as shown in the response to `  projects.instances.databases.create  ` or `  projects.instances.databases.updateDdl  ` .
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/databases/example-db/operations/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the database operation name by [listing long-running database operations](#list_long-running_database_operations) .

3.  Click **Execute** . When an operation is done, the `  done  ` field is set to `  true  ` .

To get continuous updates, repeatedly invoke the `  projects.instances.databases.operations.get  ` method until the operation is done. Use a backoff between each request. For example, make a request every 10 seconds.

### List long-running database operations

Use [`  projects.instances.databases.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/list#try-it) to list long-running database operations.

1.  Navigate to [`  projects.instances.databases.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/list#try-it) .

2.  For **name** , enter:
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/databases/example-db/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.

3.  Click **Execute** . The response contains a list of long-running operations.

### Cancel a long-running database operation

Use [`  projects.instances.databases.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/cancel#try-it) to cancel a long-running database operation.

1.  Navigate to [`  projects.instances.databases.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/cancel#try-it) .

2.  For **name** , enter the long-running database operation name as shown in the long-running database operation response.
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/databases/example-db/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the database operation name by [listing long-running database operations](#list_long-running_database_operations) .

3.  Click **Execute** .

## Schema update operations

The following are long-running schema update operations.

  - [projects.instances.databases.updateDdl](/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl)

### Check the progress of a long-running schema update operation

### Console

1.  In the Spanner navigation menu, select the **Operations** tab. The **Operations** page shows a list of running operations.

2.  Find the schema operation in the list. If it's still running, the progress bar in the **End time** column shows the percentage of the operation that is complete, as shown in the following image:

### gcloud

Use [`  gcloud spanner operations describe  `](/sdk/gcloud/reference/spanner/operations/describe) to check the progress of an operation.

1.  Get the operation ID:
    
    ``` text
    gcloud spanner operations list \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME \
    --type=DATABASE_UPDATE_DDL
    ```
    
    Replace the following:
    
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the instance name.
      - DATABASE-NAME : the name of the database.

2.  Run `  gcloud spanner operations describe  ` :
    
    ``` text
    gcloud spanner operations describe OPERATION-ID \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME
    ```
    
    Replace the following:
    
      - OPERATION-ID : the operation ID of the operation that you want to check.
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the Spanner database name.
    
    The `  progress  ` section in the output shows the percentage of the operation that's complete. The output looks similar to the following:
    
    ``` text
    done: true
    metadata:
    ...
      progress:
      - endTime: '2022-03-01T00:28:06.691403Z'
        progressPercent: 100
        startTime: '2022-03-01T00:28:04.221401Z'
      - endTime: '2022-03-01T00:28:17.624588Z'
        startTime: '2022-03-01T00:28:06.691403Z'
        progressPercent: 100
    ...
    ```

### REST v1

Get the operation ID:

``` text
gcloud spanner operations list \
--instance=INSTANCE-NAME \
--database=DATABASE-NAME \
--type=DATABASE_UPDATE_DDL
```

Replace the following:

  - INSTANCE-NAME : the Spanner instance name.
  - DATABASE-NAME : the name of the database.

Before using any of the request data, make the following replacements:

  - PROJECT-ID : the project ID.
  - INSTANCE-ID : the instance ID.
  - DATABASE-ID : the database ID.
  - OPERATION-ID : the operation ID.

HTTP method and URL:

``` text
GET https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X GET \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     "https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method GET `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
...
    "progress": [
      {
        "progressPercent": 100,
        "startTime": "2023-05-27T00:52:27.366688Z",
        "endTime": "2023-05-27T00:52:30.184845Z"
      },
      {
        "progressPercent": 100,
        "startTime": "2023-05-27T00:52:30.184845Z",
        "endTime": "2023-05-27T00:52:40.750959Z"
      }
    ],
...
  "done": true,
  "response": {
    "@type": "type.googleapis.com/google.protobuf.Empty"
  }
}
```

If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running schema update operation](#cancel_a_long-running_schema_update_operation) .

### List long-running schema update operations

### gcloud

``` text
gcloud spanner operations list \
 --instance=INSTANCE-NAME \
 --database=DATABASE-NAME \
 --type=DATABASE_UPDATE_DDL
```

Replace the following:

  - INSTANCE-NAME : the Spanner instance name.
  - DATABASE-NAME : the name of the database.

The output looks similar to the following:

``` text
OPERATION-ID     STATEMENTS                                                                                           DONE   @TYPE
_auto_op_123456  CREATE INDEX SingersByFirstLastName ON Singers(FirstName, LastName)                                  False  UpdateDatabaseDdlMetadata
_auto_op_234567  CREATE INDEX SongsBySingerAlbumSongName ON Songs(SingerId, AlbumId, SongName), INTERLEAVE IN Albums  True   CreateDatabaseMetadata
```

### Cancel a long-running schema update operation

### gcloud

1.  Get the operation ID:
    
    ``` text
    gcloud spanner operations list \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME \
    --type=DATABASE_UPDATE_DDL
    ```
    
    Replace the following:
    
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the name of the database.

2.  Use the [`  gcloud spanner operations cancel  `](/sdk/gcloud/reference/spanner/operations/cancel) to cancel a long-running schema update operation.
    
    ``` text
    gcloud spanner operations cancel OPERATION-ID \
     --instance=INSTANCE-NAME
    ```
    
    Replace the following:
    
      - OPERATION-ID : the operation ID of the operation that you want to check.
      - INSTANCE-NAME : the Spanner instance name.

### REST V1

Use [`  projects.instances.databases.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/cancel#try-it) to cancel a long-running schema update operation.

1.  Get the operation ID:
    
    ``` text
    gcloud spanner operations list \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME \
    --type=DATABASE_UPDATE_DDL
    ```
    
    Replace the following:
    
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the name of the database.

2.  Navigate to [`  projects.instances.databases.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.databases.operations/cancel#try-it) .

3.  For **name** , enter the long-running schema update operation name as shown in the long-running schema update operation response.
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/databases/example-db/operations/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the schema update operation name by [listing long-running instance operations](#list_long-running_instance_operations) .

4.  Click **Execute** . The operation stops running.

## Backup and restore operations

The following are long-running backup operations.

  - [`  projects.instances.backups.create  `](/spanner/docs/reference/rest/v1/projects.instances.backups/create#try-it)
  - [`  projects.instances.databases.restore  `](/spanner/docs/reference/rest/v1/projects.instances.databases/restore#try-it)

### Check the progress of a long-running backup or restore operation

### Console

**Backup**

1.  In the Spanner navigation menu, select the **Operations** tab. The **Operations** page shows a list of currently running operations.

2.  Find the schema operation in the list. If it's still running, the progress bar in the **End time** column shows the percentage of the operation that is complete, as shown in the following image:

**Restore**

To check the progress of the restore operation, see the progress indicator that is displayed during the restore, as shown in the following image:

If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running instance operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_instance_operation) .

### gcloud

Use [`  gcloud spanner operations describe  `](/sdk/gcloud/reference/spanner/operations/describe) to check the progress of a backup or restore operation.

1.  Get the operation ID:
    
    ``` text
    gcloud spanner operations list \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME \
    --type=TYPE
    ```
    
    Replace the following:
    
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the name of the database.
      - TYPE : the type of the operation. Possible values are `  BACKUP  ` and `  DATABASE_RESTORE  ` .

2.  Run `  gcloud spanner operations describe  ` :
    
    ``` text
    gcloud spanner operations describe OPERATION-ID \
    --instance=INSTANCE-NAME \
    --database=DATABASE-NAME
    ```
    
    Replace the following:
    
      - OPERATION-ID : the operation ID of the operation that you want to check.
      - INSTANCE-NAME : the Spanner instance name.
      - DATABASE-NAME : the Spanner database name.
    
    The `  progress  ` section in the output shows the percentage of the operation that's complete. The output looks similar to the following:
    
    ``` text
    done: true
    metadata:
    ...
      progress:
      - endTime: '2022-03-01T00:28:06.691403Z'
        progressPercent: 100
        startTime: '2022-03-01T00:28:04.221401Z'
      - endTime: '2022-03-01T00:28:17.624588Z'
        startTime: '2022-03-01T00:28:06.691403Z'
        progressPercent: 100
    ...
    ```

### REST v1

Get the operation ID:

``` text
 gcloud spanner operations list 

   --instance=INSTANCE-NAME 

   --database=DATABASE-NAME 

   --type=DATABASE_UPDATE_DDL
 
```

Replace the following:

  - INSTANCE-NAME : the Spanner instance name.
  - DATABASE-NAME : the name of the database.

Before using any of the request data, make the following replacements:

  - PROJECT-ID : the project ID.
  - INSTANCE-ID : the instance ID.
  - DATABASE-ID : the database ID.
  - OPERATION-ID : the operation ID.

HTTP method and URL:

``` text
GET https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID
```

To send your request, expand one of these options:

#### curl (Linux, macOS, or Cloud Shell)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) , or by using [Cloud Shell](/shell/docs) , which automatically logs you into the `  gcloud  ` CLI . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
curl -X GET \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     "https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID"
```

#### PowerShell (Windows)

**Note:** The following command assumes that you have logged in to the `  gcloud  ` CLI with your user account by running [`  gcloud init  `](/sdk/gcloud/reference/init) or [`  gcloud auth login  `](/sdk/gcloud/reference/auth/login) . You can check the currently active account by running [`  gcloud auth list  `](/sdk/gcloud/reference/auth/list) .

Execute the following command:

``` text
$cred = gcloud auth print-access-token
$headers = @{ "Authorization" = "Bearer $cred" }

Invoke-WebRequest `
    -Method GET `
    -Headers $headers `
    -Uri "https://spanner.googleapis.com/v1/projects/PROJECT-ID/instances/INSTANCE-ID/databases/DATABASE-ID/operations/OPERATION-ID" | Select-Object -Expand Content
```

You should receive a JSON response similar to the following:

``` text
{
...
    "progress": [
      {
        "progressPercent": 100,
        "startTime": "2023-05-27T00:52:27.366688Z",
        "endTime": "2023-05-27T00:52:30.184845Z"
      },
      {
        "progressPercent": 100,
        "startTime": "2023-05-27T00:52:30.184845Z",
        "endTime": "2023-05-27T00:52:40.750959Z"
      }
    ],
...
  "done": true,
  "response": {
    "@type": "type.googleapis.com/google.protobuf.Empty"
  }
}
```

If the operation takes too long, you can cancel it. For more information, see [Cancel a long-running backup operation](/spanner/docs/manage-and-observe-long-running-operations#cancel_a_long-running_backup_operation) .

### List long-running backup or restore operations

Use [`  projects.instances.backups.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.backups.operations/list#try-it) to list the operations on a single backup or [`  projects.instances.backupOperations.list  `](/spanner/docs/reference/rest/v1/projects.instances.backupOperations/list#try-it) to list all backup operations in the instance.

1.  Navigate to [`  projects.instances.backups.operations.list  `](/spanner/docs/reference/rest/v1/projects.instances.backups.operations/list#try-it) .

2.  For **name** , enter:
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/backups/BACKUP-NAME/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - BACKUP-NAME : the name of the backup.
      - OPERATION-ID : the operations ID.

3.  Click **Execute** . The response contains a list of long-running operations.

### Cancel a long-running backup operation

Use [`  projects.instances.backups.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.backups.operations/cancel#try-it) to cancel a long-running backup operation.

1.  Navigate to [`  projects.instances.backups.operations.cancel  `](/spanner/docs/reference/rest/v1/projects.instances.backups.operations/cancel#try-it) .

2.  For **name** , enter the long-running backup operation name as shown in the long-running backup operation response.
    
    ``` text
    projects/PROJECT-ID/instances/INSTANCE-NAME/backups/BACKUP-NAME/operations/OPERATION-ID
    ```
    
    Replace the following:
    
      - PROJECT-ID : the project ID.
      - INSTANCE-NAME : the instance name.
      - BACKUP-NAME : the name of the backup.
      - OPERATION-ID : the operations ID.
    
    You can also retrieve the backup operation name by [listing long-running backup operations](#list-long-running-backup-operations) .

3.  Click **Execute** .
