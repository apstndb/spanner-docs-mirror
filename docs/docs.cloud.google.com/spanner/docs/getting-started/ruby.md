## Objectives

This tutorial walks you through the following steps using the Spanner client library for Ruby:

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

## Prepare your local Ruby environment

1.  Install the following on your development machine if they are not already installed:
    
      - [Ruby](https://www.ruby-lang.org/en/downloads/)
      - [Bundler](https://bundler.io/#getting-started)

2.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/GoogleCloudPlatform/ruby-docs-samples
    ```
    
    Alternatively, you can [download the sample](https://github.com/GoogleCloudPlatform/ruby-docs-samples/archive/main.zip) as a zip file and extract it.

3.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd ruby-docs-samples/spanner/
    ```

4.  Install dependencies:
    
    ``` text
    bundle install
    ```

5.  Set the GOOGLE\_CLOUD\_PROJECT environment variable to your Google Cloud project ID:
    
    ``` text
    export GOOGLE_CLOUD_PROJECT=[MY_PROJECT_ID]
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with Ruby.

Take a look through the `  spanner_samples.rb  ` file, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
bundle exec ruby spanner_samples.rb create_database test-instance example-db
```

### PostgreSQL

``` text
bundle exec ruby spanner_postgresql_create_database.rb postgresql_create_database MY_PROJECT_ID test-instance example-db
```

You should see:

``` text
Created database example-db on instance test-instance
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

job = database_admin_client.create_database parent: instance_path,
                                            create_statement: "CREATE DATABASE `#{database_id}`",
                                            extra_statements: [
                                              "CREATE TABLE Singers (
      SingerId     INT64 NOT NULL,
      FirstName    STRING(1024),
      LastName     STRING(1024),
      SingerInfo   BYTES(MAX)
    ) PRIMARY KEY (SingerId)",

                                              "CREATE TABLE Albums (
      SingerId     INT64 NOT NULL,
      AlbumId      INT64 NOT NULL,
      AlbumTitle   STRING(MAX)
    ) PRIMARY KEY (SingerId, AlbumId),
    INTERLEAVE IN PARENT Singers ON DELETE CASCADE"
                                            ]

puts "Waiting for create database operation to complete"

job.wait_until_done!

puts "Created database #{database_id} on instance #{instance_id}"
```

### PostgreSQL

``` ruby
require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

def postgresql_create_database project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project: project_id

  instance_path = database_admin_client.instance_path project: project_id, instance: instance_id

  job = database_admin_client.create_database parent: instance_path,
                                              create_statement: "CREATE DATABASE \"#{database_id}\"",
                                              database_dialect: :POSTGRESQL

  puts "Waiting for create database operation to complete"

  job.wait_until_done!

  puts "Created database #{database_id} on instance #{instance_id}"
end
```

The next step is to write data to your database.

## Create a database client

Before you can do reads or writes, you must create a [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) . You can think of a `  Client  ` as a database connection: all of your interactions with Spanner must go through a `  Client  ` . Typically you create a `  Client  ` when your application starts up, then you re-use that `  Client  ` to read, write, and execute transactions. The following code shows how to create a client.

``` ruby
# Imports the Google Cloud client library
require "google/cloud/spanner"

# Your Google Cloud Platform project ID
project_id = "YOUR_PROJECT_ID"

# Instantiates a client
spanner = Google::Cloud::Spanner.new project: project_id

# Your Cloud Spanner instance ID
instance_id = "my-instance"

# Your Cloud Spanner database ID
database_id = "my-database"

# Gets a reference to a Cloud Spanner instance database
database_client = spanner.client instance_id, database_id

# Execute a simple SQL statement
results = database_client.execute_query "SELECT 1"
results.rows.each do |row|
  puts row
end
```

Read more in the [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) reference.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  execute_update()  ` method to execute a DML statement.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id
row_count = 0

client.transaction do |transaction|
  row_count = transaction.execute_update(
    "INSERT INTO Singers (SingerId, FirstName, LastName) VALUES
     (12, 'Melissa', 'Garcia'),
     (13, 'Russell', 'Morales'),
     (14, 'Jacqueline', 'Long'),
     (15, 'Dylan', 'Shaw'),
     (16, 'Billie', 'Eillish'),
     (17, 'Judy', 'Garland'),
     (18, 'Taylor', 'Swift'),
     (19, 'Miley', 'Cyrus'),
     (20, 'Michael', 'Jackson'),
     (21, 'Ariana', 'Grande'),
     (22, 'Elvis', 'Presley'),
     (23, 'Kanye', 'West'),
     (24, 'Lady', 'Gaga'),
     (25, 'Nick', 'Jonas')"
  )
end

puts "#{row_count} records inserted."
```

Run the sample using the `  write_using_dml  ` argument.

``` text
bundle exec ruby spanner_samples.rb write_using_dml test-instance example-db
```

You should see:

``` text
 4 records inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

You write data using a [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) object. The [`  Client#commit  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#commit-instance_method) method creates and commits a transaction for writes that execute atomically at a single logical point in time across columns, rows, and tables in a database.

This code shows how to write the data using mutations:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.commit do |c|
  c.insert "Singers", [
    { SingerId: 1, FirstName: "Marc",     LastName: "Richards" },
    { SingerId: 2, FirstName: "Catalina", LastName: "Smith"    },
    { SingerId: 3, FirstName: "Alice",    LastName: "Trentor"  },
    { SingerId: 4, FirstName: "Lea",      LastName: "Martin"   },
    { SingerId: 5, FirstName: "David",    LastName: "Lomond"   }
  ]
  c.insert "Albums", [
    { SingerId: 1, AlbumId: 1, AlbumTitle: "Total Junk" },
    { SingerId: 1, AlbumId: 2, AlbumTitle: "Go, Go, Go" },
    { SingerId: 2, AlbumId: 1, AlbumTitle: "Green" },
    { SingerId: 2, AlbumId: 2, AlbumTitle: "Forever Hold Your Peace" },
    { SingerId: 2, AlbumId: 3, AlbumTitle: "Terrified" }
  ]
end

puts "Inserted data"
```

Run the sample using the `  insert_data  ` argument.

``` text
bundle exec ruby spanner_samples.rb insert_data test-instance example-db
```

You should see:

``` text
Inserted data
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Ruby.

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

### Use the Spanner client library for Ruby

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for Ruby.

Use the [`  Client#execute  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#execute-instance_method) method to run the SQL query. Use a Ruby symbol `  :ColumnName  ` to access data for a specific column from a row.

Here's how to issue the query and access the data:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

Run the sample using the `  query_data  ` argument.

``` text
bundle exec ruby spanner_samples.rb query_data test-instance example-db
```

You should see the following result:

``` text
1 1 Total Junk
1 2 Go, Go, Go
2 1 Green
2 2 Forever Hold Your Peace
2 3 Terrified
```

### Query using a SQL parameter

If your application has a frequently executed query, you can improve its performance by parameterizing it. The resulting parametric query can be cached and reused, which reduces compilation costs. For more information, see [Use query parameters to speed up frequently executed queries](/spanner/docs/sql-best-practices#query-parameters) .

Here is an example of using a parameter in the `  WHERE  ` clause to query records containing a specific value for `  LastName  ` .

### GoogleSQL

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

sql_query = "SELECT SingerId, FirstName, LastName
             FROM Singers
             WHERE LastName = @lastName"

params      = { lastName: "Garcia" }
param_types = { lastName: :STRING }

client.execute(sql_query, params: params, types: param_types).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:FirstName]} #{row[:LastName]}"
end
```

### PostgreSQL

``` ruby
def spanner_postgresql_query_parameter project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  require "google/cloud/spanner"

  spanner = Google::Cloud::Spanner.new project: project_id
  client  = spanner.client instance_id, database_id

  sql_query = "SELECT SingerId, FirstName, LastName FROM Singers WHERE FirstName LIKE $1"
  params = { p1: "A%" }

  results = client.execute sql_query, params: params

  results.rows.each do |row|
    puts "SingerId: #{row[:singerid]}"
    puts "FirstName: #{row[:firstname]}"
    puts "LastName: #{row[:lastname]}"
  end
end
```

Run the sample using the query\_with\_parameter argument.

``` text
bundle exec ruby spanner_samples.rb query_with_parameter test-instance example-db
```

You should see the following result:

``` text
12 Melissa Garcia
```

## Read data using the read API

In addition to Spanner's SQL interface, Spanner also supports a read interface.

Use the [`  Client#read  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#read-instance_method) method of the [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) class to read rows from the database.

Here's how to read the data:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.read("Albums", [:SingerId, :AlbumId, :AlbumTitle]).rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

Run the sample using the `  read_data  ` argument.

``` text
bundle exec ruby spanner_samples.rb read_data test-instance example-db
```

You should see output similar to:

``` text
1 1 Total Junk
1 2 Go, Go, Go
2 1 Green
2 2 Forever Hold Your Peace
2 3 Terrified
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Ruby.

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

#### Use the Spanner client library for Ruby

Use the [`  Database#update  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database#update-instance_method) method of the [`  Database  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database) class to modify the schema:

### GoogleSQL

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

job = database_admin_client.update_database_ddl database: db_path,
                                                statements: [
                                                  "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the MarketingBudget column"
```

### PostgreSQL

``` ruby
require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

def spanner_postgresql_add_column project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  db_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin project: project_id

  db_path = db_admin_client.database_path project: project_id,
                                          instance: instance_id,
                                          database: database_id

  add_column_query = "ALTER TABLE Singers ADD COLUMN Age INTEGER"

  job = db_admin_client.update_database_ddl database: db_path,
                                            statements: [add_column_query]

  job.wait_until_done!

  if job.error?
    puts "Error while adding column. Code: #{job.error.code}. Message: #{job.error.message}"
    raise GRPC::BadStatus.new(job.error.code, job.error.message)
  end

  puts "Added Age column to Singers table in datbase #{database_id}"
end
```

Run the sample using the `  add_column  ` argument.

``` text
bundle exec ruby spanner_samples.rb add_column test-instance example-db
```

You should see:

``` text
Added the MarketingBudget column
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.commit do |c|
  c.update "Albums", [
    { SingerId: 1, AlbumId: 1, MarketingBudget: 100_000 },
    { SingerId: 2, AlbumId: 2, MarketingBudget: 500_000 }
  ]
end

puts "Updated data"
```

Run the sample using the `  update_data  ` argument.

``` text
bundle exec ruby spanner_samples.rb update_data test-instance example-db
```

You should see:

``` text
Updated data
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.execute("SELECT SingerId, AlbumId, MarketingBudget FROM Albums").rows.each do |row|
  puts "#{row[:SingerId]} #{row[:AlbumId]} #{row[:MarketingBudget]}"
end
```

To execute this query, run the sample using the `  query_data_with_new_column  ` argument.

``` text
bundle exec ruby spanner_samples.rb query_data_with_new_column test-instance example-db
```

You should see:

``` text
1 1 100000
1 2
2 1
2 2 500000
2 3
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  execute_update()  ` method to execute a DML statement.

### GoogleSQL

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner         = Google::Cloud::Spanner.new project: project_id
client          = spanner.client instance_id, database_id
transfer_amount = 200_000

client.transaction do |transaction|
  first_album = transaction.execute(
    "SELECT MarketingBudget from Albums
     WHERE SingerId = 1 and AlbumId = 1"
  ).rows.first
  second_album = transaction.execute(
    "SELECT MarketingBudget from Albums
    WHERE SingerId = 2 and AlbumId = 2"
  ).rows.first
  raise "The second album does not have enough funds to transfer" if second_album[:MarketingBudget] < transfer_amount

  new_first_album_budget  = first_album[:MarketingBudget] + transfer_amount
  new_second_album_budget = second_album[:MarketingBudget] - transfer_amount

  transaction.execute_update(
    "UPDATE Albums SET MarketingBudget = @albumBudget WHERE SingerId = 1 and AlbumId = 1",
    params: { albumBudget: new_first_album_budget }
  )
  transaction.execute_update(
    "UPDATE Albums SET MarketingBudget = @albumBudget WHERE SingerId = 2 and AlbumId = 2",
    params: { albumBudget: new_second_album_budget }
  )
end

puts "Transaction complete"
```

### PostgreSQL

``` ruby
def spanner_postgresql_dml_getting_started_update project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  require "google/cloud/spanner"

  spanner = Google::Cloud::Spanner.new project: project_id
  client = spanner.client instance_id, database_id

  client.transaction do |transaction|
    transaction.execute_update(
      "UPDATE Singers SET Rating = $1 WHERE SingerId = 1",
      params: { p1: BigDecimal(4) },
      types: { p1: :PG_NUMERIC }
    )
  end

  puts "Transaction complete"
end
```

Run the sample using the `  write_with_transaction_using_dml  ` argument.

``` text
bundle exec ruby spanner_samples.rb write_with_transaction_using_dml test-instance example-db
```

You should see:

``` text
Transaction complete
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for Ruby.

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

#### Using the Spanner client library for Ruby

Use the [`  Database#update  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database#update-instance_method) method of the [`  Database  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database) class to add an index:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

job = database_admin_client.update_database_ddl database: db_path,
                                                statements: [
                                                  "CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the AlbumsByAlbumTitle index"
```

Run the sample using the `  create_index  ` argument.

``` text
bundle exec ruby spanner_samples.rb create_index test-instance example-db
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added the AlbumsByAlbumTitle index
```

### Read using the index

For SQL queries, Spanner automatically uses an appropriate index. In the read interface, you must specify the index in your request.

To use the index in the read interface, provide an `  index  ` parameter to the [`  read  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#read-instance_method) method of the [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) class.

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

result = client.read "Albums", [:AlbumId, :AlbumTitle],
                     index: "AlbumsByAlbumTitle"

result.rows.each do |row|
  puts "#{row[:AlbumId]} #{row[:AlbumTitle]}"
end
```

Run the sample using the `  read_data_with_index  ` argument.

``` text
bundle exec ruby spanner_samples.rb read_data_with_index test-instance example-db
```

You should see:

``` text
2 Forever Hold Your Peace
2 Go, Go, Go
1 Green
3 Terrified
1 Total Junk
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

#### Using the Spanner client library for Ruby

Use the [`  Database#update  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database#update-instance_method) method of the [`  Database  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Database) class to add an index with a `  STORING  ` clause:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"
require "google/cloud/spanner/admin/database"

database_admin_client = Google::Cloud::Spanner::Admin::Database.database_admin

db_path = database_admin_client.database_path project: project_id,
                                              instance: instance_id,
                                              database: database_id

job = database_admin_client.update_database_ddl database: db_path,
                                                statements: [
                                                  "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle)
   STORING (MarketingBudget)"
                                                ]

puts "Waiting for database update to complete"

job.wait_until_done!

puts "Added the AlbumsByAlbumTitle2 storing index"
```

Run the sample using the `  create_storing_index  ` argument.

``` text
bundle exec ruby spanner_samples.rb create_storing_index test-instance example-db
```

You should see:

``` text
Added the AlbumsByAlbumTitle2 index
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

result = client.read "Albums", [:AlbumId, :AlbumTitle, :MarketingBudget],
                     index: "AlbumsByAlbumTitle2"

result.rows.each do |row|
  puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:MarketingBudget]}"
end
```

Run the sample using the `  read_data_with_storing_index  ` argument.

``` text
bundle exec ruby spanner_samples.rb read_data_with_storing_index test-instance example-db
```

You should see output similar to:

``` text
2 Forever Hold Your Peace 300000
2 Go, Go, Go
1 Green
3 Terrified
1 Total Junk 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Use a [`  Snapshot  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Snapshot) object for executing read-only transactions. Use the [`  snapshot  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client#snapshot-instance_method) method of the [`  Client  `](/ruby/docs/reference/google-cloud-spanner/latestGoogle/Cloud/Spanner/Client) class to get a `  Snapshot  ` object.

The following shows how to run a query and perform a read in the same read-only transaction:

``` ruby
# project_id  = "Your Google Cloud project ID"
# instance_id = "Your Spanner instance ID"
# database_id = "Your Spanner database ID"

require "google/cloud/spanner"

spanner = Google::Cloud::Spanner.new project: project_id
client  = spanner.client instance_id, database_id

client.snapshot do |snapshot|
  snapshot.execute("SELECT SingerId, AlbumId, AlbumTitle FROM Albums").rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end

  # Even if changes occur in-between the reads, the transaction ensures that
  # both return the same data.
  snapshot.read("Albums", [:AlbumId, :AlbumTitle, :SingerId]).rows.each do |row|
    puts "#{row[:AlbumId]} #{row[:AlbumTitle]} #{row[:SingerId]}"
  end
end
```

Run the sample using the `  read_only_transaction  ` argument.

``` text
bundle exec ruby spanner_samples.rb read_only_transaction test-instance example-db
```

You should see output similar to:

``` text
2 Forever Hold Your Peace 2
2 Go, Go, Go 1
1 Green 2
3 Terrified 2
1 Total Junk 1
1 Total Junk 1
2 Go, Go, Go 1
1 Green 2
2 Forever Hold Your Peace 2
3 Terrified 2
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
