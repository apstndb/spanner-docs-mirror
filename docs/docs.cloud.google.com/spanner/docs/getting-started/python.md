## Objectives

This tutorial walks you through the following steps using the Spanner client library for Python:

  - Create a Spanner instance and database.
  - Write, read, and execute SQL queries on data in the database.
  - Update the database schema.
  - Update data using a read-write transaction.
  - Add a secondary index to the database.
  - Use the index to read and execute SQL queries on data.
  - Retrieve data using a read-only transaction.

## Costs

This tutorial uses Spanner, which is a billable component of the Google Cloud. For information on the cost of using Spanner, see [Pricing](https://cloud.google.com/spanner/pricing) .

## Before you begin

Complete the steps described in [Set up](/spanner/docs/getting-started/set-up#set_up_a_project) , which cover creating and setting a default Google Cloud project, enabling billing, enabling the Cloud Spanner API, and setting up OAuth 2.0 to get authentication credentials to use the Cloud Spanner API.

In particular, make sure that you run [`  gcloud auth application-default login  `](/sdk/gcloud/reference/auth/application-default/login) to set up your local development environment with authentication credentials.

**Note:** If you don't plan to keep the resources that you create in this tutorial, consider creating a new Google Cloud project instead of selecting an existing project. After you finish the tutorial, you can delete the project, removing all resources associated with the project.

## Prepare your local Python environment

1.  Follow the instructions in [Setting Up a Python Development Environment](/python/docs/setup) .

2.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/googleapis/python-spanner
    ```
    
    Alternatively, you can [download the sample](https://github.com/googleapis/python-spanner/archive/main.zip) as a zip file and extract it.

3.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd python-spanner/samples/samples
    ```

4.  Create an isolated Python environment, and install dependencies:
    
    ``` text
    virtualenv -m venv env
    source env/bin/activate
    pip install -r requirements.txt
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with Python.

Take a look through the `  snippets.py  ` file, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
python snippets.py test-instance --database-id example-db create_database
```

### PostgreSQL

``` text
python pg_snippets.py test-instance --database-id example-db create_database
```

You should see:

``` text
Created database example-db on instance test-instance
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` python
def create_database(instance_id, database_id):
    """Creates a database and tables for sample data."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.CreateDatabaseRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        create_statement=f"CREATE DATABASE `{database_id}`",
        extra_statements=[
            """CREATE TABLE Singers (
            SingerId     INT64 NOT NULL,
            FirstName    STRING(1024),
            LastName     STRING(1024),
            SingerInfo   BYTES(MAX),
            FullName   STRING(2048) AS (
                ARRAY_TO_STRING([FirstName, LastName], " ")
            ) STORED
        ) PRIMARY KEY (SingerId)""",
            """CREATE TABLE Albums (
            SingerId     INT64 NOT NULL,
            AlbumId      INT64 NOT NULL,
            AlbumTitle   STRING(MAX)
        ) PRIMARY KEY (SingerId, AlbumId),
        INTERLEAVE IN PARENT Singers ON DELETE CASCADE""",
        ],
    )

    operation = database_admin_api.create_database(request=request)

    print("Waiting for operation to complete...")
    database = operation.result(OPERATION_TIMEOUT_SECONDS)

    print(
        "Created database {} on instance {}".format(
            database.name,
            database_admin_api.instance_path(spanner_client.project, instance_id),
        )
    )
```

### PostgreSQL

``` python
def create_database(instance_id, database_id):
    """Creates a PostgreSql database and tables for sample data."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.CreateDatabaseRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        create_statement=f'CREATE DATABASE "{database_id}"',
        database_dialect=DatabaseDialect.POSTGRESQL,
    )

    operation = database_admin_api.create_database(request=request)

    print("Waiting for operation to complete...")
    database = operation.result(OPERATION_TIMEOUT_SECONDS)

    create_table_using_ddl(database.name)
    print("Created database {} on instance {}".format(database_id, instance_id))


def create_table_using_ddl(database_name):
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_name,
        statements=[
            """CREATE TABLE Singers (
  SingerId   bigint NOT NULL,
  FirstName  character varying(1024),
  LastName   character varying(1024),
  SingerInfo bytea,
  FullName   character varying(2048)
    GENERATED ALWAYS AS (FirstName || ' ' || LastName) STORED,
  PRIMARY KEY (SingerId)
  )""",
            """CREATE TABLE Albums (
  SingerId     bigint NOT NULL,
  AlbumId      bigint NOT NULL,
  AlbumTitle   character varying(1024),
  PRIMARY KEY (SingerId, AlbumId)
  ) INTERLEAVE IN PARENT Singers ON DELETE CASCADE""",
        ],
    )
    operation = spanner_client.database_admin_api.update_database_ddl(request)
    operation.result(OPERATION_TIMEOUT_SECONDS)
```

The next step is to write data to your database.

## Create a database client

Before you can do reads or writes, you must create a [`  Client  `](/python/docs/reference/spanner/latest/client-api) . You can think of a `  Client  ` as a database connection: all of your interactions with Spanner must go through a `  Client  ` . Typically you create a `  Client  ` when your application starts up, then you re-use that `  Client  ` to read, write, and execute transactions. The following code shows how to create a client.

``` python
# Imports the Google Cloud Client Library.
from google.cloud import spanner

# Your Cloud Spanner instance ID.
# instance_id = "my-instance-id"
#
# Your Cloud Spanner database ID.
# database_id = "my-database-id"
# Instantiate a client.
spanner_client = spanner.Client()

# Get a Cloud Spanner instance by ID.
instance = spanner_client.instance(instance_id)

# Get a Cloud Spanner database by ID.
database = instance.database(database_id)

# Execute a simple SQL statement.
with database.snapshot() as snapshot:
    results = snapshot.execute_sql("SELECT 1")

    for row in results:
        print(row)
```

Read more in the [`  Client  `](/python/docs/reference/spanner/latest/client-api) reference.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  execute_update()  ` method to execute a DML statement.

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def insert_singers(transaction):
    row_ct = transaction.execute_update(
        "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES "
        "(12, 'Melissa', 'Garcia'), "
        "(13, 'Russell', 'Morales'), "
        "(14, 'Jacqueline', 'Long'), "
        "(15, 'Dylan', 'Shaw')"
    )
    print("{} record(s) inserted.".format(row_ct))

database.run_in_transaction(insert_singers)
```

Run the sample using the `  insert_with_dml  ` argument.

``` text
python snippets.py test-instance --database-id example-db insert_with_dml
```

You should see:

``` text
4 record(s) inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You write data using a [`  Batch  `](/python/docs/reference/spanner/latest/batch-api) object. A `  Batch  ` object is a container for mutation operations. A mutation represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a Spanner database.

The [`  insert()  `](/python/docs/reference/spanner/latest/batch-usage#inserting-records-using-a-batch) method in the `  Batch  ` class adds one or more insert mutations to the batch. All mutations in a single batch are applied atomically.

This code shows how to write the data using mutations:

``` python
def insert_data(instance_id, database_id):
    """Inserts sample data into the given database.

    The database and table must already exist and can be created using
    `create_database`.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.insert(
            table="Singers",
            columns=("SingerId", "FirstName", "LastName"),
            values=[
                (1, "Marc", "Richards"),
                (2, "Catalina", "Smith"),
                (3, "Alice", "Trentor"),
                (4, "Lea", "Martin"),
                (5, "David", "Lomond"),
            ],
        )

        batch.insert(
            table="Albums",
            columns=("SingerId", "AlbumId", "AlbumTitle"),
            values=[
                (1, 1, "Total Junk"),
                (1, 2, "Go, Go, Go"),
                (2, 1, "Green"),
                (2, 2, "Forever Hold Your Peace"),
                (2, 3, "Terrified"),
            ],
        )

    print("Inserted data.")
```

Run the sample using the `  insert_data  ` argument.

``` text
python snippets.py test-instance --database-id example-db insert_data
```

You should see:

``` text
Inserted data.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Python.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
```

**Note:** For the GoogleSQL reference, see [Query syntax in GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax) and for PostgreSQL reference, see [PostgreSQL lexical structure and syntax](/spanner/docs/reference/postgresql/lexical) .

The result shows:

``` text
SingerId AlbumId AlbumTitle
1        1       Total Junk
1        2       Go, Go, Go
2        1       Green
2        2       Forever Hold Your Peace
2        3       Terrified
```

### Use the Spanner client library for Python

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for Python.

Use the [`  execute_sql()  `](/python/docs/reference/spanner/latest/snapshot-api#google.cloud.spanner_v1.snapshot.Snapshot.execute_sql) method of a [`  Snapshot  `](/python/docs/reference/spanner/latest/snapshot-api) object to run the SQL query. To get a `  Snapshot  ` object, call the [`  snapshot()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.snapshot) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class in a `  with  ` statement.

Here's how to issue the query and access the data:

``` python
def query_data(instance_id, database_id):
    """Queries sample data from the database using SQL."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

Run the sample using the `  query_data  ` argument.

``` text
python snippets.py test-instance --database-id example-db query_data
```

You should see the following result:

``` text
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold Your Peace
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
```

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

### GoogleSQL

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId, FirstName, LastName FROM Singers "
        "WHERE LastName = @lastName",
        params={"lastName": "Garcia"},
        param_types={"lastName": spanner.param_types.STRING},
    )

    for row in results:
        print("SingerId: {}, FirstName: {}, LastName: {}".format(*row))
```

### PostgreSQL

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql(
        "SELECT SingerId, FirstName, LastName FROM Singers " "WHERE LastName = $1",
        params={"p1": "Garcia"},
        param_types={"p1": spanner.param_types.STRING},
    )

    for row in results:
        print("SingerId: {}, FirstName: {}, LastName: {}".format(*row))
```

Run the sample using the query\_data\_with\_parameter argument.

``` text
python snippets.py test-instance --database-id example-db query_data_with_parameter
```

You should see the following result:

``` text
SingerId: 12, FirstName: Melissa, LastName: Garcia
```

## Read data using the read API

In addition to Spanner's SQL interface, Spanner also supports a read interface.

Use the [`  read()  `](/python/docs/reference/spanner/latest/snapshot-api#google.cloud.spanner_v1.snapshot.Snapshot.read) method of a [`  Snapshot  `](/python/docs/reference/spanner/latest/snapshot-api) object to read rows from the database. To get a `  Snapshot  ` object, call the [`  snapshot()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.snapshot) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class in a `  with  ` statement. Use a [`  KeySet  `](/python/docs/reference/spanner/latest/keyset-api) object to define a collection of keys and key ranges to read.

Here's how to read the data:

``` python
def read_data(instance_id, database_id):
    """Reads sample data from the database."""
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

Run the sample using the `  read_data  ` argument.

``` text
python snippets.py test-instance --database-id example-db read_data
```

You should see output similar to:

``` text
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold Your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Python.

#### On the command line

Use the following [`  ALTER TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE Albums ADD COLUMN MarketingBudget BIGINT'
```

You should see:

``` text
Schema updating...done.
```

#### Use the Spanner client library for Python

Use the [`  update_ddl()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.update_ddl) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class to modify the schema:

### GoogleSQL

``` python
def add_column(instance_id, database_id):
    """Adds a new column to the Albums table in the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64",
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)
    print("Added the MarketingBudget column.")
```

### PostgreSQL

``` python
def add_column(instance_id, database_id):
    """Adds a new column to the Albums table in the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["ALTER TABLE Albums ADD COLUMN MarketingBudget BIGINT"],
    )
    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print("Added the MarketingBudget column.")
```

Run the sample using the `  add_column  ` argument.

``` text
python snippets.py test-instance --database-id example-db add_column
```

You should see:

``` text
Added the MarketingBudget column.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

``` python
def update_data(instance_id, database_id):
    """Updates sample data in the database.

    This updates the `MarketingBudget` column which must be created before
    running this sample. You can add the column by running the `add_column`
    sample or by running this DDL statement against your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.batch() as batch:
        batch.update(
            table="Albums",
            columns=("SingerId", "AlbumId", "MarketingBudget"),
            values=[(1, 1, 100000), (2, 2, 500000)],
        )

    print("Updated data.")
```

Run the sample using the `  update_data  ` argument.

``` text
python snippets.py test-instance --database-id example-db update_data
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

``` python
def query_data_with_new_column(instance_id, database_id):
    """Queries sample data from the database using SQL.

    This sample uses the `MarketingBudget` column. You can add the column
    by running the `add_column` sample or by running this DDL statement against
    your database:

        ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, MarketingBudget FROM Albums"
        )

        for row in results:
            print("SingerId: {}, AlbumId: {}, MarketingBudget: {}".format(*row))
```

To execute this query, run the sample using the `  query_data_with_new_column  ` argument.

``` text
python snippets.py test-instance --database-id example-db query_data_with_new_column
```

You should see:

``` text
SingerId: 2, AlbumId: 2, MarketingBudget: 500000
SingerId: 1, AlbumId: 2, MarketingBudget: None
SingerId: 2, AlbumId: 1, MarketingBudget: None
SingerId: 2, AlbumId: 3, MarketingBudget: None
SingerId: 1, AlbumId: 1, MarketingBudget: 100000
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  execute_update()  ` method to execute a DML statement.

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
database = instance.database(database_id)

def transfer_budget(transaction):
    # Transfer marketing budget from one album to another. Performed in a
    # single transaction to ensure that the transfer is atomic.
    second_album_result = transaction.execute_sql(
        "SELECT MarketingBudget from Albums " "WHERE SingerId = 2 and AlbumId = 2"
    )
    second_album_row = list(second_album_result)[0]
    second_album_budget = second_album_row[0]

    transfer_amount = 200000

    # Transaction will only be committed if this condition still holds at
    # the time of commit. Otherwise it will be aborted and the callable
    # will be rerun by the client library
    if second_album_budget >= transfer_amount:
        first_album_result = transaction.execute_sql(
            "SELECT MarketingBudget from Albums "
            "WHERE SingerId = 1 and AlbumId = 1"
        )
        first_album_row = list(first_album_result)[0]
        first_album_budget = first_album_row[0]

        second_album_budget -= transfer_amount
        first_album_budget += transfer_amount

        # Update first album
        transaction.execute_update(
            "UPDATE Albums "
            "SET MarketingBudget = @AlbumBudget "
            "WHERE SingerId = 1 and AlbumId = 1",
            params={"AlbumBudget": first_album_budget},
            param_types={"AlbumBudget": spanner.param_types.INT64},
        )

        # Update second album
        transaction.execute_update(
            "UPDATE Albums "
            "SET MarketingBudget = @AlbumBudget "
            "WHERE SingerId = 2 and AlbumId = 2",
            params={"AlbumBudget": second_album_budget},
            param_types={"AlbumBudget": spanner.param_types.INT64},
        )

        print(
            "Transferred {} from Album2's budget to Album1's".format(
                transfer_amount
            )
        )

database.run_in_transaction(transfer_budget)
```

Run the sample using the `  write_with_dml_transaction  ` argument.

``` text
python snippets.py test-instance --database-id example-db write_with_dml_transaction
```

You should see:

``` text
Transferred 200000 from Album2's budget to Album1's
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for Python.

#### On the command line

Use the following [`  CREATE INDEX  `](/spanner/docs/reference/standard-sql/data-definition-language#create_index) command to add an index to the database:

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)'
```

You should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for Python

Use the [`  update_ddl()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.update_ddl) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class to add an index:

``` python
def add_index(instance_id, database_id):
    """Adds a simple index to the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=["CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)"],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print("Added the AlbumsByAlbumTitle index.")
```

Run the sample using the `  add_index  ` argument.

``` text
python snippets.py test-instance --database-id example-db add_index
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added the AlbumsByAlbumTitle index.
```

### Read using the index

For SQL queries, Spanner automatically uses an appropriate index. In the read interface, you must specify the index in your request.

To use the index in the read interface, provide an `  Index  ` argument to the [`  read()  `](/python/docs/reference/spanner/latest/snapshot-api#google.cloud.spanner_v1.snapshot.Snapshot.read) method of a [`  Snapshot  `](/python/docs/reference/spanner/latest/snapshot-api) object. To get a `  Snapshot  ` object, call the [`  snapshot()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.snapshot) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class in a `  with  ` statement.

``` python
def read_data_with_index(instance_id, database_id):
    """Reads sample data from the database using an index.

    The index must exist before running this sample. You can add the index
    by running the `add_index` sample or by running this DDL statement against
    your database:

        CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums",
            columns=("AlbumId", "AlbumTitle"),
            keyset=keyset,
            index="AlbumsByAlbumTitle",
        )

        for row in results:
            print("AlbumId: {}, AlbumTitle: {}".format(*row))
```

Run the sample using the `  read_data_with_index  ` argument.

``` text
python snippets.py test-instance --database-id example-db read_data_with_index
```

You should see:

``` text
AlbumId: 2, AlbumTitle: Forever Hold Your Peace
AlbumId: 2, AlbumTitle: Go, Go, Go
AlbumId: 1, AlbumTitle: Green
AlbumId: 3, AlbumTitle: Terrified
AlbumId: 1, AlbumTitle: Total Junk
```

### Add an index for index-only reads

You might have noticed that the previous read example doesn't include reading the `  MarketingBudget  ` column. This is because Spanner's read interface doesn't support the ability to join an index with a data table to look up values that are not stored in the index.

Create an alternate definition of `  AlbumsByAlbumTitle  ` that stores a copy of `  MarketingBudget  ` in the index.

#### On the command line

### GoogleSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)
```

### PostgreSQL

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) INCLUDE (MarketingBudget)
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Schema updating...done.
```

#### Using the Spanner client library for Python

Use the [`  update_ddl()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.update_ddl) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class to add an index with a `  STORING  ` clause:

``` python
def add_storing_index(instance_id, database_id):
    """Adds an storing index to the example database."""

    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.UpdateDatabaseDdlRequest(
        database=database_admin_api.database_path(
            spanner_client.project, instance_id, database_id
        ),
        statements=[
            "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle)"
            "STORING (MarketingBudget)"
        ],
    )

    operation = database_admin_api.update_database_ddl(request)

    print("Waiting for operation to complete...")
    operation.result(OPERATION_TIMEOUT_SECONDS)

    print("Added the AlbumsByAlbumTitle2 index.")
```

Run the sample using the `  add_storing_index  ` argument.

``` text
python snippets.py test-instance --database-id example-db add_storing_index
```

You should see:

``` text
Added the AlbumsByAlbumTitle2 index.
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

``` python
def read_data_with_storing_index(instance_id, database_id):
    """Reads sample data from the database using an index with a storing
    clause.

    The index must exist before running this sample. You can add the index
    by running the `add_scoring_index` sample or by running this DDL statement
    against your database:

        CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle)
        STORING (MarketingBudget)

    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot() as snapshot:
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums",
            columns=("AlbumId", "AlbumTitle", "MarketingBudget"),
            keyset=keyset,
            index="AlbumsByAlbumTitle2",
        )

        for row in results:
            print("AlbumId: {}, AlbumTitle: {}, " "MarketingBudget: {}".format(*row))
```

Run the sample using the `  read_data_with_storing_index  ` argument.

``` text
python snippets.py test-instance --database-id example-db read_data_with_storing_index
```

You should see output similar to:

``` text
AlbumId: 2, AlbumTitle: Forever Hold Your Peace, MarketingBudget: 300000
AlbumId: 2, AlbumTitle: Go, Go, Go, MarketingBudget: None
AlbumId: 1, AlbumTitle: Green, MarketingBudget: None
AlbumId: 3, AlbumTitle: Terrified, MarketingBudget: None
AlbumId: 1, AlbumTitle: Total Junk, MarketingBudget: 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Use a [`  Snapshot  `](/python/docs/reference/spanner/latest/snapshot-api) object for executing read-only transactions. To get a `  Snapshot  ` object, call the [`  snapshot()  `](/python/docs/reference/spanner/latest/database-api#google.cloud.spanner_v1.database.Database.snapshot) method of the [`  Database  `](/python/docs/reference/spanner/latest/database-api) class in a `  with  ` statement.

The following shows how to run a query and perform a read in the same read-only transaction:

``` python
def read_only_transaction(instance_id, database_id):
    """Reads data inside of a read-only transaction.

    Within the read-only transaction, or "snapshot", the application sees
    consistent view of the database at a particular timestamp.
    """
    spanner_client = spanner.Client()
    instance = spanner_client.instance(instance_id)
    database = instance.database(database_id)

    with database.snapshot(multi_use=True) as snapshot:
        # Read using SQL.
        results = snapshot.execute_sql(
            "SELECT SingerId, AlbumId, AlbumTitle FROM Albums"
        )

        print("Results from first read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))

        # Perform another read using the `read` method. Even if the data
        # is updated in-between the reads, the snapshot ensures that both
        # return the same data.
        keyset = spanner.KeySet(all_=True)
        results = snapshot.read(
            table="Albums", columns=("SingerId", "AlbumId", "AlbumTitle"), keyset=keyset
        )

        print("Results from second read:")
        for row in results:
            print("SingerId: {}, AlbumId: {}, AlbumTitle: {}".format(*row))
```

Run the sample using the `  read_only_transaction  ` argument.

``` text
python snippets.py test-instance --database-id example-db read_only_transaction
```

You should see output similar to:

``` text
Results from first read:
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold Your Peace
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
Results from second read:
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold Your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
```

## Cleanup

To avoid incurring additional charges to your Cloud Billing account for the resources used in this tutorial, drop the database and delete the instance that you created.

### Delete the database

If you delete an instance, all databases within it are automatically deleted. This step shows how to delete a database without deleting an instance (you would still incur charges for the instance).

#### On the command line

``` text
gcloud spanner databases delete example-db --instance=test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click the instance.

3.  Click the database that you want to delete.

4.  In the **Database details** page, click **Delete** .

5.  Confirm that you want to delete the database and click **Delete** .

### Delete the instance

Deleting an instance automatically drops all databases created in that instance.

#### On the command line

``` text
gcloud spanner instances delete test-instance
```

#### Using the Google Cloud console

1.  Go to the **Spanner Instances** page in the Google Cloud console.

2.  Click your instance.

3.  Click **Delete** .

4.  Confirm that you want to delete the instance and click **Delete** .

## What's next

  - Learn how to [access Spanner with a virtual machine instance](/spanner/docs/configure-virtual-machine-instance) .

  - Learn about authorization and authentication credentials in [Authenticate to Cloud services using client libraries](/docs/authentication/getting-started) .

  - Learn more about Spanner [Schema design best practices](/spanner/docs/schema-design) .
