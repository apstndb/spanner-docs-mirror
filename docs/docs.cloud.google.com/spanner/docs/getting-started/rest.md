## Objectives

This tutorial walks you through the following steps using the Cloud Spanner API with REST:

  - Create a Spanner instance and database.
  - Write, read, and execute SQL queries on data in the database.
  - Update the database schema.
  - Add a secondary index to the database.
  - Use the index to read and execute SQL queries on data.
  - Retrieve data using a read-only transaction.

If you want to use Spanner client libraries instead of using the REST API, see [Tutorials](https://docs.cloud.google.com/spanner/docs/tutorials) .

## Costs

This tutorial uses Spanner, which is a billable component of the Google Cloud. For information on the cost of using Spanner, see [Pricing](https://cloud.google.com/spanner/pricing) .

## Before you begin

## Ways to make REST calls

You can make Spanner REST calls using:

  - The **Try-It\!** feature found in the [Spanner API reference documentation](https://docs.cloud.google.com/spanner/docs/reference/rest) .
  - [Google APIs Explorer](https://developers.google.com/explorer-help/) , which contains the [Cloud Spanner API](https://developers.google.com/apis-explorer/#p/spanner.googleapis.com/v1/) and other Google APIs.
  - Other tools or frameworks that support HTTP REST calls.

## Conventions used on this page

  - The examples use `  <var>PROJECT_ID</var>  ` as the Google Cloud project ID. Substitute your Google Cloud project ID for `  <var>PROJECT_ID</var>  ` .

  - The examples create and use an instance ID of `  test-instance  ` . Substitute your instance ID if you are not using `  test-instance  ` .

  - The examples create and use a database ID of `  example-db  ` . Substitute your database ID if you are not using `  example-db  ` .

  - The examples use `  <var>SESSION</var>  ` as part of a session name. Substitute the value you receive when you [create a session](https://docs.cloud.google.com/spanner/docs/getting-started/rest#create_a_session) for `  <var>SESSION</var>  ` .

  - The examples use a transaction ID of `  <var>TRANSACTION_ID</var>  ` . Substitute the value you receive when you create a transaction for `  <var>TRANSACTION_ID</var>  ` .

  - The **Try-It\!** functionality supports interactively adding individual HTTP request fields. Most examples in this document provide the entire request instead of describing how to interactively add individual fields to the request.

## Instances

When you first use Spanner, create an instance. An instance allocates resources that Spanner databases use. When you create an instance, you choose where your data is stored and how much [compute capacity](https://docs.cloud.google.com/spanner/docs/compute-capacity) the instance has.

### List instance configurations

When you create an instance, you specify an *instance configuration* , which defines the geographic placement and replication of your databases in that instance. Choose a regional configuration to store data in one region, or a multi-region configuration to distribute data across multiple regions. Learn more in [Instances](https://docs.cloud.google.com/spanner/docs/instances) .

Use `  projects.instanceConfigs.list  ` to determine which configurations are available for your Google Cloud project.

1.  Click [`  projects.instanceConfigs.list  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instanceConfigs/list#try-it) .

2.  For **parent** , enter:
    
    projects/ PROJECT\_ID

3.  Click **Execute** . The response shows the available instance configurations. Here's an example response (your project may have different instance configurations):
    
        { "instanceConfigs": [ { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-asia-south1", "displayName":
        "asia-south1" }, { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-asia-east1", "displayName":
        "asia-east1" }, { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-asia-northeast1",
        "displayName": "asia-northeast1" }, { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-europe-west1",
        "displayName": "europe-west1" }, { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-us-east4", "displayName":
        "us-east4" }, { "name":
        "projects/<var>PROJECT_ID</var>/instanceConfigs/regional-us-central1", "displayName":
        "us-central1" } ] }

You use the `  name  ` value for one of the instance configurations when you create your instance.

### Create an instance

1.  Click [`  projects.instances.create  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/create#try-it) .

2.  For **parent** , enter:
    
        projects/<var>PROJECT_ID</var>

3.  Click **Add request body parameters** and select `  instance  ` .

4.  Click the hint bubble for **instance** to see the possible fields. Add values for the following fields:
    
      - `  nodeCount  ` : Enter `  1  ` .
      - `  config  ` : Enter the `  name  ` value of one of the regional instance configurations returned when you [list instance configurations](https://docs.cloud.google.com/spanner/docs/getting-started/rest#listing_instance_configurations) .
      - `  displayName  ` : Enter `  Test Instance  ` .

5.  Click the hint bubble that follows the closing bracket for **instance** and select **instanceId** .

6.  For `  instanceId  ` , enter `  test-instance  ` .  
    Your **Try It\!** instance creation page should now look like this:
    
    ![Instance creation page in the Try-It\! feature](https://docs.cloud.google.com/static/spanner/docs/images/create_instance_try_it.png)

7.  Click **Execute** . The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

List instances using [`  projects.instances.list  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/list) .

## Create a database

Create a database named `  example-db  ` .

1.  Click [`  projects.instances.databases.create  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/create#try-it) .

2.  For **parent** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance

3.  Click **Add request body parameters** and select `  createStatement  ` .

4.  For `  createStatement  ` , enter:
    
        CREATE DATABASE `example-db`

The database name, `  example-db  ` , contains a hyphen, so enclose it in backticks ( ``  `  `` ).

1.  Click **Execute** . The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

List your databases using [`  projects.instances.databases.list  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/list) .

## Create a schema

Use Spanner's [Data Definition Language](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-definition-language) (DDL) to create, alter, or drop tables, and to create or drop indexes.

1.  Click [`  projects.instances.databases.updateDdl  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it) .

2.  For **database** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  For **Request body** , use the following:
    
        {
          "statements": [
            "CREATE TABLE Singers ( SingerId INT64 NOT NULL, FirstName STRING(1024), LastName STRING(1024), SingerInfo BYTES(MAX) ) PRIMARY KEY (SingerId)",
           "CREATE TABLE Albums ( SingerId INT64 NOT NULL, AlbumId INT64 NOT NULL, AlbumTitle STRING(MAX)) PRIMARY KEY (SingerId, AlbumId), INTERLEAVE IN PARENT Singers ON DELETE CASCADE"
          ]
        }
    
    The `  statements  ` array contains the DDL statements that define the schema.

4.  Click **Execute** . The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

The schema defines two tables, `  Singers  ` and `  Albums  ` , for a basic music application. This document uses these tables. Review the [example schema](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

Retrieve your schema using [`  projects.instances.databases.getDdl  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/getDdl) .

## Create a session

Before you add, update, delete, or query data, create a [session](https://docs.cloud.google.com/spanner/docs/sessions) . A session represents a communication channel with the Spanner database service. (You do not directly use a session if you are using a Spanner [client library](https://docs.cloud.google.com/spanner/docs/reference/libraries) , because the client library manages sessions on your behalf.)

1.  Click [`  projects.instances.databases.sessions.create  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/create#try-it) .

2.  For **database** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  Click **Execute** .

4.  The response shows the session that you created, in the form
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>
    
    Use this session when you read or write to your database.

Sessions are intended to be long-lived. The Spanner database service deletes a session when the session is idle for more than one hour. Attempts to use a deleted session result in `  NOT_FOUND  ` . If you encounter this error, create and use a new session. See if a session is still alive using [`  projects.instances.databases.sessions.get  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/get) .

For related information, see [Keep an idle session alive](https://docs.cloud.google.com/spanner/docs/sessions#keep_an_idle_session_alive) .

Sessions are an advanced concept. For more details and best practices, see [Sessions](https://docs.cloud.google.com/spanner/docs/sessions) .

Next, write data to your database.

## Write data

You write data using the [`  Mutation  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#Mutation) type. A `  Mutation  ` is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, deletes, and other actions that apply atomically to different rows and tables in a Spanner database.

1.  Click [`  projects.instances.databases.sessions.commit  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
    ``` 
     {
       "singleUseTransaction": {
         "readWrite": {}
       },
       "mutations": [
         {
           "insertOrUpdate": {
             "table": "Singers",
             "columns": [
               "SingerId",
               "FirstName",
               "LastName"
             ],
             "values": [
               [
                 "1",
                 "Marc",
                 "Richards"
               ],
               [
                 "2",
                 "Catalina",
                 "Smith"
               ],
               [
                 "3",
                 "Alice",
                 "Trentor"
               ],
               [
                 "4",
                 "Lea",
                 "Martin"
               ],
               [
                 "5",
                 "David",
                 "Lomond"
               ]
             ]
           }
         },
         {
           "insertOrUpdate": {
             "table": "Albums",
             "columns": [
               "SingerId",
               "AlbumId",
               "AlbumTitle"
             ],
             "values": [
               [
                 "1",
                 "1",
                 "Total Junk"
               ],
               [
                 "1",
                 "2",
                 "Go, Go, Go"
               ],
               [
                 "2",
                 "1",
                 "Green"
               ],
               [
                 "2",
                 "2",
                 "Forever Hold Your Peace"
               ],
               [
                 "2",
                 "3",
                 "Terrified"
               ]
             ]
           }
         }
       ]
     }
    ```

4.  Click **Execute** . The response shows the commit timestamp.

This example used `  insertOrUpdate  ` . Other [operations](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#Mutation.FIELDS) for `  Mutations  ` are `  insert  ` , `  update  ` , `  replace  ` , and `  delete  ` .

For information on how to encode data types, see [TypeCode](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetMetadata#TypeCode) .

## Query data using SQL

1.  Click [`  projects.instances.databases.sessions.executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
          "sql": "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        }

4.  Click **Execute** . The response shows the query results.

## Read data using the read API

1.  Click [`  projects.instances.databases.sessions.read  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
          "table": "Albums",
          "columns": [
            "SingerId",
            "AlbumId",
            "AlbumTitle"
          ],
          "keySet": {
            "all": true
          }
        }

4.  Click **Execute** . The response shows the read results.

## Update the database schema

Add a new column called `  MarketingBudget  ` to the `  Albums  ` table. This requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates do not require taking the database offline and they do not lock entire tables or columns; you can continue writing data to the database during the schema update.

### Add a column

1.  Click [`  projects.instances.databases.updateDdl  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it) .

2.  For **database** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  For **Request body** , use the following:
    
    ```` 
     {
       "statements": [
         "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64"
       ]
     }
     ```
    
    The `statements` array contains the DDL statements that define the schema.
    ````

4.  Click **Execute** . This may take a few minutes to complete, even after the REST call returns a response. The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

### Write data to the new column

This code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

1.  Click [`  projects.instances.databases.sessions.commit  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/commit#try-it) .

2.  For **session** , enter:
    
    ``` 
         projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>
    ```
    
    (You receive this value when you [create a session](https://docs.cloud.google.com/spanner/docs/getting-started/rest#create_a_session) .)

3.  For **Request body** , use the following:
    
    ``` 
     {
       "singleUseTransaction": {
         "readWrite": {}
       },
       "mutations": [
         {
           "update": {
             "table": "Albums",
             "columns": [
               "SingerId",
               "AlbumId",
               "MarketingBudget"
             ],
             "values": [
               [
                 "1",
                 "1",
                 "100000"
               ],
               [
                 "2",
                 "2",
                 "500000"
               ]
             ]
           }
         }
       ]
     }
    ```

4.  Click **Execute** . The response shows the commit timestamp.

Execute a SQL query or a read call to fetch the values you just wrote.

1.  Click [`  projects.instances.databases.sessions.executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
    ``` 
     {
       "sql": "SELECT SingerId, AlbumId, MarketingBudget FROM Albums"
     }
    ```

4.  Click **Execute** . The response shows two rows that contain the updated `  MarketingBudget  ` values:
    
    ``` 
     "rows": [
       [
         "1",
         "1",
         "100000"
       ],
       [
         "1",
         "2",
         null
       ],
       [
         "2",
         "1",
         null
       ],
       [
         "2",
         "2",
         "500000"
       ],
       [
         "2",
         "3",
         null
       ]
     ]
    ```

## Use a secondary index

To fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range, read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that do not meet the criteria. However, this full table scan is expensive, especially for tables with many rows. To speed up row retrieval when searching by non-primary key columns, create a [secondary index](https://docs.cloud.google.com/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Similar to other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to certain tables or columns during this process. For more information, see [index backfilling](https://docs.cloud.google.com/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that run faster with the index. If you use the read interface, specify the index that you want to use.

### Add a secondary index

Add an index using `  updateDdl  ` .

1.  Click [`  projects.instances.databases.updateDdl  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it) .

2.  For **database** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  For **Request body** , use the following:
    
        {
           "statements": [
             "CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)"
           ]
        }

4.  Click **Execute** . This may take a few minutes to complete, even after the REST call returns a response. The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

### Query using the index

1.  Click [`  projects.instances.databases.sessions.executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
          "sql": "SELECT AlbumId, AlbumTitle, MarketingBudget FROM Albums WHERE AlbumTitle >= 'Aardvark' AND AlbumTitle < 'Goo'"
        }

4.  Click **Execute** . The response shows the following rows:
    
        "rows": [
           [
             "2",
             "Go, Go, Go",
             null
           ],
           [
             "2",
             "Forever Hold Your Peace",
             "500000"
           ]
        ]

### Read using the index

1.  Click [`  projects.instances.databases.sessions.read  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
           "table": "Albums",
           "columns": [
             "AlbumId",
             "AlbumTitle"
           ],
           "keySet": {
             "all": true
           },
           "index": "AlbumsByAlbumTitle"
        }

4.  Click **Execute** . The response shows the following rows:
    
        "rows": [
           [
             "2",
             "Forever Hold Your Peace"
           ],
           [
             "2",
             "Go, Go, Go"
           ],
           [
             "1",
             "Green"
           ],
           [
             "3",
             "Terrified"
           ],
           [
             "1",
             "Total Junk"
           ]
        ]

### Add an index with the STORING clause

The previous read example did not include the `  MarketingBudget  ` column. This occurs because the Spanner read interface does not support joining an index with a data table to look up values not stored in the index.

Create an alternate definition of `  AlbumsByAlbumTitle  ` that stores a copy of `  MarketingBudget  ` in the index.

Add a STORING index using `  updateDdl  ` .

1.  Click [`  projects.instances.databases.updateDdl  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/updateDdl#try-it) .

2.  For **database** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  For **Request body** , use the following:
    
        {
          "statements": [
            "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)"
          ]
        }

4.  Click **Execute** . This may take a few minutes to complete, even after the REST call returns a response. The response returns a [long-running operation](https://docs.cloud.google.com/spanner/docs/manage-long-running-operations) . Query this operation to check its status.

Now, execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

1.  Click [`  projects.instances.databases.sessions.read  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
          "table": "Albums",
          "columns": [
            "AlbumId",
            "AlbumTitle",
            "MarketingBudget"
          ],
          "keySet": {
            "all": true
          },
          "index": "AlbumsByAlbumTitle2"
        }

4.  Click **Execute** . The response shows the following rows:
    
        "rows": [
           [
             "2",
             "Forever Hold Your Peace",
             "500000"
           ],
           [
             "2",
             "Go, Go, Go",
             null
           ],
           [
             "1",
             "Green",
             null
           ],
           [
             "3",
             "Terrified",
             null
           ],
           [
             "1",
             "Total Junk",
             "100000"
           ]
        ]

## Retrieve data using read-only transactions

To execute more than one read at the same timestamp, use [Read-only transactions](https://docs.cloud.google.com/spanner/docs/transactions#read-only_transactions) . These transactions observe a consistent prefix of the transaction commit history, so your application always gets consistent data.

### Create a read-only transaction

1.  Click [`  projects.instances.databases.sessions.beginTransaction  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/beginTransaction#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request Body** , use the following:
    
        {
          "options": {
            "readOnly": {}
          }
        }

4.  Click **Execute** .

5.  The response shows the ID of the transaction that you created.

Use the read-only transaction to retrieve data at a consistent timestamp, even if the data changed after you created the read-only transaction.

### Run a query using the read-only transaction

1.  Click [`  projects.instances.databases.sessions.executeSql  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/executeSql#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
          "sql": "SELECT SingerId, AlbumId, AlbumTitle FROM Albums",
          "transaction": {
            "id": "<var>TRANSACTION_ID</var>"
          }
        }

4.  Click **Execute** . The response shows rows similar to the following:
    
        "rows": [
           [
             "2",
             "2",
             "Forever Hold Your Peace"
           ],
           [
             "1",
             "2",
             "Go, Go, Go"
           ],
           [
             "2",
             "1",
             "Green"
           ],
           [
             "2",
             "3",
             "Terrified"
           ],
           [
             "1",
             "1",
             "Total Junk"
           ]
        ]

### Read using the read-only transaction

1.  Click [`  projects.instances.databases.sessions.read  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/read#try-it) .

2.  For **session** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db/sessions/<var>SESSION</var>

3.  For **Request body** , use the following:
    
        {
           "table": "Albums",
           "columns": [
             "SingerId",
             "AlbumId",
             "AlbumTitle"
           ],
           "keySet": {
             "all": true
           },
           "transaction": {
             "id": "<var>TRANSACTION_ID</var>"
           }
        }

4.  Click **Execute** . The response shows rows similar to the following:
    
        "rows": [
           [
             "1",
             "1",
             "Total Junk"
           ],
           [
             "1",
             "2",
             "Go, Go, Go"
           ],
           [
             "2",
             "1",
             "Green"
           ],
           [
             "2",
             "2",
             "Forever Hold Your Peace"
           ],
           [
             "2",
             "3",
             "Terrified"
           ]
        ]

Spanner also supports read-write transactions, which execute a set of reads and writes atomically at a single logical point in time. For more information, see [Read-write transactions](https://docs.cloud.google.com/spanner/docs/transactions#read-write_transactions) . (The **Try-It\!** functionality is not suitable for demonstrating a read-write transaction.)

## Cleanup

To avoid additional charges to your Google Cloud account for the resources used in this tutorial, drop the database and delete the instance you created.

### Drop a database

1.  Click [`  projects.instances.databases.dropDatabase  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances.databases/dropDatabase#try-it) .

2.  For **name** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance/databases/example-db

3.  Click **Execute** .

### Delete an instance

1.  Click [`  projects.instances.delete  `](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/projects.instances/delete#try-it) .

2.  For **name** , enter:
    
        projects/<var>PROJECT_ID</var>/instances/test-instance

3.  Click **Execute** .

## What's next

  - [Access Spanner in a Virtual Machine Instance](https://docs.cloud.google.com/spanner/docs/configure-virtual-machine-instance) : create a virtual machine instance with access to your Spanner database.
  - Learn more about [Spanner concepts](https://docs.cloud.google.com/spanner/docs/concepts) .
