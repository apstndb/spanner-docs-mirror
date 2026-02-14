## Objectives

This tutorial walks you through the following steps using the Spanner client library for PHP:

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

## Prepare your local PHP environment

1.  Follow the steps in [Service accounts](/docs/authentication#service_accounts) to set up a service account as your Application Default Credentials. Following those steps, you should obtain both a service account key file (in JSON) and a `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable that lets you authenticate to the Spanner API.

2.  Install the following on your development machine if they are not already installed:
    
      - [PHP](http://php.net/)
      - [composer](https://getcomposer.org/)
      - [gRPC](/php/grpc)

3.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/GoogleCloudPlatform/php-docs-samples
    ```
    
    Alternatively, you can [download the sample](https://github.com/GoogleCloudPlatform/php-docs-samples/archive/master.zip) as a zip file and extract it.

4.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd php-docs-samples/spanner
    ```

5.  Install dependencies:
    
    ``` text
    composer install
    ```
    
    This installs the Spanner client library for PHP, which you can add to any project by running `  composer require google/cloud-spanner  ` .

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with PHP.

Take a look at the functions in `  src/create_database.php  ` and `  src/add_column.php  ` , which show how to create a database and modify a database schema. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
php src/create_database.php test-instance example-db
```

### PostgreSQL

``` text
php src/pg_create_database.php test-instance example-db
```

You should see:

``` text
Created database example-db on instance test-instance
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateDatabaseRequest;

/**
 * Creates a database and tables for sample data.
 * Example:
 * ```
 * create_database($instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_database(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $instance = $databaseAdminClient->instanceName($projectId, $instanceId);

    $operation = $databaseAdminClient->createDatabase(
        new CreateDatabaseRequest([
            'parent' => $instance,
            'create_statement' => sprintf('CREATE DATABASE `%s`', $databaseId),
            'extra_statements' => [
                'CREATE TABLE Singers (' .
                'SingerId     INT64 NOT NULL,' .
                'FirstName    STRING(1024),' .
                'LastName     STRING(1024),' .
                'SingerInfo   BYTES(MAX),' .
                'FullName     STRING(2048) AS' .
                '(ARRAY_TO_STRING([FirstName, LastName], " ")) STORED' .
                ') PRIMARY KEY (SingerId)',
                'CREATE TABLE Albums (' .
                    'SingerId     INT64 NOT NULL,' .
                    'AlbumId      INT64 NOT NULL,' .
                    'AlbumTitle   STRING(MAX)' .
                ') PRIMARY KEY (SingerId, AlbumId),' .
                'INTERLEAVE IN PARENT Singers ON DELETE CASCADE'
            ]
        ])
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created database %s on instance %s' . PHP_EOL,
        $databaseId, $instanceId);
}
````

### PostgreSQL

``` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\CreateDatabaseRequest;
use Google\Cloud\Spanner\Admin\Database\V1\DatabaseDialect;
use Google\Cloud\Spanner\Admin\Database\V1\GetDatabaseRequest;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Creates a database that uses Postgres dialect
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_create_database(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $instance = $databaseAdminClient->instanceName($projectId, $instanceId);
    $databaseName = $databaseAdminClient->databaseName($projectId, $instanceId, $databaseId);

    $table1Query = 'CREATE TABLE Singers (
        SingerId   bigint NOT NULL PRIMARY KEY,
        FirstName  varchar(1024),
        LastName   varchar(1024),
        SingerInfo bytea,
        FullName character varying(2048) GENERATED
        ALWAYS AS (FirstName || \' \' || LastName) STORED
    )';
    $table2Query = 'CREATE TABLE Albums (
        AlbumId      bigint NOT NULL,
        SingerId     bigint NOT NULL REFERENCES Singers (SingerId),
        AlbumTitle   text,
        PRIMARY KEY(SingerId, AlbumId)
    )';

    $operation = $databaseAdminClient->createDatabase(
        new CreateDatabaseRequest([
            'parent' => $instance,
            'create_statement' => sprintf('CREATE DATABASE "%s"', $databaseId),
            'extra_statements' => [],
            'database_dialect' => DatabaseDialect::POSTGRESQL
        ])
    );

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$table1Query, $table2Query]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);
    $operation->pollUntilComplete();

    $database = $databaseAdminClient->getDatabase(
        new GetDatabaseRequest(['name' => $databaseAdminClient->databaseName($projectId, $instanceId, $databaseId)])
    );
    $dialect = DatabaseDialect::name($database->getDatabaseDialect());

    printf('Created database %s with dialect %s on instance %s' . PHP_EOL,
        $databaseId, $dialect, $instanceId);
}
```

The next step is to write data to your database.

## Create a database client

To do reads and writes, you need to obtain an instance of [`  Google\Cloud\Spanner\Database  `](/php/docs/reference/cloud-spanner/latest/database) .

``` php
# Includes the autoloader for libraries installed with composer
require __DIR__ . '/vendor/autoload.php';

# Imports the Google Cloud client library
use Google\Cloud\Spanner\SpannerClient;

# Your Google Cloud Platform project ID
$projectId = 'YOUR_PROJECT_ID';

# Instantiates a client
$spanner = new SpannerClient([
    'projectId' => $projectId
]);

# Your Cloud Spanner instance ID.
$instanceId = 'your-instance-id';

# Get a Cloud Spanner instance by ID.
$instance = $spanner->instance($instanceId);

# Your Cloud Spanner database ID.
$databaseId = 'your-database-id';

# Get a Cloud Spanner database by ID.
$database = $instance->database($databaseId);

# Execute a simple SQL statement.
$results = $database->execute('SELECT "Hello World" as test');

foreach ($results as $row) {
    print($row['test'] . PHP_EOL);
}
```

You can think of a `  Database  ` as a database connection: all of your interactions with Spanner must go through a `  Database  ` . Typically you create a `  Database  ` when your application starts up, then you re-use that `  Database  ` to read, write, and execute transactions. Each client uses resources in Spanner.

If you create multiple clients in the same app, you should call [`  Database::close  `](/php/docs/reference/cloud-spanner/latest/database?method=close) to clean up the client's resources, including network connections, as soon as it is no longer needed.

Read more in the [`  Database  `](/php/docs/reference/cloud-spanner/latest/database) reference.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  executeUpdate()  ` method to execute a DML statement.

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Inserts sample data into the given database with a DML statement.
 *
 * The database and table must already exist and can be created using
 * `create_database`.
 * Example:
 * ```
 * insert_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function write_data_with_dml(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        $rowCount = $t->executeUpdate(
            'INSERT Singers (SingerId, FirstName, LastName) VALUES '
            . "(12, 'Melissa', 'Garcia'), "
            . "(13, 'Russell', 'Morales'), "
            . "(14, 'Jacqueline', 'Long'), "
            . "(15, 'Dylan', 'Shaw')");
        $t->commit();
        printf('Inserted %d row(s).' . PHP_EOL, $rowCount);
    });
}
````

Run the sample file `  src/write_data_with_dml.php  ` .

``` text
php src/write_data_with_dml.php test-instance example-db
```

You should see:

``` text
Inserted 4 row(s).
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You write data using the [`  Database::insertBatch  `](/php/docs/reference/cloud-spanner/latest/database?method=insertBatch) method. `  insertBatch  ` adds new rows to a table. All inserts in a single batch are applied atomically.

This code shows how to write the data using mutations:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Inserts sample data into the given database.
 *
 * The database and table must already exist and can be created using
 * `create_database`.
 * Example:
 * ```
 * insert_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function insert_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->insertBatch('Singers', [
            ['SingerId' => 1, 'FirstName' => 'Marc', 'LastName' => 'Richards'],
            ['SingerId' => 2, 'FirstName' => 'Catalina', 'LastName' => 'Smith'],
            ['SingerId' => 3, 'FirstName' => 'Alice', 'LastName' => 'Trentor'],
            ['SingerId' => 4, 'FirstName' => 'Lea', 'LastName' => 'Martin'],
            ['SingerId' => 5, 'FirstName' => 'David', 'LastName' => 'Lomond'],
        ])
        ->insertBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'AlbumTitle' => 'Total Junk'],
            ['SingerId' => 1, 'AlbumId' => 2, 'AlbumTitle' => 'Go, Go, Go'],
            ['SingerId' => 2, 'AlbumId' => 1, 'AlbumTitle' => 'Green'],
            ['SingerId' => 2, 'AlbumId' => 2, 'AlbumTitle' => 'Forever Hold Your Peace'],
            ['SingerId' => 2, 'AlbumId' => 3, 'AlbumTitle' => 'Terrified']
        ])
        ->commit();

    print('Inserted data.' . PHP_EOL);
}
````

Run the sample file `  src/insert_data.php  ` .

``` text
php src/insert_data.php test-instance example-db
```

You should see:

``` text
Inserted data.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for PHP.

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

### Use the Spanner client library for PHP

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for PHP.

Use [`  Database::execute()  `](/php/docs/reference/cloud-spanner/latest/database?method=execute) to run the SQL query.

Here's how to issue the query and access the data:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL.
 * Example:
 * ```
 * query_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
    );

    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

Run the sample file `  src/query_data.php  ` .

``` text
php src/query_data.php test-instance example-db
```

You should see the following result:

``` text
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold Your Peace
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
```

Your results won't necessarily be in this order. If you need to ensure the ordering of the result, use an `  ORDER BY  ` clause, as documented in [SQL best practices](/spanner/docs/sql-best-practices#use_order_by_to_ensure_the_ordering_of_your_sql_results) .

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

### GoogleSQL

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL with a parameter.
 * Example:
 * ```
 * query_data_with_parameter($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, FirstName, LastName FROM Singers ' .
        'WHERE LastName = @lastName',
        ['parameters' => ['lastName' => 'Garcia']]
    );

    foreach ($results as $row) {
        printf('SingerId: %s, FirstName: %s, LastName: %s' . PHP_EOL,
            $row['SingerId'], $row['FirstName'], $row['LastName']);
    }
}
````

### PostgreSQL

``` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Execute a query with parameters on a Spanner PostgreSQL database.
 * The PostgreSQL dialect uses positional parameters, as
 * opposed to the named parameters of Cloud Spanner.
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_query_parameter(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    printf('Listing all singers with a last name that starts with \'A\'' . PHP_EOL);

    $results = $database->execute(
        'SELECT SingerId, FirstName, LastName' .
        ' FROM Singers' .
        ' WHERE LastName LIKE $1',
        [
        'parameters' => [
            'p1' => 'A%'
        ]
    ]
    );

    foreach ($results as $row) {
        printf('SingerId: %s, Firstname: %s, LastName: %s' . PHP_EOL, $row['singerid'], $row['firstname'], $row['lastname']);
    }
}
```

Run the sample file `  src/query_data_with_parameter.php  ` .

``` text
php src/query_data_with_parameter.php test-instance example-db
```

You should see the following result:

``` text
SingerId: 12, FirstName: Melissa, LastName: Garcia
```

## Read data using the read API

In addition to Spanner's SQL interface, Spanner also supports a read interface.

Use [`  Database::read()  `](/php/docs/reference/cloud-spanner/latest/database?method=read) to read rows from the database. Use a `  KeySet  ` object to define a collection of keys and key ranges to read.

Here's how to read the data:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database.
 * Example:
 * ```
 * read_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle']
    );

    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

Run the sample in `  read_data.php  ` file.

``` text
php read_data.php test-instance example-db
```

You should see output similar to:

``` text
SingerId: 1, AlbumId: 1, AlbumTitle: Total Junk
SingerId: 1, AlbumId: 2, AlbumTitle: Go, Go, Go
SingerId: 2, AlbumId: 1, AlbumTitle: Green
SingerId: 2, AlbumId: 2, AlbumTitle: Forever Hold your Peace
SingerId: 2, AlbumId: 3, AlbumTitle: Terrified
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for PHP.

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

#### Use the Spanner client library for PHP

Use [`  Database::updateDdl  `](/php/docs/reference/cloud-spanner/latest/database?method=updateDdl) to modify the schema:

### GoogleSQL

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds a new column to the Albums table in the example database.
 * Example:
 * ```
 * add_column($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_column(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => ['ALTER TABLE Albums ADD COLUMN MarketingBudget INT64']
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added the MarketingBudget column.' . PHP_EOL);
}
````

### PostgreSQL

``` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Add a column to a table present in a PG Spanner database.
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_add_column(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = 'ALTER TABLE Albums ADD COLUMN MarketingBudget bigint';
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    print('Added column MarketingBudget on table Albums' . PHP_EOL);
}
```

Run the sample file `  src/add_column.php  ` .

``` text
php src/add_column.php test-instance example-db
```

You should see:

``` text
Added the MarketingBudget column.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Updates sample data in the database.
 *
 * This updates the `MarketingBudget` column which must be created before
 * running this sample. You can add the column by running the `add_column`
 * sample or by running this DDL statement against your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * update_data($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function update_data(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $operation = $database->transaction(['singleUse' => true])
        ->updateBatch('Albums', [
            ['SingerId' => 1, 'AlbumId' => 1, 'MarketingBudget' => 100000],
            ['SingerId' => 2, 'AlbumId' => 2, 'MarketingBudget' => 500000],
        ])
        ->commit();

    print('Updated data.' . PHP_EOL);
}
````

Run the sample file `  src/update_data.php  ` .

``` text
php src/update_data.php test-instance example-db
```

You should see:

``` text
Updated data.
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Queries sample data from the database using SQL.
 * This sample uses the `MarketingBudget` column. You can add the column
 * by running the `add_column` sample or by running this DDL statement against
 * your database:
 *
 *      ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * query_data_with_new_column($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function query_data_with_new_column(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $results = $database->execute(
        'SELECT SingerId, AlbumId, MarketingBudget FROM Albums'
    );

    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, MarketingBudget: %d' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['MarketingBudget']);
    }
}
````

To execute this query, run the sample file `  src/query-data-with-new-column.php  ` .

``` text
php src/query_data_with_new_column.php test-instance example-db
```

You should see:

``` text
SingerId: 1, AlbumId: 1, MarketingBudget: 100000
SingerId: 1, AlbumId: 2, MarketingBudget: 0
SingerId: 2, AlbumId: 1, MarketingBudget: 0
SingerId: 2, AlbumId: 2, MarketingBudget: 500000
SingerId: 2, AlbumId: 3, MarketingBudget: 0
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  executeUpdate()  ` method to execute a DML statement.

### GoogleSQL

```` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 * Performs a read-write transaction to update two sample records in the
 * database.
 *
 * This will transfer 200,000 from the `MarketingBudget` field for the second
 * Album to the first Album. If the `MarketingBudget` for the second Album is
 * too low, it will raise an exception.
 *
 * Before running this sample, you will need to run the `update_data` sample
 * to populate the fields.
 * Example:
 * ```
 * write_data_with_dml_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function write_data_with_dml_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $database->runTransaction(function (Transaction $t) {
        // Transfer marketing budget from one album to another. We do it in a transaction to
        // ensure that the transfer is atomic.
        $transferAmount = 200000;

        $results = $t->execute(
            'SELECT MarketingBudget from Albums WHERE SingerId = 2 and AlbumId = 2'
        );
        $resultsRow = $results->rows()->current();
        $album2budget = $resultsRow['MarketingBudget'];

        // Transaction will only be committed if this condition still holds at the time of
        // commit. Otherwise it will be aborted and the callable will be rerun by the
        // client library.
        if ($album2budget >= $transferAmount) {
            $results = $t->execute(
                'SELECT MarketingBudget from Albums WHERE SingerId = 1 and AlbumId = 1'
            );
            $resultsRow = $results->rows()->current();
            $album1budget = $resultsRow['MarketingBudget'];

            $album2budget -= $transferAmount;
            $album1budget += $transferAmount;

            // Update the albums
            $t->executeUpdate(
                'UPDATE Albums '
                . 'SET MarketingBudget = @AlbumBudget '
                . 'WHERE SingerId = 1 and AlbumId = 1',
                [
                    'parameters' => [
                        'AlbumBudget' => $album1budget
                    ]
                ]
            );
            $t->executeUpdate(
                'UPDATE Albums '
                . 'SET MarketingBudget = @AlbumBudget '
                . 'WHERE SingerId = 2 and AlbumId = 2',
                [
                    'parameters' => [
                        'AlbumBudget' => $album2budget
                    ]
                ]
            );

            $t->commit();

            print('Transaction complete.' . PHP_EOL);
        }
    });
}
````

### PostgreSQL

``` php
use Google\Cloud\Spanner\SpannerClient;
use Google\Cloud\Spanner\Transaction;

/**
 *
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function pg_dml_getting_started_update(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    // Transfer marketing budget from one album to another. We do it in a transaction to
    // ensure that the transfer is atomic.
    $database->runTransaction(function (Transaction $t) {
        $sql = 'SELECT marketingbudget as "MarketingBudget" from Albums WHERE '
        . 'SingerId = 2 and AlbumId = 2';

        $result = $t->execute($sql);
        $row = $result->rows()->current();
        $budgetAlbum2 = $row['MarketingBudget'];
        $transfer = 200000;

        // Transaction will only be committed if this condition still holds at the time of
        // commit. Otherwise it will be aborted.
        if ($budgetAlbum2 > $transfer) {
            $sql = 'SELECT marketingbudget as "MarketingBudget" from Albums WHERE '
            . 'SingerId = 1 and AlbumId = 1';
            $result = $t->execute($sql);
            $row = $result->rows()->current();
            $budgetAlbum1 = $row['MarketingBudget'];

            $budgetAlbum1 += $transfer;
            $budgetAlbum2 -= $transfer;

            $t->executeUpdateBatch([
                [
                    'sql' => 'UPDATE Albums '
                    . 'SET MarketingBudget = $1 '
                    . 'WHERE SingerId = 1 and AlbumId = 1',
                    [
                        'parameters' => [
                            'p1' => $budgetAlbum1
                        ]
                    ]
                ],
                [
                    'sql' => 'UPDATE Albums '
                    . 'SET MarketingBudget = $1 '
                    . 'WHERE SingerId = 2 and AlbumId = 2',
                    [
                        'parameters' => [
                            'p1' => $budgetAlbum2
                        ]
                    ]
                ],
            ]);
            $t->commit();

            print('Marketing budget updated.' . PHP_EOL);
        } else {
            $t->rollback();
        }
    });
}
```

Run the sample file `  src/write_data_with_dml_transaction.php  ` .

``` text
php src/write_data_with_dml_transaction.php test-instance example-db
```

You should see:

``` text
Transaction complete.
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for PHP.

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

#### Using the Spanner client library for PHP

Use [`  Database::updateDdl  `](/php/docs/reference/cloud-spanner/latest/database?method=updateDdl) to add an index:

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds a simple index to the example database.
 * Example:
 * ```
 * create_index($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_index(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = 'CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)';
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added the AlbumsByAlbumTitle index.' . PHP_EOL);
}
````

Run the sample file `  src/create_index.php  ` .

``` text
php src/create_index.php test-instance example-db
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added the AlbumsByAlbumTitle index.
```

### Read using the index

For SQL queries, Spanner automatically uses an appropriate index. In the read interface, you must specify the index in your request.

To use the index in the read interface, use the [`  Database::read  `](/php/docs/reference/cloud-spanner/latest/database?method=read) method.

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database using an index.
 *
 * The index must exist before running this sample. You can add the index
 * by running the `add_index` sample or by running this DDL statement against
 * your database:
 *
 *     CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)
 *
 * Example:
 * ```
 * read_data_with_index($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data_with_index(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['AlbumId', 'AlbumTitle'],
        ['index' => 'AlbumsByAlbumTitle']
    );

    foreach ($results->rows() as $row) {
        printf('AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

Run the sample file `  src/read_data_with_index.php  ` .

``` text
php src/read_data_with_index.php test-instance example-db
```

You should see:

``` text
AlbumId: 2, AlbumTitle: Forever Hold your Peace
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

#### Using the Spanner client library for PHP

Use [`  Database::updateDdl  `](/php/docs/reference/cloud-spanner/latest/database?method=updateDdl) to add an index with a `  STORING  ` clause:

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds an storing index to the example database.
 *
 * This sample uses the `MarketingBudget` column. You can add the column
 * by running the `add_column` sample or by running this DDL statement against
 * your database:
 *
 *     ALTER TABLE Albums ADD COLUMN MarketingBudget INT64
 *
 * Example:
 * ```
 * create_storing_index($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function create_storing_index(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);
    $statement = 'CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) ' .
        'STORING (MarketingBudget)';
    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [$statement]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    print('Waiting for operation to complete...' . PHP_EOL);
    $operation->pollUntilComplete();

    printf('Added the AlbumsByAlbumTitle2 index.' . PHP_EOL);
}
````

Run the sample file `  src/create_storing_index.php  ` .

``` text
php src/create_storing_index.php test-instance example-db
```

You should see:

``` text
Added the AlbumsByAlbumTitle2 index.
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads sample data from the database using an index with a storing
 * clause.
 *
 * The index must exist before running this sample. You can add the index
 * by running the `add_storing_index` sample or by running this DDL statement
 * against your database:
 *
 *     CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle)
 *     STORING (MarketingBudget)
 *
 * Example:
 * ```
 * read_data_with_storing_index($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data_with_storing_index(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['AlbumId', 'AlbumTitle', 'MarketingBudget'],
        ['index' => 'AlbumsByAlbumTitle2']
    );

    foreach ($results->rows() as $row) {
        printf('AlbumId: %s, AlbumTitle: %s, MarketingBudget: %d' . PHP_EOL,
            $row['AlbumId'], $row['AlbumTitle'], $row['MarketingBudget']);
    }
}
````

Run the sample file `  src/read_data_with_storing_index.php  ` .

``` text
php src/read_data_with_storing_index.php test-instance example-db
```

You should see output similar to:

``` text
AlbumId: 2, AlbumTitle: Forever Hold your Peace, MarketingBudget: 300000
AlbumId: 2, AlbumTitle: Go, Go, Go, MarketingBudget: 0
AlbumId: 1, AlbumTitle: Green, MarketingBudget: 0
AlbumId: 3, AlbumTitle: Terrified, MarketingBudget: 0
AlbumId: 1, AlbumTitle: Total Junk, MarketingBudget: 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Use a [`  Snapshot  `](/php/docs/reference/cloud-spanner/latest/snapshot) object for executing read-only transactions. Use the [`  Database::snapshot  `](/php/docs/reference/cloud-spanner/latest/database?method=snapshot) method to get a `  Snapshot  ` object.

The following shows how to run a query and perform a read in the same read-only transaction:

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Reads data inside of a read-only transaction.
 *
 * Within the read-only transaction, or "snapshot", the application sees
 * consistent view of the database at a particular timestamp.
 * Example:
 * ```
 * read_only_transaction($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_only_transaction(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId);

    $snapshot = $database->snapshot();
    $results = $snapshot->execute(
        'SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
    );
    print('Results from the first read:' . PHP_EOL);
    foreach ($results as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }

    // Perform another read using the `read` method. Even if the data
    // is updated in-between the reads, the snapshot ensures that both
    // return the same data.
    $keySet = $spanner->keySet(['all' => true]);
    $results = $database->read(
        'Albums',
        $keySet,
        ['SingerId', 'AlbumId', 'AlbumTitle']
    );

    print('Results from the second read:' . PHP_EOL);
    foreach ($results->rows() as $row) {
        printf('SingerId: %s, AlbumId: %s, AlbumTitle: %s' . PHP_EOL,
            $row['SingerId'], $row['AlbumId'], $row['AlbumTitle']);
    }
}
````

Run the sample file `  src/read_only_transaction.php  ` .

``` text
php src/read_only_transaction.php test-instance example-db
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
