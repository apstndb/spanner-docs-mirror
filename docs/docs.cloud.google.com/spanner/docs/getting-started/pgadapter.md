## Objectives

This tutorial walks you through the following steps using the Spanner PGAdapter local proxy for PostgreSQL drivers:

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

## Prepare your local PGAdapter environment

You can use PostgreSQL drivers in combination with PGAdapter to connect to Spanner. PGAdapter is a local proxy that translates the PostgreSQL network protocol to the Spanner gRPC protocol.

**Tip:** See [Latency Comparisons](https://github.com/GoogleCloudPlatform/pgadapter/tree/postgresql-dialect/benchmarks/latency-comparison) for benchmarks comparing the latency of PGAdapter and the regular Spanner client libraries.

PGAdapter requires either Java or Docker to run.

1.  Install one of the following on your development machine if none of them are already installed:
    
      - Java 8 JDK ( [download](http://openjdk.java.net/) ).
      - Docker ( [download](https://docs.docker.com/get-docker/) ).

2.  Clone the sample app repository to your local machine:
    
    ``` text
    git clone https://github.com/GoogleCloudPlatform/pgadapter.git
    ```

3.  Change to the directory that contains the Spanner sample code:
    
    ### psql
    
    ``` text
    cd pgadapter/samples/snippets/psql-snippets
    ```
    
    ### Java
    
    ``` text
    cd pgadapter/samples/snippets/java-snippets
    mvn package -DskipTests
    ```
    
    ### Go
    
    ``` text
    cd pgadapter/samples/snippets/golang-snippets
    ```
    
    ### Node.js
    
    ``` text
    cd pgadapter/samples/snippets/nodejs-snippets
    npm install
    ```
    
    ### Python
    
    ``` text
    cd pgadapter/samples/snippets/python-snippets
    python -m venv ./venv
    pip install -r requirements.txt
    cd samples
    ```
    
    ### C\#
    
    ``` text
    cd pgadapter/samples/snippets/dotnet-snippets
    ```
    
    ### PHP
    
    ``` text
    cd pgadapter/samples/snippets/php-snippets
    composer install
    cd samples
    ```

## Create an instance

**Tip:** You can skip this step if you are using PGAdapter with the Emulator. PGAdapter automatically creates the instance on the emulator when you connect to PGAdapter.

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with PGAdapter.

Take a look through the `  samples/snippets  ` folder, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Start PGAdapter

Start PGAdapter on your local development machine and point it to the instance that you created.

The following commands assume that you have executed `  gcloud auth application-default login  ` .

### Java Application

``` text
wget https://storage.googleapis.com/pgadapter-jar-releases/pgadapter.tar.gz \
    && tar -xzvf pgadapter.tar.gz
java -jar pgadapter.jar -i test-instance
```

### Docker

``` text
docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter
docker run \
    --name pgadapter \
    --rm -d -p 5432:5432 \
    -v "$HOME/.config/gcloud":/gcloud:ro \
    --env CLOUDSDK_CONFIG=/gcloud \
    gcr.io/cloud-spanner-pg-adapter/pgadapter \
    -i test-instance -x
```

**Note:** Replace `  $HOME/.config/gcloud/  ` with `  %APPDATA%\gcloud  ` if you are on Windows, or run `  gcloud info --format='value(config.paths.global_config_dir)'  ` to find your current `  gcloud  ` configuration folder.

### Emulator

``` text
docker pull gcr.io/cloud-spanner-pg-adapter/pgadapter-emulator
docker run \
    --name pgadapter-emulator \
    --rm -d \
    -p 5432:5432 \
    -p 9010:9010 \
    -p 9020:9020 \
    gcr.io/cloud-spanner-pg-adapter/pgadapter-emulator
```

This starts PGAdapter with an embedded Spanner emulator. This embedded emulator automatically creates any Spanner instance or database that you connect to without the need to manually create them beforehand.

We recommend that you run PGAdapter in production as either a side-car container or as an in-process dependency. For more information on deploying PGAdapter in production, see [Choose a method for running PGAdapter](/spanner/docs/pgadapter-start#run-pgadapter) .

## Create a database

**Tip:** You can skip this step if you are using PGAdapter with the Emulator. PGAdapter automatically creates the database on the emulator when you connect to PGAdapter.

``` text
gcloud spanner databases create example-db --instance=test-instance \
--database-dialect=POSTGRESQL
```

You should see:

``` text
Creating database...done.
```

### Create tables

The following code creates two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### psql

``` bash
#!/bin/bash

# Set the connection variables for psql.
# The following statements use the existing value of the variable if it has
# already been set, and otherwise assigns a default value.
export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Create two tables in one batch.
psql << SQL
-- Create the singers table
CREATE TABLE singers (
  singer_id   bigint not null primary key,
  first_name  character varying(1024),
  last_name   character varying(1024),
  singer_info bytea,
  full_name   character varying(2048) GENERATED ALWAYS
          AS (first_name || ' ' || last_name) STORED
);

-- Create the albums table. This table is interleaved in the parent table
-- "singers".
CREATE TABLE albums (
  singer_id     bigint not null,
  album_id      bigint not null,
  album_title   character varying(1024),
  primary key (singer_id, album_id)
)
-- The 'interleave in parent' clause is a Spanner-specific extension to
-- open-source PostgreSQL.
INTERLEAVE IN PARENT singers ON DELETE CASCADE;
SQL

echo "Created Singers & Albums tables in database: [${PGDATABASE}]"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

class CreateTables {
  static void createTables(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (Statement statement = connection.createStatement()) {
        // Create two tables in one batch.
        statement.addBatch(
            "create table singers ("
                + "  singer_id   bigint primary key not null,"
                + "  first_name  varchar(1024),"
                + "  last_name   varchar(1024),"
                + "  singer_info bytea,"
                + "  full_name   varchar(2048) generated always as (\n"
                + "      case when first_name is null then last_name\n"
                + "          when last_name  is null then first_name\n"
                + "          else first_name || ' ' || last_name\n"
                + "      end) stored"
                + ")");
        statement.addBatch(
            "create table albums ("
                + "  singer_id     bigint not null,"
                + "  album_id      bigint not null,"
                + "  album_title   varchar,"
                + "  primary key (singer_id, album_id)"
                + ") interleave in parent singers on delete cascade");
        statement.executeBatch();
        System.out.println("Created Singers & Albums tables in database: [" + database + "]");
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func CreateTables(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Create two tables in one batch on Spanner.
    br := conn.SendBatch(ctx, &pgx.Batch{QueuedQueries: []*pgx.QueuedQuery{
        {SQL: "create table singers (" +
            "  singer_id   bigint primary key not null," +
            "  first_name  character varying(1024)," +
            "  last_name   character varying(1024)," +
            "  singer_info bytea," +
            "  full_name   character varying(2048) generated " +
            "  always as (first_name || ' ' || last_name) stored" +
            ")"},
        {SQL: "create table albums (" +
            "  singer_id     bigint not null," +
            "  album_id      bigint not null," +
            "  album_title   character varying(1024)," +
            "  primary key (singer_id, album_id)" +
            ") interleave in parent singers on delete cascade"},
    }})
    cmd, err := br.Exec()
    if err != nil {
        return err
    }
    if cmd.String() != "CREATE" {
        return fmt.Errorf("unexpected command tag: %v", cmd.String())
    }
    if err := br.Close(); err != nil {
        return err
    }
    fmt.Printf("Created Singers & Albums tables in database: [%s]\n", database)

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function createTables(host: string, port: number, database: string): Promise<void> {
  // Connect to Spanner through PGAdapter.
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Create two tables in one batch.
  await connection.query("start batch ddl");
  await connection.query("create table singers (" +
      "  singer_id   bigint primary key not null," +
      "  first_name  character varying(1024)," +
      "  last_name   character varying(1024)," +
      "  singer_info bytea," +
      "  full_name   character varying(2048) generated " +
      "  always as (first_name || ' ' || last_name) stored" +
      ")");
  await connection.query("create table albums (" +
      "  singer_id     bigint not null," +
      "  album_id      bigint not null," +
      "  album_title   character varying(1024)," +
      "  primary key (singer_id, album_id)" +
      ") interleave in parent singers on delete cascade");
  await connection.query("run batch");
  console.log(`Created Singers & Albums tables in database: [${database}]`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def create_tables(host: string, port: int, database: string):
    # Connect to Cloud Spanner using psycopg3 through PGAdapter.
    with psycopg.connect("host={host} port={port} "
                         "dbname={database} "
                         "sslmode=disable".format(host=host, port=port,
                                                  database=database)) as conn:
        # Enable autocommit to execute DDL statements, as psycopg otherwise
        # tries to use a read/write transaction.
        conn.autocommit = True

        # Use a pipeline to execute multiple DDL statements in one batch.
        with conn.pipeline():
            conn.execute("create table singers ("
                         + "  singer_id   bigint primary key not null,"
                         + "  first_name  character varying(1024),"
                         + "  last_name   character varying(1024),"
                         + "  singer_info bytea,"
                         + "  full_name   character varying(2048) generated "
                         + "  always as (first_name || ' ' || last_name) stored"
                         + ")")
            conn.execute("create table albums ("
                         + "  singer_id     bigint not null,"
                         + "  album_id      bigint not null,"
                         + "  album_title   character varying(1024),"
                         + "  primary key (singer_id, album_id)"
                         + ") interleave in parent singers on delete cascade")
        print("Created Singers & Albums tables in database: [{database}]"
              .format(database=database))
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class CreateTablesSample
{
    public static void CreateTables(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Create two tables in one batch.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add(new NpgsqlBatchCommand(
            "create table singers ("
            + "  singer_id   bigint primary key not null,"
            + "  first_name  varchar(1024),"
            + "  last_name   varchar(1024),"
            + "  singer_info bytea,"
            + "  full_name   varchar(2048) generated always as (\n"
            + "      case when first_name is null then last_name\n"
            + "          when last_name  is null then first_name\n"
            + "          else first_name || ' ' || last_name\n"
            + "      end) stored"
            + ")"));
        batch.BatchCommands.Add(new NpgsqlBatchCommand(
            "create table albums ("
            + "  singer_id     bigint not null,"
            + "  album_id      bigint not null,"
            + "  album_title   varchar,"
            + "  primary key (singer_id, album_id)"
            + ") interleave in parent singers on delete cascade"));
        batch.ExecuteNonQuery();
        Console.WriteLine($"Created Singers & Albums tables in database: [{database}]");
    }
}
```

### PHP

``` php
function create_tables(string $host, string $port, string $database): void
{
    // Connect to Spanner through PGAdapter using the PostgreSQL PDO driver.
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Create two tables in one batch.
    $connection->exec("start batch ddl");
    $connection->exec("create table singers ("
        ."  singer_id   bigint primary key not null,"
        ."  first_name  character varying(1024),"
        ."  last_name   character varying(1024),"
        ."  singer_info bytea,"
        ."  full_name   character varying(2048) generated "
        ."  always as (first_name || ' ' || last_name) stored"
        .")");
    $connection->exec("create table albums ("
        ."  singer_id     bigint not null,"
        ."  album_id      bigint not null,"
        ."  album_title   character varying(1024),"
        ."  primary key (singer_id, album_id)"
        .") interleave in parent singers on delete cascade");
    $connection->exec("run batch");
    print("Created Singers & Albums tables in database: [{$database}]\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./create_tables.sh example-db
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar createtables example-db
```

### Go

``` text
go run sample_runner.go createtables example-db
```

### Node.js

``` text
npm start createtables example-db
```

### Python

``` text
python create_tables.py example-db
```

### C\#

``` text
dotnet run createtables example-db
```

### PHP

``` text
php create_tables.php example-db
```

The next step is to write data to your database.

## Create a connection

Before you can do reads or writes, you must create a connection to PGAdapter. All of your interactions with Spanner must go through a `  Connection  ` . The database name is specified in the connection string.

### psql

``` bash
#!/bin/bash

# Set the connection variables for psql.
# The following statements use the existing value of the variable if it has
# already been set, and otherwise assigns a default value.
export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Connect to Cloud Spanner using psql through PGAdapter
# and execute a simple query.
psql -c "select 'Hello world!' as hello"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

class CreateConnection {
  static void createConnection(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (ResultSet resultSet =
          connection.createStatement().executeQuery("select 'Hello world!' as hello")) {
        while (resultSet.next()) {
          System.out.printf("Greeting from Cloud Spanner PostgreSQL: %s\n", resultSet.getString(1));
        }
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func CreateConnection(host string, port int, database string) error {
    ctx := context.Background()
    // Connect to Cloud Spanner using pgx through PGAdapter.
    // 'sslmode=disable' is optional, but adding it reduces the connection time,
    // as pgx will then skip first trying to create an SSL connection.
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    row := conn.QueryRow(ctx, "select 'Hello world!' as hello")
    var msg string
    if err := row.Scan(&msg); err != nil {
        return err
    }
    fmt.Printf("Greeting from Cloud Spanner PostgreSQL: %s\n", msg)

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function createConnection(host: string, port: number, database: string): Promise<void> {
  // Connect to Spanner through PGAdapter.
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  const result = await connection.query("select 'Hello world!' as hello");
  console.log(`Greeting from Cloud Spanner PostgreSQL: ${result.rows[0]['hello']}`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def create_connection(host: string, port: int, database: string):
    # Connect to Cloud Spanner using psycopg3 through PGAdapter.
    # 'sslmode=disable' is optional, but adding it reduces the connection time,
    # as psycopg3 will then skip first trying to create an SSL connection.
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("select 'Hello world!' as hello")
            print("Greeting from Cloud Spanner PostgreSQL:", cur.fetchone()[0])
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class CreateConnectionSample
{
    public static void CreateConnection(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = new NpgsqlCommand("select 'Hello World!' as hello", connection);
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            var greeting = reader.GetString(0);
            Console.WriteLine($"Greeting from Cloud Spanner PostgreSQL: {greeting}");
        }
    }
}
```

### PHP

``` php
function create_connection(string $host, string $port, string $database): void
{
    // Connect to Spanner through PGAdapter using the PostgreSQL PDO driver.
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Execute a query on Spanner through PGAdapter.
    $statement = $connection->query("select 'Hello world!' as hello");
    $rows = $statement->fetchAll();

    printf("Greeting from Cloud Spanner PostgreSQL: %s\n", $rows[0][0]);

    // Cleanup resources.
    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./create_connection.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar createconnection example-db
```

### Go

``` text
go run sample_runner.go createconnection example-db
```

### Node.js

``` text
npm start createconnection example-db
```

### Python

``` text
python create_connection.py example-db
```

### C\#

``` text
dotnet run createconnection example-db
```

### PHP

``` text
php create_connection.php example-db
```

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

These samples show how to execute a DML statement on Spanner using a PostgreSQL driver.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql -c "INSERT INTO singers (singer_id, first_name, last_name) VALUES
                             (12, 'Melissa', 'Garcia'),
                             (13, 'Russel', 'Morales'),
                             (14, 'Jacqueline', 'Long'),
                             (15, 'Dylan', 'Shaw')"

echo "4 records inserted"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;

class WriteDataWithDml {
  static class Singer {
    private final long singerId;
    private final String firstName;
    private final String lastName;

    Singer(final long id, final String first, final String last) {
      this.singerId = id;
      this.firstName = first;
      this.lastName = last;
    }
  }

  static void writeDataWithDml(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Add 4 rows in one statement.
      // JDBC always uses '?' as a parameter placeholder.
      try (PreparedStatement preparedStatement =
          connection.prepareStatement(
              "INSERT INTO singers (singer_id, first_name, last_name) VALUES "
                  + "(?, ?, ?), "
                  + "(?, ?, ?), "
                  + "(?, ?, ?), "
                  + "(?, ?, ?)")) {

        final List<Singer> singers =
            Arrays.asList(
                new Singer(/* SingerId= */ 12L, "Melissa", "Garcia"),
                new Singer(/* SingerId= */ 13L, "Russel", "Morales"),
                new Singer(/* SingerId= */ 14L, "Jacqueline", "Long"),
                new Singer(/* SingerId= */ 15L, "Dylan", "Shaw"));

        // Note that JDBC parameters start at index 1.
        int paramIndex = 0;
        for (Singer singer : singers) {
          preparedStatement.setLong(++paramIndex, singer.singerId);
          preparedStatement.setString(++paramIndex, singer.firstName);
          preparedStatement.setString(++paramIndex, singer.lastName);
        }

        int updateCount = preparedStatement.executeUpdate();
        System.out.printf("%d records inserted.\n", updateCount);
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func WriteDataWithDml(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    tag, err := conn.Exec(ctx,
        "INSERT INTO singers (singer_id, first_name, last_name) "+
            "VALUES ($1, $2, $3), ($4, $5, $6), "+
            "       ($7, $8, $9), ($10, $11, $12)",
        12, "Melissa", "Garcia",
        13, "Russel", "Morales",
        14, "Jacqueline", "Long",
        15, "Dylan", "Shaw")
    if err != nil {
        return err
    }
    fmt.Printf("%v records inserted\n", tag.RowsAffected())

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function writeDataWithDml(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  const result = await connection.query("INSERT INTO singers (singer_id, first_name, last_name) " +
      "VALUES ($1, $2, $3), ($4, $5, $6), " +
      "       ($7, $8, $9), ($10, $11, $12)",
       [12, "Melissa", "Garcia",
        13, "Russel", "Morales",
        14, "Jacqueline", "Long",
        15, "Dylan", "Shaw"])
  console.log(`${result.rowCount} records inserted`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def write_data_with_dml(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("INSERT INTO singers (singer_id, first_name, last_name)"
                        " VALUES (%s, %s, %s), (%s, %s, %s), "
                        "        (%s, %s, %s), (%s, %s, %s)",
                        (12, "Melissa", "Garcia",
                         13, "Russel", "Morales",
                         14, "Jacqueline", "Long",
                         15, "Dylan", "Shaw",))
            print("%d records inserted" % cur.rowcount)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class WriteDataWithDmlSample
{
    readonly struct Singer
    {
        public Singer(long singerId, string firstName, string lastName)
        {
            SingerId = singerId;
            FirstName = firstName;
            LastName = lastName;
        }

        public long SingerId { get; }
        public string FirstName { get; }
        public string LastName { get; }
    }

    public static void WriteDataWithDml(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();
        // Add 4 rows in one statement.
        using var cmd = new NpgsqlCommand("INSERT INTO singers (singer_id, first_name, last_name) VALUES "
                                          + "($1, $2, $3), "
                                          + "($4, $5, $6), "
                                          + "($7, $8, $9), "
                                          + "($10, $11, $12)", connection);
        List<Singer> singers =
        [
            new Singer(/* SingerId = */ 12L, "Melissa", "Garcia"),
            new Singer(/* SingerId = */ 13L, "Russel", "Morales"),
            new Singer(/* SingerId = */ 14L, "Jacqueline", "Long"),
            new Singer(/* SingerId = */ 15L, "Dylan", "Shaw")
        ];
        foreach (var singer in singers)
        {
            cmd.Parameters.Add(new NpgsqlParameter { Value = singer.SingerId });
            cmd.Parameters.Add(new NpgsqlParameter { Value = singer.FirstName });
            cmd.Parameters.Add(new NpgsqlParameter { Value = singer.LastName });
        }
        var updateCount = cmd.ExecuteNonQuery();
        Console.WriteLine($"{updateCount} records inserted.");
    }
}
```

### PHP

``` php
function write_data_with_dml(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $sql = "INSERT INTO singers (singer_id, first_name, last_name)"
                        ." VALUES (?, ?, ?), (?, ?, ?), "
                        ."        (?, ?, ?), (?, ?, ?)";
    $statement = $connection->prepare($sql);
    $statement->execute([
        12, "Melissa", "Garcia",
        13, "Russel", "Morales",
        14, "Jacqueline", "Long",
        15, "Dylan", "Shaw"
    ]);
    printf("%d records inserted\n", $statement->rowCount());

    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./write_data_with_dml.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar writeusingdml example-db
```

### Go

``` text
go run sample_runner.go writeusingdml example-db
```

### Node.js

``` text
npm start writeusingdml example-db
```

### Python

``` text
python write_data_with_dml.py example-db
```

### C\#

``` text
dotnet run writeusingdml example-db
```

### PHP

``` text
php write_data_with_dml.php example-db
```

You should see the following response:

``` text
 4 records inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with a DML batch

PGAdapter supports executing DML batches. Sending multiple DML statements in one batch reduces the number of round-trips to Spanner and improves the performance of your application.

**Tip:** You can also execute DML batches with the [`  START BATCH DML  ` command](/spanner/docs/pgadapter-session-mgmt-commands#start_batch_dml) .

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Create a prepared insert statement and execute this prepared
# insert statement three times in one SQL string. The single
# SQL string with three insert statements will be executed as
# a single DML batch on Spanner.
psql -c "PREPARE insert_singer AS
           INSERT INTO singers (singer_id, first_name, last_name)
           VALUES (\$1, \$2, \$3)" \
     -c "EXECUTE insert_singer (16, 'Sarah', 'Wilson');
         EXECUTE insert_singer (17, 'Ethan', 'Miller');
         EXECUTE insert_singer (18, 'Maya', 'Patel');"

echo "3 records inserted"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;

class WriteDataWithDmlBatch {
  static class Singer {
    private final long singerId;
    private final String firstName;
    private final String lastName;

    Singer(final long id, final String first, final String last) {
      this.singerId = id;
      this.firstName = first;
      this.lastName = last;
    }
  }

  static void writeDataWithDmlBatch(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Add multiple rows in one DML batch.
      // JDBC always uses '?' as a parameter placeholder.
      try (PreparedStatement preparedStatement =
          connection.prepareStatement(
              "INSERT INTO singers (singer_id, first_name, last_name) VALUES (?, ?, ?)")) {
        final List<Singer> singers =
            Arrays.asList(
                new Singer(/* SingerId= */ 16L, "Sarah", "Wilson"),
                new Singer(/* SingerId= */ 17L, "Ethan", "Miller"),
                new Singer(/* SingerId= */ 18L, "Maya", "Patel"));

        for (Singer singer : singers) {
          // Note that JDBC parameters start at index 1.
          int paramIndex = 0;
          preparedStatement.setLong(++paramIndex, singer.singerId);
          preparedStatement.setString(++paramIndex, singer.firstName);
          preparedStatement.setString(++paramIndex, singer.lastName);
          preparedStatement.addBatch();
        }

        int[] updateCounts = preparedStatement.executeBatch();
        System.out.printf("%d records inserted.\n", Arrays.stream(updateCounts).sum());
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func WriteDataWithDmlBatch(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    sql := "INSERT INTO singers (singer_id, first_name, last_name) " +
        "VALUES ($1, $2, $3)"
    batch := &pgx.Batch{}
    batch.Queue(sql, 16, "Sarah", "Wilson")
    batch.Queue(sql, 17, "Ethan", "Miller")
    batch.Queue(sql, 18, "Maya", "Patel")
    br := conn.SendBatch(ctx, batch)
    _, err = br.Exec()
    if err := br.Close(); err != nil {
        return err
    }

    if err != nil {
        return err
    }
    fmt.Printf("%v records inserted\n", batch.Len())

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function writeDataWithDmlBatch(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // node-postgres does not support PostgreSQL pipeline mode, so we must use the
  // `start batch dml` / `run batch` statements to execute a DML batch.
  const sql = "INSERT INTO singers (singer_id, first_name, last_name) VALUES ($1, $2, $3)";
  await connection.query("start batch dml");
  await connection.query(sql, [16, "Sarah", "Wilson"]);
  await connection.query(sql, [17, "Ethan", "Miller"]);
  await connection.query(sql, [18, "Maya", "Patel"]);
  const result = await connection.query("run batch");
  // RUN BATCH returns the update counts as an array of strings, with one element for each
  // DML statement in the batch. This calculates the total number of affected rows from that array.
  const updateCount = result.rows[0]["UPDATE_COUNTS"]
      .map((s: string) => parseInt(s))
      .reduce((c: number, current: number) => c + current, 0);
  console.log(`${updateCount} records inserted`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def write_data_with_dml_batch(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.executemany("INSERT INTO singers "
                            "(singer_id, first_name, last_name) "
                            "VALUES (%s, %s, %s)",
                            [(16, "Sarah", "Wilson",),
                             (17, "Ethan", "Miller",),
                             (18, "Maya", "Patel",), ])
            print("%d records inserted" % cur.rowcount)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class WriteDataWithDmlBatchSample
{
    readonly struct Singer
    {
        public Singer(long singerId, string firstName, string lastName)
        {
            SingerId = singerId;
            FirstName = firstName;
            LastName = lastName;
        }

        public long SingerId { get; }
        public string FirstName { get; }
        public string LastName { get; }
    }

    public static void WriteDataWithDmlBatch(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Add multiple rows in one DML batch.
        const string sql = "INSERT INTO singers (singer_id, first_name, last_name) VALUES ($1, $2, $3)";
        List<Singer> singers =
        [
            new Singer(/* SingerId = */ 16L, "Sarah", "Wilson"),
            new Singer(/* SingerId = */ 17L, "Ethan", "Miller"),
            new Singer(/* SingerId = */ 18L, "Maya", "Patel")
        ];
        using var batch = new NpgsqlBatch(connection);
        foreach (var singer in singers)
        {
            batch.BatchCommands.Add(new NpgsqlBatchCommand
            {
                CommandText = sql,
                Parameters =
                {
                    new NpgsqlParameter {Value = singer.SingerId},
                    new NpgsqlParameter {Value = singer.FirstName},
                    new NpgsqlParameter {Value = singer.LastName}
                }
            });
        }
        var updateCount = batch.ExecuteNonQuery();
        Console.WriteLine($"{updateCount} records inserted.");
    }
}
```

### PHP

``` php
function write_data_with_dml_batch(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Use START BATCH DML / RUN BATCH to run a batch of DML statements.
    // Create a prepared statement for the DML that should be executed.
    $sql = "INSERT INTO singers (singer_id, first_name, last_name) VALUES (?, ?, ?)";
    $statement = $connection->prepare($sql);
    // Start a DML batch.
    $connection->exec("START BATCH DML");

    $statement->execute([16, "Sarah", "Wilson"]);
    $statement->execute([17, "Ethan", "Miller"]);
    $statement->execute([18, "Maya", "Patel"]);

    // Run the DML batch. Use the 'query(..)' method, as the update counts are returned as a row
    // containing an array with the update count of each statement in the batch.
    $statement = $connection->query("RUN BATCH");
    $result = $statement->fetchAll();
    $update_count = $result[0][0];

    printf("%s records inserted\n", $update_count);

    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./write_data_with_dml_batch.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar writeusingdmlbatch example-db
```

### Go

``` text
go run sample_runner.go writeusingdmlbatch example-db
```

### Node.js

``` text
npm start writeusingdmlbatch example-db
```

### Python

``` text
python write_data_with_dml_batch.py example-db
```

### C\#

``` text
dotnet run writeusingdmlbatch example-db
```

### PHP

``` text
php write_data_with_dml_batch.php example-db
```

You should see:

``` text
3 records inserted.
```

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

PGAdapter translates the PostgreSQL `  COPY  ` command to mutations. Using `  COPY  ` is the most efficient way to quickly insert data in your Spanner database.

`  COPY  ` operations are by default atomic. Atomic operations on Spanner are bound by the commit size limit. See [CRUD limit](/spanner/quotas#limits-for) for more information.

These examples show how to execute a non-atomic `  COPY  ` operation. This lets the `  COPY  ` operation exceed the commit size limit.

**Tip:** See [COPY support](https://github.com/GoogleCloudPlatform/pgadapter/blob/postgresql-dialect/docs/copy.md) for more information on how to copy data directly from PostgreSQL to Spanner. You can also [import data directly from MySQL](https://github.com/GoogleCloudPlatform/pgadapter/blob/postgresql-dialect/docs/import-mysql-data.md) using the `  COPY  ` command.

### psql

``` bash
#!/bin/bash

# Get the source directory of this script.
directory=${BASH_SOURCE%/*}/

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Copy data to Spanner from a tab-separated text file using the COPY command.
psql -c "COPY singers (singer_id, first_name, last_name) FROM STDIN" \
  < "${directory}singers_data.txt"
psql -c "COPY albums FROM STDIN" \
  < "${directory}albums_data.txt"

echo "Copied singers and albums"
```

### Java

``` java
import java.io.IOException;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import org.postgresql.PGConnection;
import org.postgresql.copy.CopyManager;

class WriteDataWithCopy {

  static void writeDataWithCopy(String host, int port, String database)
      throws SQLException, IOException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Unwrap the PostgreSQL JDBC connection interface to get access to
      // a CopyManager.
      PGConnection pgConnection = connection.unwrap(PGConnection.class);
      CopyManager copyManager = pgConnection.getCopyAPI();

      // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
      // will succeed even if it exceeds Spanner's mutation limit per transaction.
      connection
          .createStatement()
          .execute("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
      long numSingers =
          copyManager.copyIn(
              "COPY singers (singer_id, first_name, last_name) FROM STDIN",
              WriteDataWithCopy.class.getResourceAsStream("singers_data.txt"));
      System.out.printf("Copied %d singers\n", numSingers);

      long numAlbums =
          copyManager.copyIn(
              "COPY albums FROM STDIN",
              WriteDataWithCopy.class.getResourceAsStream("albums_data.txt"));
      System.out.printf("Copied %d albums\n", numAlbums);
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"
    "os"

    "github.com/jackc/pgx/v5"
)

func WriteDataWithCopy(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
    // will succeed even if it exceeds Spanner's mutation limit per transaction.
    conn.Exec(ctx, "set spanner.autocommit_dml_mode='partitioned_non_atomic'")

    file, err := os.Open("samples/singers_data.txt")
    if err != nil {
        return err
    }
    tag, err := conn.PgConn().CopyFrom(ctx, file,
        "copy singers (singer_id, first_name, last_name) from stdin")
    if err != nil {
        return err
    }
    fmt.Printf("Copied %v singers\n", tag.RowsAffected())

    file, err = os.Open("samples/albums_data.txt")
    if err != nil {
        return err
    }
    tag, err = conn.PgConn().CopyFrom(ctx, file,
        "copy albums from stdin")
    if err != nil {
        return err
    }
    fmt.Printf("Copied %v albums\n", tag.RowsAffected())

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';
import { pipeline } from 'node:stream/promises'
import fs from 'node:fs'
import { from as copyFrom } from 'pg-copy-streams'
import path from "path";

async function writeDataWithCopy(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
  // will succeed even if it exceeds Spanner's mutation limit per transaction.
  await connection.query("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
  // Copy data from a csv file to Spanner using the COPY command.
  // Note that even though the command says 'from stdin', the actual input comes from a file.
  const copySingersStream = copyFrom('copy singers (singer_id, first_name, last_name) from stdin');
  const ingestSingersStream = connection.query(copySingersStream);
  const sourceSingersStream = fs.createReadStream(path.join(__dirname, 'singers_data.txt'));
  await pipeline(sourceSingersStream, ingestSingersStream);
  console.log(`Copied ${copySingersStream.rowCount} singers`);

  const copyAlbumsStream = copyFrom('copy albums from stdin');
  const ingestAlbumsStream = connection.query(copyAlbumsStream);
  const sourceAlbumsStream = fs.createReadStream(path.join(__dirname, 'albums_data.txt'));
  await pipeline(sourceAlbumsStream, ingestAlbumsStream);
  console.log(`Copied ${copyAlbumsStream.rowCount} albums`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import os
import string
import psycopg


def write_data_with_copy(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:

        script_dir = os.path.dirname(os.path.abspath(__file__))
        singers_file_path = os.path.join(script_dir, "singers_data.txt")
        albums_file_path = os.path.join(script_dir, "albums_data.txt")

        conn.autocommit = True
        block_size = 1024
        with conn.cursor() as cur:
            with open(singers_file_path, "r") as f:
                with cur.copy("COPY singers (singer_id, first_name, last_name) "
                              "FROM STDIN") as copy:
                    while data := f.read(block_size):
                        copy.write(data)
            print("Copied %d singers" % cur.rowcount)

            with open(albums_file_path, "r") as f:
                with cur.copy("COPY albums "
                              "FROM STDIN") as copy:
                    while data := f.read(block_size):
                        copy.write(data)
            print("Copied %d albums" % cur.rowcount)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class WriteDataWithCopySample
{
    public static void WriteDataWithCopy(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
        // will succeed even if it exceeds Spanner's mutation limit per transaction.
        using var cmd = new NpgsqlCommand("set spanner.autocommit_dml_mode='partitioned_non_atomic'", connection);
        cmd.ExecuteNonQuery();

        var singerCount = 0;
        using var singerReader = new StreamReader("singers_data.txt");
        using (var singerWriter = connection.BeginTextImport("COPY singers (singer_id, first_name, last_name) FROM STDIN"))
        {
            while (singerReader.ReadLine() is { } line)
            {
                singerWriter.WriteLine(line);
                singerCount++;
            }
        }
        Console.WriteLine($"Copied {singerCount} singers");

        var albumCount = 0;
        using var albumReader = new StreamReader("albums_data.txt");
        using (var albumWriter = connection.BeginTextImport("COPY albums FROM STDIN"))
        {
            while (albumReader.ReadLine() is { } line)
            {
                albumWriter.WriteLine(line);
                albumCount++;
            }
        }
        Console.WriteLine($"Copied {albumCount} albums");
    }
}
```

### PHP

``` php
function write_data_with_copy(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $dir = dirname(__FILE__);

    $connection->pgsqlCopyFromFile(
        "singers",
        sprintf("%s/singers_data.txt", $dir),
        "\t",
        "\\\\N",
        "singer_id, first_name, last_name");
    print("Copied 5 singers\n");

    $connection->pgsqlCopyFromFile(
        "albums",
        sprintf("%s/albums_data.txt", $dir));
    print("Copied 5 albums\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./write_data_with_copy.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar write example-db
```

### Go

``` text
go run sample_runner.go write example-db
```

### Node.js

``` text
npm start write example-db
```

### Python

``` text
python write_data_with_copy.py example-db
```

### C\#

``` text
dotnet run write example-db
```

### PHP

``` text
php write_data_with_copy.php example-db
```

You should see:

``` text
Copied 5 singers
Copied 5 albums
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using a PostgreSQL driver.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT singer_id, album_id, album_title FROM albums'
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

### Use a PostgreSQL driver

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using a PostgreSQL driver.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql -c "SELECT singer_id, album_id, album_title
         FROM albums"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

class QueryData {
  static void queryData(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (ResultSet resultSet =
          connection
              .createStatement()
              .executeQuery("SELECT singer_id, album_id, album_title FROM albums")) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %d %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getLong("album_id"),
              resultSet.getString("album_title"));
        }
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func QueryData(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    rows, err := conn.Query(ctx, "SELECT singer_id, album_id, album_title "+
        "FROM albums")
    defer rows.Close()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId, albumId int64
        var title string
        err = rows.Scan(&singerId, &albumId, &title)
        if err != nil {
            return err
        }
        fmt.Printf("%v %v %v\n", singerId, albumId, title)
    }

    return rows.Err()
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function queryData(host: string, port: number, database: string): Promise<void> {
  // Connect to Spanner through PGAdapter.
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  const result = await connection.query("SELECT singer_id, album_id, album_title " +
      "FROM albums");
  for (const row of result.rows) {
    console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
  }

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def query_data(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("SELECT singer_id, album_id, album_title "
                        "FROM albums")
            for album in cur:
                print(album)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class QueryDataSample
{
    public static void QueryData(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = new NpgsqlCommand("SELECT singer_id, album_id, album_title FROM albums", connection);
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine($"{reader.GetInt64(0)} {reader.GetInt64(1)} {reader.GetString(2)}");
        }
    }
}
```

### PHP

``` php
function query_data(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $statement = $connection->query("SELECT singer_id, album_id, album_title "
        ."FROM albums "
        ."ORDER BY singer_id, album_id"
    );
    $rows = $statement->fetchAll();
    foreach ($rows as $album)
    {
        printf("%s\t%s\t%s\n", $album["singer_id"], $album["album_id"], $album["album_title"]);
    }

    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./query_data.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar query example-db
```

### Go

``` text
go run sample_runner.go query example-db
```

### Node.js

``` text
npm start query example-db
```

### Python

``` text
python query_data.py example-db
```

### C\#

``` text
dotnet run query example-db
```

### PHP

``` text
php query_data.php example-db
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

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Create a prepared statement to use a query parameter.
# Using a prepared statement for executing the same SQL string multiple
# times increases the execution speed of the statement.
psql -c "PREPARE select_singer AS
         SELECT singer_id, first_name, last_name
         FROM singers
         WHERE last_name = \$1" \
     -c "EXECUTE select_singer ('Garcia')"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

class QueryDataWithParameter {
  static void queryDataWithParameter(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (PreparedStatement statement =
          connection.prepareStatement(
              "SELECT singer_id, first_name, last_name "
                  + "FROM singers "
                  + "WHERE last_name = ?")) {
        statement.setString(1, "Garcia");
        try (ResultSet resultSet = statement.executeQuery()) {
          while (resultSet.next()) {
            System.out.printf(
                "%d %s %s\n",
                resultSet.getLong("singer_id"),
                resultSet.getString("first_name"),
                resultSet.getString("last_name"));
          }
        }
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func QueryDataWithParameter(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    rows, err := conn.Query(ctx,
        "SELECT singer_id, first_name, last_name "+
            "FROM singers "+
            "WHERE last_name = $1", "Garcia")
    defer rows.Close()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId int64
        var firstName, lastName string
        err = rows.Scan(&singerId, &firstName, &lastName)
        if err != nil {
            return err
        }
        fmt.Printf("%v %v %v\n", singerId, firstName, lastName)
    }

    return rows.Err()
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function queryWithParameter(host: string, port: number, database: string): Promise<void> {
  // Connect to Spanner through PGAdapter.
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  const result = await connection.query(
      "SELECT singer_id, first_name, last_name " +
      "FROM singers " +
      "WHERE last_name = $1", ["Garcia"]);
  for (const row of result.rows) {
    console.log(`${row["singer_id"]} ${row["first_name"]} ${row["last_name"]}`);
  }

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def query_data_with_parameter(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("SELECT singer_id, first_name, last_name "
                        "FROM singers "
                        "WHERE last_name = %s", ("Garcia",))
            for singer in cur:
                print(singer)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class QueryDataWithParameterSample
{
    public static void QueryDataWithParameter(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = new NpgsqlCommand("SELECT singer_id, first_name, last_name "
                                          + "FROM singers "
                                          + "WHERE last_name = $1", connection);
        cmd.Parameters.Add(new NpgsqlParameter { Value = "Garcia" });
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["first_name"]} {reader["last_name"]}");
        }
    }
}
```

### PHP

``` php
function query_data_with_parameter(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $statement = $connection->prepare("SELECT singer_id, first_name, last_name "
                        ."FROM singers "
                        ."WHERE last_name = ?"
    );
    $statement->execute(["Garcia"]);
    $rows = $statement->fetchAll();
    foreach ($rows as $singer)
    {
        printf("%s\t%s\t%s\n", $singer["singer_id"], $singer["first_name"], $singer["last_name"]);
    }

    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./query_data_with_parameter.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar querywithparameter example-db
```

### Go

``` text
go run sample_runner.go querywithparameter example-db
```

### Node.js

``` text
npm start querywithparameter example-db
```

### Python

``` text
python query_data_with_parameter.py example-db
```

### C\#

``` text
dotnet run querywithparameter example-db
```

### PHP

``` text
php query_data_with_parameter.php example-db
```

You should see the following result:

``` text
12 Melissa Garcia
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using a PostgreSQL driver.

#### On the command line

Use the following [`  ALTER TABLE  `](/spanner/docs/reference/standard-sql/data-definition-language#alter_table) command to add the new column to the table:

``` text
gcloud spanner databases ddl update example-db --instance=test-instance \
    --ddl='ALTER TABLE albums ADD COLUMN marketing_budget BIGINT'
```

You should see:

``` text
Schema updating...done.
```

#### Use a PostgreSQL driver

Execute the DDL statement using a PostgreSQL driver to modify the schema:

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql -c "ALTER TABLE albums ADD COLUMN marketing_budget bigint"
echo "Added marketing_budget column"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

class AddColumn {
  static void addColumn(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      connection.createStatement().execute("alter table albums add column marketing_budget bigint");
      System.out.println("Added marketing_budget column");
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func AddColumn(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    _, err = conn.Exec(ctx,
        "ALTER TABLE albums "+
            "ADD COLUMN marketing_budget bigint")
    if err != nil {
        return err
    }
    fmt.Println("Added marketing_budget column")

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function addColumn(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  await connection.query(
      "ALTER TABLE albums " +
      "ADD COLUMN marketing_budget bigint");
  console.log("Added marketing_budget column");

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def add_column(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        # DDL can only be executed when autocommit=True.
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("ALTER TABLE albums "
                        "ADD COLUMN marketing_budget bigint")
            print("Added marketing_budget column")
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class AddColumnSample
{
    public static void AddColumn(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = connection.CreateCommand();
        cmd.CommandText = "alter table albums add column marketing_budget bigint";
        cmd.ExecuteNonQuery();
        Console.WriteLine("Added marketing_budget column");
    }
}
```

### PHP

``` php
function add_column(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $connection->exec("ALTER TABLE albums ADD COLUMN marketing_budget bigint");
    print("Added marketing_budget column\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./add_column.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar addmarketingbudget example-db
```

### Go

``` text
go run sample_runner.go addmarketingbudget example-db
```

### Node.js

``` text
npm start addmarketingbudget example-db
```

### Python

``` text
python add_column.py example-db
```

### C\#

``` text
dotnet run addmarketingbudget example-db
```

### PHP

``` text
php add_column.php example-db
```

You should see:

``` text
Added marketing_budget column
```

### Execute a DDL batch

It is recommended to execute multiple schema modifications in one batch. You can execute multiple DDL statements in one batch by using the built-in batching feature of your PostgreSQL driver, by submitting all the DDL statements as one SQL string separated by semicolons, or by using the `  START BATCH DDL  ` and `  RUN BATCH  ` statements.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Use a single SQL command to batch multiple statements together.
# Executing multiple DDL statements as one batch is more efficient
# than executing each statement individually.
# Separate the statements with semicolons.
psql << SQL

CREATE TABLE venues (
  venue_id    bigint not null primary key,
  name        varchar(1024),
  description jsonb
);

CREATE TABLE concerts (
  concert_id bigint not null primary key ,
  venue_id   bigint not null,
  singer_id  bigint not null,
  start_time timestamptz,
  end_time   timestamptz,
  constraint fk_concerts_venues foreign key
    (venue_id) references venues (venue_id),
  constraint fk_concerts_singers foreign key
    (singer_id) references singers (singer_id)
);

SQL

echo "Added venues and concerts tables"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

class DdlBatch {
  static void ddlBatch(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (Statement statement = connection.createStatement()) {
        // Create two new tables in one batch.
        statement.addBatch(
            "CREATE TABLE venues ("
                + "  venue_id    bigint not null primary key,"
                + "  name        varchar(1024),"
                + "  description jsonb"
                + ")");
        statement.addBatch(
            "CREATE TABLE concerts ("
                + "  concert_id bigint not null primary key ,"
                + "  venue_id   bigint not null,"
                + "  singer_id  bigint not null,"
                + "  start_time timestamptz,"
                + "  end_time   timestamptz,"
                + "  constraint fk_concerts_venues foreign key"
                + "    (venue_id) references venues (venue_id),"
                + "  constraint fk_concerts_singers foreign key"
                + "    (singer_id) references singers (singer_id)"
                + ")");
        statement.executeBatch();
      }
      System.out.println("Added venues and concerts tables");
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func DdlBatch(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Executing multiple DDL statements as one batch is
    // more efficient than executing each statement
    // individually.
    br := conn.SendBatch(ctx, &pgx.Batch{QueuedQueries: []*pgx.QueuedQuery{
        {SQL: "CREATE TABLE venues (" +
            "  venue_id    bigint not null primary key," +
            "  name        varchar(1024)," +
            "  description jsonb" +
            ")"},
        {SQL: "CREATE TABLE concerts (" +
            "  concert_id bigint not null primary key ," +
            "  venue_id   bigint not null," +
            "  singer_id  bigint not null," +
            "  start_time timestamptz," +
            "  end_time   timestamptz," +
            "  constraint fk_concerts_venues foreign key" +
            "    (venue_id) references venues (venue_id)," +
            "  constraint fk_concerts_singers foreign key" +
            "    (singer_id) references singers (singer_id)" +
            ")"},
    }})
    if _, err := br.Exec(); err != nil {
        return err
    }
    if err := br.Close(); err != nil {
        return err
    }
    fmt.Println("Added venues and concerts tables")

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function ddlBatch(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Executing multiple DDL statements as one batch is
  // more efficient than executing each statement
  // individually.
  await connection.query("start batch ddl");
  await connection.query("CREATE TABLE venues (" +
      "  venue_id    bigint not null primary key," +
      "  name        varchar(1024)," +
      "  description jsonb" +
      ")");
  await connection.query("CREATE TABLE concerts (" +
      "  concert_id bigint not null primary key ," +
      "  venue_id   bigint not null," +
      "  singer_id  bigint not null," +
      "  start_time timestamptz," +
      "  end_time   timestamptz," +
      "  constraint fk_concerts_venues foreign key" +
      "    (venue_id) references venues (venue_id)," +
      "  constraint fk_concerts_singers foreign key" +
      "    (singer_id) references singers (singer_id)" +
      ")");
  await connection.query("run batch");
  console.log("Added venues and concerts tables");

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def ddl_batch(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        # DDL can only be executed when autocommit=True.
        conn.autocommit = True
        # Use a pipeline to batch multiple statements together.
        # Executing multiple DDL statements as one batch is
        # more efficient than executing each statement
        # individually.
        with conn.pipeline():
            # The following statements are buffered on PGAdapter
            # until the pipeline ends.
            conn.execute("CREATE TABLE venues ("
                         "  venue_id    bigint not null primary key,"
                         "  name        varchar(1024),"
                         "  description jsonb"
                         ")")
            conn.execute("CREATE TABLE concerts ("
                         "  concert_id bigint not null primary key ,"
                         "  venue_id   bigint not null,"
                         "  singer_id  bigint not null,"
                         "  start_time timestamptz,"
                         "  end_time   timestamptz,"
                         "  constraint fk_concerts_venues foreign key"
                         "    (venue_id) references venues (venue_id),"
                         "  constraint fk_concerts_singers foreign key"
                         "    (singer_id) references singers (singer_id)"
                         ")")
        print("Added venues and concerts tables")
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class DdlBatchSample
{
    public static void DdlBatch(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Create two new tables in one batch.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add(new NpgsqlBatchCommand(
            "CREATE TABLE venues ("
            + "  venue_id    bigint not null primary key,"
            + "  name        varchar(1024),"
            + "  description jsonb"
            + ")"));
        batch.BatchCommands.Add(new NpgsqlBatchCommand(
            "CREATE TABLE concerts ("
            + "  concert_id bigint not null primary key ,"
            + "  venue_id   bigint not null,"
            + "  singer_id  bigint not null,"
            + "  start_time timestamptz,"
            + "  end_time   timestamptz,"
            + "  constraint fk_concerts_venues foreign key"
            + "    (venue_id) references venues (venue_id),"
            + "  constraint fk_concerts_singers foreign key"
            + "    (singer_id) references singers (singer_id)"
            + ")"));
        batch.ExecuteNonQuery();
        Console.WriteLine("Added venues and concerts tables");
    }
}
```

### PHP

``` php
function ddl_batch(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Executing multiple DDL statements as one batch is
    // more efficient than executing each statement
    // individually.
    $connection->exec("start batch ddl");
    $connection->exec("CREATE TABLE venues ("
        ."  venue_id    bigint not null primary key,"
        ."  name        varchar(1024),"
        ."  description jsonb"
        .")");
    $connection->exec("CREATE TABLE concerts ("
        ."  concert_id bigint not null primary key ,"
        ."  venue_id   bigint not null,"
        ."  singer_id  bigint not null,"
        ."  start_time timestamptz,"
        ."  end_time   timestamptz,"
        ."  constraint fk_concerts_venues foreign key"
        ."    (venue_id) references venues (venue_id),"
        ."  constraint fk_concerts_singers foreign key"
        ."    (singer_id) references singers (singer_id)"
        .")");
    $connection->exec("run batch");
    print("Added venues and concerts tables\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./ddl_batch.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar ddlbatch example-db
```

### Go

``` text
go run sample_runner.go ddlbatch example-db
```

### Node.js

``` text
npm start ddlbatch example-db
```

### Python

``` text
python ddl_batch.py example-db
```

### C\#

``` text
dotnet run ddlbatch example-db
```

### PHP

``` text
php ddl_batch.php example-db
```

You should see:

``` text
Added venues and concerts tables
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

PGAdapter translates the PostgreSQL `  COPY  ` command to mutations. `  COPY  ` commands are by default translated to `  Insert  ` mutations. Execute `  set spanner.copy_upsert=true  ` to translate `  COPY  ` commands to `  InsertOrUpdate  ` mutations. This can be used to update existing data in Spanner.

**Tip:** For a full list of commands that can be used to access Spanner features with PGAdapter, see [PGAdapter session management commands](/spanner/docs/pgadapter-session-mgmt-commands) .

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Instruct PGAdapter to use insert-or-update for COPY statements.
# This enables us to use COPY to update data.
psql -c "set spanner.copy_upsert=true" \
     -c "COPY albums (singer_id, album_id, marketing_budget) FROM STDIN
         WITH (DELIMITER ';')" \
<< DATA
1;1;100000
2;2;500000
DATA

echo "Copied albums using upsert"
```

### Java

``` java
import java.io.IOException;
import java.io.StringReader;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import org.postgresql.PGConnection;
import org.postgresql.copy.CopyManager;

class UpdateDataWithCopy {

  static void updateDataWithCopy(String host, int port, String database)
      throws SQLException, IOException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Unwrap the PostgreSQL JDBC connection interface to get access to
      // a CopyManager.
      PGConnection pgConnection = connection.unwrap(PGConnection.class);
      CopyManager copyManager = pgConnection.getCopyAPI();

      // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
      // will succeed even if it exceeds Spanner's mutation limit per transaction.
      connection
          .createStatement()
          .execute("set spanner.autocommit_dml_mode='partitioned_non_atomic'");

      // Instruct PGAdapter to use insert-or-update for COPY statements.
      // This enables us to use COPY to update existing data.
      connection.createStatement().execute("set spanner.copy_upsert=true");

      // COPY uses mutations to insert or update existing data in Spanner.
      long numAlbums =
          copyManager.copyIn(
              "COPY albums (singer_id, album_id, marketing_budget) FROM STDIN",
              new StringReader("1\t1\t100000\n" + "2\t2\t500000\n"));
      System.out.printf("Updated %d albums\n", numAlbums);
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"
    "io"

    "github.com/jackc/pgx/v5"
)

func UpdateDataWithCopy(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Enable non-atomic mode. This makes the COPY operation non-atomic,
    // and allows it to exceed the Spanner mutation limit.
    if _, err := conn.Exec(ctx,
        "set spanner.autocommit_dml_mode='partitioned_non_atomic'"); err != nil {
        return err
    }
    // Instruct PGAdapter to use insert-or-update for COPY statements.
    // This enables us to use COPY to update data.
    if _, err := conn.Exec(ctx, "set spanner.copy_upsert=true"); err != nil {
        return err
    }

    // Create a pipe that can be used to write the data manually that we want to copy.
    reader, writer := io.Pipe()
    // Write the data to the pipe using a separate goroutine. This allows us to stream the data
    // to the COPY operation row-by-row.
    go func() error {
        for _, record := range []string{"1\t1\t100000\n", "2\t2\t500000\n"} {
            if _, err := writer.Write([]byte(record)); err != nil {
                return err
            }
        }
        if err := writer.Close(); err != nil {
            return err
        }
        return nil
    }()
    tag, err := conn.PgConn().CopyFrom(ctx, reader, "COPY albums (singer_id, album_id, marketing_budget) FROM STDIN")
    if err != nil {
        return err
    }
    fmt.Printf("Updated %v albums\n", tag.RowsAffected())

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';
import { pipeline } from 'node:stream/promises'
import { from as copyFrom } from 'pg-copy-streams'
import {Readable} from "stream";

async function updateDataWithCopy(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
  // will succeed even if it exceeds Spanner's mutation limit per transaction.
  await connection.query("set spanner.autocommit_dml_mode='partitioned_non_atomic'");

  // Instruct PGAdapter to use insert-or-update for COPY statements.
  // This enables us to use COPY to update existing data.
  await connection.query("set spanner.copy_upsert=true");

  // Copy data to Spanner using the COPY command.
  const copyStream = copyFrom('COPY albums (singer_id, album_id, marketing_budget) FROM STDIN');
  const ingestStream = connection.query(copyStream);

  // Create a source stream and attach the source to the destination.
  const sourceStream = new Readable();
  const operation = pipeline(sourceStream, ingestStream);
  // Manually push data to the source stream to write data to Spanner.
  sourceStream.push("1\t1\t100000\n");
  sourceStream.push("2\t2\t500000\n");
  // Push a 'null' to indicate the end of the stream.
  sourceStream.push(null);
  // Wait for the copy operation to finish.
  await operation;
  console.log(`Updated ${copyStream.rowCount} albums`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def update_data_with_copy(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            # Instruct PGAdapter to use insert-or-update for COPY statements.
            # This enables us to use COPY to update data.
            cur.execute("set spanner.copy_upsert=true")

            # COPY uses mutations to insert or update existing data in Spanner.
            with cur.copy("COPY albums (singer_id, album_id, marketing_budget) "
                          "FROM STDIN") as copy:
                copy.write_row((1, 1, 100000))
                copy.write_row((2, 2, 500000))
            print("Updated %d albums" % cur.rowcount)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class UpdateDataWithCopySample
{
    public static void UpdateDataWithCopy(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Enable 'partitioned_non_atomic' mode. This ensures that the COPY operation
        // will succeed even if it exceeds Spanner's mutation limit per transaction.
        using var cmd = connection.CreateCommand();
        cmd.CommandText = "set spanner.autocommit_dml_mode='partitioned_non_atomic'";
        cmd.ExecuteNonQuery();

        // Instruct PGAdapter to use insert-or-update for COPY statements.
        // This enables us to use COPY to update existing data.
        cmd.CommandText = "set spanner.copy_upsert=true";
        cmd.ExecuteNonQuery();

        // COPY uses mutations to insert or update existing data in Spanner.
        using (var albumWriter = connection.BeginTextImport(
                   "COPY albums (singer_id, album_id, marketing_budget) FROM STDIN"))
        {
            albumWriter.WriteLine("1\t1\t100000");
            albumWriter.WriteLine("2\t2\t500000");
        }
        Console.WriteLine($"Updated 2 albums");
    }
}
```

### PHP

``` php
function update_data_with_copy(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Instruct PGAdapter to use insert-or-update for COPY statements.
    // This enables us to use COPY to update data.
    $connection->exec("set spanner.copy_upsert=true");

    // COPY uses mutations to insert or update existing data in Spanner.
    $connection->pgsqlCopyFromArray(
        "albums",
        ["1\t1\t100000", "2\t2\t500000"],
        "\t",
        "\\\\N",
        "singer_id, album_id, marketing_budget",
    );
    print("Updated 2 albums\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./update_data_with_copy.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar update example-db
```

### Go

``` text
go run sample_runner.go update example-db
```

### Node.js

``` text
npm start update example-db
```

### Python

``` text
python update_data_with_copy.py example-db
```

### C\#

``` text
dotnet run update example-db
```

### PHP

``` text
php update_data_with_copy.php example-db
```

You should see:

``` text
Updated 2 albums
```

You can also execute a SQL query to fetch the values that you just wrote.

Here's the code to execute the query:

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql -c "SELECT singer_id, album_id, marketing_budget
         FROM albums
         ORDER BY singer_id, album_id"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

class QueryDataWithNewColumn {
  static void queryDataWithNewColumn(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      try (ResultSet resultSet =
          connection
              .createStatement()
              .executeQuery(
                  "SELECT singer_id, album_id, marketing_budget "
                      + "FROM albums "
                      + "ORDER BY singer_id, album_id")) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %d %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getLong("album_id"),
              resultSet.getString("marketing_budget"));
        }
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "database/sql"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func QueryDataWithNewColumn(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    rows, err := conn.Query(ctx, "SELECT singer_id, album_id, marketing_budget "+
        "FROM albums "+
        "ORDER BY singer_id, album_id")
    defer rows.Close()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId, albumId int64
        var marketingBudget sql.NullString
        err = rows.Scan(&singerId, &albumId, &marketingBudget)
        if err != nil {
            return err
        }
        var budget string
        if marketingBudget.Valid {
            budget = marketingBudget.String
        } else {
            budget = "NULL"
        }
        fmt.Printf("%v %v %v\n", singerId, albumId, budget)
    }

    return rows.Err()
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function queryDataWithNewColumn(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  const result = await connection.query(
      "SELECT singer_id, album_id, marketing_budget "
      + "FROM albums "
      + "ORDER BY singer_id, album_id"
  );
  for (const row of result.rows) {
    console.log(`${row["singer_id"]} ${row["album_id"]} ${row["marketing_budget"]}`);
  }

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def query_data_with_new_column(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            cur.execute("SELECT singer_id, album_id, marketing_budget "
                        "FROM albums "
                        "ORDER BY singer_id, album_id")
            for album in cur:
                print(album)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class QueryDataWithNewColumnSample
{
    public static void QueryWithNewColumnData(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = new NpgsqlCommand("SELECT singer_id, album_id, marketing_budget "
                                          + "FROM albums "
                                          + "ORDER BY singer_id, album_id", connection);
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["album_id"]} {reader["marketing_budget"]}");
        }
    }
}
```

### PHP

``` php
function query_data_with_new_column(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    $statement = $connection->query(
        "SELECT singer_id, album_id, marketing_budget "
        ."FROM albums "
        ."ORDER BY singer_id, album_id"
    );
    $rows = $statement->fetchAll();
    foreach ($rows as $album)
    {
        printf("%s\t%s\t%s\n", $album["singer_id"], $album["album_id"], $album["marketing_budget"]);
    }

    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the query with this command:

### psql

``` text
PGDATABASE=example-db ./query_data_with_new_column.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar querymarketingbudget example-db
```

### Go

``` text
go run sample_runner.go querymarketingbudget example-db
```

### Node.js

``` text
npm start querymarketingbudget example-db
```

### Python

``` text
python query_data_with_new_column.py example-db
```

### C\#

``` text
dotnet run querymarketingbudget example-db
```

### PHP

``` text
php query_data_with_new_column.php example-db
```

You should see:

``` text
1 1 100000
1 2 null
2 1 null
2 2 500000
2 3 null
```

## Update data

You can update data using DML in a read-write transaction.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql << SQL
  -- Transfer marketing budget from one album to another.
  -- We do it in a transaction to ensure that the transfer is atomic.
  -- Begin a read/write transaction.
  begin;

  -- Increase the marketing budget of album 1 if album 2 has enough budget.
  -- The condition that album 2 has enough budget is guaranteed for the
  -- duration of the transaction, as read/write transactions in Spanner use
  -- external consistency as the default isolation level.
  update albums set
    marketing_budget = marketing_budget + 200000
  where singer_id = 1
    and  album_id = 1
    and exists (
      select album_id
      from albums
      where singer_id = 2
        and  album_id = 2
        and marketing_budget > 200000
      );

  -- Decrease the marketing budget of album 2.      
  update albums set
    marketing_budget = marketing_budget - 200000
  where singer_id = 2
    and  album_id = 2
    and marketing_budget > 200000;

  -- Commit the transaction to make the changes to both marketing budgets
  -- durably stored in the database and visible to other transactions.
  commit;  
SQL

echo "Transferred marketing budget from Album 2 to Album 1"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

class UpdateDataWithTransaction {

  static void writeWithTransactionUsingDml(String host, int port, String database)
      throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Set AutoCommit=false to enable transactions.
      connection.setAutoCommit(false);

      // Transfer marketing budget from one album to another. We do it in a
      // transaction to ensure that the transfer is atomic. There is no need
      // to explicitly start the transaction. The first statement on the
      // connection will start a transaction when AutoCommit=false.
      String selectMarketingBudgetSql =
          "SELECT marketing_budget from albums WHERE singer_id = ? and album_id = ?";
      long album2Budget = 0;
      try (PreparedStatement selectMarketingBudgetStatement =
          connection.prepareStatement(selectMarketingBudgetSql)) {
        // Bind the query parameters to SingerId=2 and AlbumId=2.
        selectMarketingBudgetStatement.setLong(1, 2);
        selectMarketingBudgetStatement.setLong(2, 2);
        try (ResultSet resultSet = selectMarketingBudgetStatement.executeQuery()) {
          while (resultSet.next()) {
            album2Budget = resultSet.getLong("marketing_budget");
          }
        }
        // The transaction will only be committed if this condition still holds
        // at the time of commit. Otherwise, the transaction will be aborted.
        final long transfer = 200000;
        if (album2Budget >= transfer) {
          long album1Budget = 0;
          // Re-use the existing PreparedStatement for selecting the
          // marketing_budget to get the budget for Album 1.
          // Bind the query parameters to SingerId=1 and AlbumId=1.
          selectMarketingBudgetStatement.setLong(1, 1);
          selectMarketingBudgetStatement.setLong(2, 1);
          try (ResultSet resultSet = selectMarketingBudgetStatement.executeQuery()) {
            while (resultSet.next()) {
              album1Budget = resultSet.getLong("marketing_budget");
            }
          }

          // Transfer part of the marketing budget of Album 2 to Album 1.
          album1Budget += transfer;
          album2Budget -= transfer;
          String updateSql =
              "UPDATE albums "
                  + "SET marketing_budget = ? "
                  + "WHERE singer_id = ? and album_id = ?";
          try (PreparedStatement updateStatement = connection.prepareStatement(updateSql)) {
            // Update Album 1.
            int paramIndex = 0;
            updateStatement.setLong(++paramIndex, album1Budget);
            updateStatement.setLong(++paramIndex, 1);
            updateStatement.setLong(++paramIndex, 1);
            // Create a DML batch by calling addBatch
            // on the current PreparedStatement.
            updateStatement.addBatch();

            // Update Album 2 in the same DML batch.
            paramIndex = 0;
            updateStatement.setLong(++paramIndex, album2Budget);
            updateStatement.setLong(++paramIndex, 2);
            updateStatement.setLong(++paramIndex, 2);
            updateStatement.addBatch();

            // Execute both DML statements in one batch.
            updateStatement.executeBatch();
          }
        }
      }
      // Commit the current transaction.
      connection.commit();
      System.out.println("Transferred marketing budget from Album 2 to Album 1");
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func WriteWithTransactionUsingDml(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Transfer marketing budget from one album to another. We do it in a
    // transaction to ensure that the transfer is atomic.
    tx, err := conn.Begin(ctx)
    if err != nil {
        return err
    }
    const selectSql = "SELECT marketing_budget " +
        "from albums " +
        "WHERE singer_id = $1 and album_id = $2"
    // Get the marketing_budget of singer 2 / album 2.
    row := tx.QueryRow(ctx, selectSql, 2, 2)
    var budget2 int64
    if err := row.Scan(&budget2); err != nil {
        tx.Rollback(ctx)
        return err
    }
    const transfer = 20000
    // The transaction will only be committed if this condition still holds
    // at the time of commit. Otherwise, the transaction will be aborted.
    if budget2 >= transfer {
        // Get the marketing_budget of singer 1 / album 1.
        row := tx.QueryRow(ctx, selectSql, 1, 1)
        var budget1 int64
        if err := row.Scan(&budget1); err != nil {
            tx.Rollback(ctx)
            return err
        }
        // Transfer part of the marketing budget of Album 2 to Album 1.
        budget1 += transfer
        budget2 -= transfer
        const updateSql = "UPDATE albums " +
            "SET marketing_budget = $1 " +
            "WHERE singer_id = $2 and album_id = $3"
        // Start a DML batch and execute it as part of the current transaction.
        batch := &pgx.Batch{}
        batch.Queue(updateSql, budget1, 1, 1)
        batch.Queue(updateSql, budget2, 2, 2)
        br := tx.SendBatch(ctx, batch)
        _, err = br.Exec()
        if err := br.Close(); err != nil {
            tx.Rollback(ctx)
            return err
        }
    }
    // Commit the current transaction.
    tx.Commit(ctx)
    fmt.Println("Transferred marketing budget from Album 2 to Album 1")

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function writeWithTransactionUsingDml(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Transfer marketing budget from one album to another. We do it in a
  // transaction to ensure that the transfer is atomic. node-postgres
  // requires you to explicitly start the transaction by executing 'begin'.
  await connection.query("begin");
  const selectMarketingBudgetSql = "SELECT marketing_budget " +
      "from albums " +
      "WHERE singer_id = $1 and album_id = $2";
  // Get the marketing_budget of singer 2 / album 2.
  const album2BudgetResult = await connection.query(selectMarketingBudgetSql, [2, 2]);
  let album2Budget = album2BudgetResult.rows[0]["marketing_budget"];
  const transfer = 200000;
  // The transaction will only be committed if this condition still holds
  // at the time of commit. Otherwise, the transaction will be aborted.
  if (album2Budget >= transfer) {
    // Get the marketing budget of singer 1 / album 1.
    const album1BudgetResult = await connection.query(selectMarketingBudgetSql, [1, 1]);
    let album1Budget = album1BudgetResult.rows[0]["marketing_budget"];
    // Transfer part of the marketing budget of Album 2 to Album 1.
    album1Budget += transfer;
    album2Budget -= transfer;
    const updateSql = "UPDATE albums " +
        "SET marketing_budget = $1 " +
        "WHERE singer_id = $2 and album_id = $3";
    // Start a DML batch. This batch will become part of the current transaction.
    // TODO: Enable when https://github.com/googleapis/java-spanner/pull/3114 has been merged
    // await connection.query("start batch dml");
    // Update the marketing budget of both albums.
    await connection.query(updateSql, [album1Budget, 1, 1]);
    await connection.query(updateSql, [album2Budget, 2, 2]);
    // TODO: Enable when https://github.com/googleapis/java-spanner/pull/3114 has been merged
    // await connection.query("run batch");
  }
  // Commit the current transaction.
  await connection.query("commit");
  console.log("Transferred marketing budget from Album 2 to Album 1");

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def update_data_with_transaction(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        # Set autocommit=False to use transactions.
        # The first statement that is executed starts the transaction.
        conn.autocommit = False
        with conn.cursor() as cur:
            # Transfer marketing budget from one album to another.
            # We do it in a transaction to ensure that the transfer is atomic.
            # There is no need to explicitly start the transaction. The first
            # statement on the connection will start a transaction when
            # AutoCommit=false.
            select_marketing_budget_sql = ("SELECT marketing_budget "
                                           "from albums "
                                           "WHERE singer_id = %s "
                                           "and album_id = %s")
            # Get the marketing budget of Album #2.
            cur.execute(select_marketing_budget_sql, (2, 2))
            album2_budget = cur.fetchone()[0]
            transfer = 200000
            if album2_budget > transfer:
                # Get the marketing budget of Album #1.
                cur.execute(select_marketing_budget_sql, (1, 1))
                album1_budget = cur.fetchone()[0]
                # Transfer the marketing budgets and write the update back
                # to the database.
                album1_budget += transfer
                album2_budget -= transfer
                update_sql = ("update albums "
                              "set marketing_budget = %s "
                              "where singer_id = %s "
                              "and   album_id = %s")
                # Use a pipeline to execute two DML statements in one batch.
                with conn.pipeline():
                    cur.execute(update_sql, (album1_budget, 1, 1,))
                    cur.execute(update_sql, (album2_budget, 2, 2,))
            else:
                print("Insufficient budget to transfer")
        # Commit the transaction.
        conn.commit()
        print("Transferred marketing budget from Album 2 to Album 1")
```

### C\#

``` csharp
using Npgsql;
using System.Data;

namespace dotnet_snippets;

public static class TagsSample
{
    public static void Tags(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Start a transaction with isolation level Serializable.
        // Spanner only supports this isolation level. Trying to use a lower
        // isolation level (including the default isolation level READ COMMITTED),
        // will result in an error.
        var transaction = connection.BeginTransaction(IsolationLevel.Serializable);

        // Create a command that uses the current transaction.
        using var cmd = connection.CreateCommand();
        cmd.Transaction = transaction;

        // Set the TRANSACTION_TAG session variable to set a transaction tag
        // for the current transaction.
        cmd.CommandText = "set spanner.transaction_tag='example-tx-tag'";
        cmd.ExecuteNonQuery();

        // Set the STATEMENT_TAG session variable to set the request tag
        // that should be included with the next SQL statement.
        cmd.CommandText = "set spanner.statement_tag='query-marketing-budget'";
        cmd.ExecuteNonQuery();

        // Get the marketing_budget of Album (1,1).
        cmd.CommandText = "select marketing_budget from albums where singer_id=$1 and album_id=$2";
        cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
        cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
        var marketingBudget = (long?)cmd.ExecuteScalar();

        // Reduce the marketing budget by 10% if it is more than 1,000.
        if (marketingBudget > 1000L)
        {
            marketingBudget -= (long) (marketingBudget * 0.1);

            // Set the statement tag to use for the update statement.
            cmd.Parameters.Clear();
            cmd.CommandText = "set spanner.statement_tag='reduce-marketing-budget'";
            cmd.ExecuteNonQuery();

            cmd.CommandText = "update albums set marketing_budget=$1 where singer_id=$2 AND album_id=$3";
            cmd.Parameters.Add(new NpgsqlParameter { Value = marketingBudget });
            cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
            cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
            cmd.ExecuteNonQuery();
        }

        // Commit the current transaction.
        transaction.Commit();
        Console.WriteLine("Reduced marketing budget");
    }
}
```

### PHP

``` php
function update_data_with_transaction(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Start a read/write transaction.
    $connection->beginTransaction();
    // Transfer marketing budget from one album to another.
    // We do it in a transaction to ensure that the transfer is atomic.

    // Create a prepared statement that we can use to execute the same
    // SQL string multiple times with different parameter values.
    $select_marketing_budget_statement = $connection->prepare(
        "SELECT marketing_budget "
        ."from albums "
        ."WHERE singer_id = ? "
        ."and album_id = ?"
    );
    // Get the marketing budget of Album #2.
    $select_marketing_budget_statement->execute([2, 2]);
    $album2_budget = $select_marketing_budget_statement->fetchAll()[0][0];
    $select_marketing_budget_statement->closeCursor();

    $transfer = 200000;
    if ($album2_budget > $transfer) {
        // Get the marketing budget of Album #1.
        $select_marketing_budget_statement->execute([1, 1]);
        $album1_budget = $select_marketing_budget_statement->fetchAll()[0][0];
        $select_marketing_budget_statement->closeCursor();
        // Transfer the marketing budgets and write the update back
        // to the database.
        $album1_budget += $transfer;
        $album2_budget -= $transfer;
        // PHP PDO also supports named query parameters.
        $update_statement = $connection->prepare(
            "update albums "
                ."set marketing_budget = :budget "
                ."where singer_id = :singer_id "
                ."and   album_id = :album_id"
        );
        // Start a DML batch. This batch will become part of the current transaction.
        // $connection->exec("start batch dml");
        // Update the marketing budget of both albums.
        $update_statement->execute(["budget" => $album1_budget, "singer_id" => 1, "album_id" => 1]);
        $update_statement->execute(["budget" => $album2_budget, "singer_id" => 2, "album_id" => 2]);
        // $connection->exec("run batch");
    } else {
        print("Insufficient budget to transfer\n");
    }
    // Commit the transaction.
    $connection->commit();
    print("Transferred marketing budget from Album 2 to Album 1\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./update_data_with_transaction.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar writewithtransactionusingdml example-db
```

### Go

``` text
go run sample_runner.go writewithtransactionusingdml example-db
```

### Node.js

``` text
npm start writewithtransactionusingdml example-db
```

### Python

``` text
python update_data_with_transaction.py example-db
```

### C\#

``` text
dotnet run writewithtransactionusingdml example-db
```

### PHP

``` text
php update_data_with_transaction.php example-db
```

You should see:

``` text
Transferred marketing budget from Album 2 to Album 1
```

### Transaction tags and request tags

Use [transaction tags and request tags](/spanner/docs/introspection/troubleshooting-with-tags) to troubleshoot transactions and queries in Spanner. You can set transaction tags and request tags with the `  SPANNER.TRANSACTION_TAG  ` and `  SPANNER.STATEMENT_TAG  ` session variables.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql << SQL
  -- Start a transaction.
  begin;
  -- Set the TRANSACTION_TAG session variable to set a transaction tag
  -- for the current transaction. This can only be executed at the start
  -- of the transaction.
  set spanner.transaction_TAG='example-tx-tag';

  -- Set the STATEMENT_TAG session variable to set the request tag
  -- that should be included with the next SQL statement.
  set spanner.statement_tag='query-marketing-budget';

  select marketing_budget
  from albums
  where singer_id = 1
    and album_id  = 1;

  -- Reduce the marketing budget by 10% if it is more than 1,000.
  -- Set a statement tag for the update statement.
  set spanner.statement_tag='reduce-marketing-budget';

  update albums
    set marketing_budget = marketing_budget - (marketing_budget * 0.1)::bigint
  where singer_id = 1
    and album_id  = 1
    and marketing_budget > 1000;

  commit;  
SQL

echo "Reduced marketing budget"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

class Tags {

  static void tags(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Set AutoCommit=false to enable transactions.
      connection.setAutoCommit(false);
      // Set the TRANSACTION_TAG session variable to set a transaction tag
      // for the current transaction.
      connection.createStatement().execute("set spanner.transaction_tag='example-tx-tag'");

      // Set the STATEMENT_TAG session variable to set the request tag
      // that should be included with the next SQL statement.
      connection.createStatement().execute("set spanner.statement_tag='query-marketing-budget'");
      long marketingBudget = 0L;
      long singerId = 1L;
      long albumId = 1L;
      try (PreparedStatement statement =
          connection.prepareStatement(
              "select marketing_budget from albums where singer_id=? and album_id=?")) {
        statement.setLong(1, singerId);
        statement.setLong(2, albumId);
        try (ResultSet albumResultSet = statement.executeQuery()) {
          while (albumResultSet.next()) {
            marketingBudget = albumResultSet.getLong(1);
          }
        }
      }
      // Reduce the marketing budget by 10% if it is more than 1,000.
      final long maxMarketingBudget = 1000L;
      final float reduction = 0.1f;
      if (marketingBudget > maxMarketingBudget) {
        marketingBudget -= (long) (marketingBudget * reduction);
        connection.createStatement().execute("set spanner.statement_tag='reduce-marketing-budget'");
        try (PreparedStatement statement =
            connection.prepareStatement(
                "update albums set marketing_budget=? where singer_id=? AND album_id=?")) {
          int paramIndex = 0;
          statement.setLong(++paramIndex, marketingBudget);
          statement.setLong(++paramIndex, singerId);
          statement.setLong(++paramIndex, albumId);
          statement.executeUpdate();
        }
      }

      // Commit the current transaction.
      connection.commit();
      System.out.println("Reduced marketing budget");
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func Tags(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    tx, err := conn.Begin(ctx)
    if err != nil {
        return err
    }

    // Set the TRANSACTION_TAG session variable to set a transaction tag
    // for the current transaction.
    _, _ = tx.Exec(ctx, "set spanner.transaction_tag='example-tx-tag'")

    // Set the STATEMENT_TAG session variable to set the request tag
    // that should be included with the next SQL statement.
    _, _ = tx.Exec(ctx, "set spanner.statement_tag='query-marketing-budget'")

    row := tx.QueryRow(ctx, "select marketing_budget "+
        "from albums "+
        "where singer_id=$1 and album_id=$2", 1, 1)
    var budget int64
    if err := row.Scan(&budget); err != nil {
        tx.Rollback(ctx)
        return err
    }

    // Reduce the marketing budget by 10% if it is more than 1,000.
    if budget > 1000 {
        budget = int64(float64(budget) - float64(budget)*0.1)
        _, _ = tx.Exec(ctx, "set spanner.statement_tag='reduce-marketing-budget'")
        if _, err := tx.Exec(ctx, "update albums set marketing_budget=$1 where singer_id=$2 AND album_id=$3", budget, 1, 1); err != nil {
            tx.Rollback(ctx)
            return err
        }
    }
    // Commit the current transaction.
    tx.Commit(ctx)
    fmt.Println("Reduced marketing budget")

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function tags(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  await connection.query("begin");
  // Set the TRANSACTION_TAG session variable to set a transaction tag
  // for the current transaction.
  await connection.query("set spanner.transaction_tag='example-tx-tag'");
  // Set the STATEMENT_TAG session variable to set the request tag
  // that should be included with the next SQL statement.
  await connection.query("set spanner.statement_tag='query-marketing-budget'");
  const budgetResult = await connection.query(
      "select marketing_budget " +
      "from albums " +
      "where singer_id=$1 and album_id=$2", [1, 1])
  let budget = budgetResult.rows[0]["marketing_budget"];
  // Reduce the marketing budget by 10% if it is more than 1,000.
  if (budget > 1000) {
    budget = budget - budget * 0.1;
    await connection.query("set spanner.statement_tag='reduce-marketing-budget'");
    await connection.query("update albums set marketing_budget=$1 "
        + "where singer_id=$2 AND album_id=$3", [budget, 1, 1]);
  }
  // Commit the current transaction.
  await connection.query("commit");
  console.log("Reduced marketing budget");

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def tags(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        # Set autocommit=False to enable transactions.
        conn.autocommit = False
        with conn.cursor() as cur:
            # Set the TRANSACTION_TAG session variable to set a transaction tag
            # for the current transaction.
            cur.execute("set spanner.transaction_TAG='example-tx-tag'")

            # Set the STATEMENT_TAG session variable to set the request tag
            # that should be included with the next SQL statement.
            cur.execute("set spanner.statement_tag='query-marketing-budget'")

            singer_id = 1
            album_id = 1
            cur.execute("select marketing_budget "
                        "from albums "
                        "where singer_id = %s "
                        "  and album_id  = %s",
                        (singer_id, album_id,))
            marketing_budget = cur.fetchone()[0]

            # Reduce the marketing budget by 10% if it is more than 1,000.
            max_marketing_budget = 1000
            reduction = 0.1
            if marketing_budget > max_marketing_budget:
                # Make sure the marketing_budget remains an int.
                marketing_budget -= int(marketing_budget * reduction)
                # Set a statement tag for the update statement.
                cur.execute(
                    "set spanner.statement_tag='reduce-marketing-budget'")
                cur.execute("update albums set marketing_budget = %s "
                            "where singer_id = %s "
                            "  and album_id  = %s",
                            (marketing_budget, singer_id, album_id,))
            else:
                print("Marketing budget already less than or equal to 1,000")
        # Commit the transaction.
        conn.commit()
        print("Reduced marketing budget")
```

### C\#

``` csharp
using Npgsql;
using System.Data;

namespace dotnet_snippets;

public static class TagsSample
{
    public static void Tags(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Start a transaction with isolation level Serializable.
        // Spanner only supports this isolation level. Trying to use a lower
        // isolation level (including the default isolation level READ COMMITTED),
        // will result in an error.
        var transaction = connection.BeginTransaction(IsolationLevel.Serializable);

        // Create a command that uses the current transaction.
        using var cmd = connection.CreateCommand();
        cmd.Transaction = transaction;

        // Set the TRANSACTION_TAG session variable to set a transaction tag
        // for the current transaction.
        cmd.CommandText = "set spanner.transaction_tag='example-tx-tag'";
        cmd.ExecuteNonQuery();

        // Set the STATEMENT_TAG session variable to set the request tag
        // that should be included with the next SQL statement.
        cmd.CommandText = "set spanner.statement_tag='query-marketing-budget'";
        cmd.ExecuteNonQuery();

        // Get the marketing_budget of Album (1,1).
        cmd.CommandText = "select marketing_budget from albums where singer_id=$1 and album_id=$2";
        cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
        cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
        var marketingBudget = (long?)cmd.ExecuteScalar();

        // Reduce the marketing budget by 10% if it is more than 1,000.
        if (marketingBudget > 1000L)
        {
            marketingBudget -= (long) (marketingBudget * 0.1);

            // Set the statement tag to use for the update statement.
            cmd.Parameters.Clear();
            cmd.CommandText = "set spanner.statement_tag='reduce-marketing-budget'";
            cmd.ExecuteNonQuery();

            cmd.CommandText = "update albums set marketing_budget=$1 where singer_id=$2 AND album_id=$3";
            cmd.Parameters.Add(new NpgsqlParameter { Value = marketingBudget });
            cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
            cmd.Parameters.Add(new NpgsqlParameter { Value = 1L });
            cmd.ExecuteNonQuery();
        }

        // Commit the current transaction.
        transaction.Commit();
        Console.WriteLine("Reduced marketing budget");
    }
}
```

### PHP

``` php
function tags(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Start a read/write transaction.
    $connection->beginTransaction();

    // Set the TRANSACTION_TAG session variable to set a transaction tag
    // for the current transaction.
    $connection->exec("set spanner.transaction_TAG='example-tx-tag'");

    // Set the STATEMENT_TAG session variable to set the request tag
    // that should be included with the next SQL statement.
    $connection->exec("set spanner.statement_tag='query-marketing-budget'");

    $singer_id = 1;
    $album_id = 1;
    $statement = $connection->prepare(
        "select marketing_budget "
        ."from albums "
        ."where singer_id = ? "
        ."  and album_id  = ?"
    );
    $statement->execute([1, 1]);
    $marketing_budget = $statement->fetchAll()[0][0];
    $statement->closeCursor();

    # Reduce the marketing budget by 10% if it is more than 1,000.
    $max_marketing_budget = 1000;
    $reduction = 0.1;
    if ($marketing_budget > $max_marketing_budget) {
        // Make sure the marketing_budget remains an int.
        $marketing_budget -= intval($marketing_budget * $reduction);
        // Set a statement tag for the update statement.
        $connection->exec("set spanner.statement_tag='reduce-marketing-budget'");
        $update_statement = $connection->prepare(
            "update albums set marketing_budget = :budget "
            ."where singer_id = :singer_id "
            ."  and album_id  = :album_id"
        );
        $update_statement->execute([
            "budget" => $marketing_budget,
            "singer_id" => $singer_id,
            "album_id" => $album_id,
        ]);
    } else {
        print("Marketing budget already less than or equal to 1,000\n");
    }
    // Commit the transaction.
    $connection->commit();
    print("Reduced marketing budget\n");

    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./tags.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar tags example-db
```

### Go

``` text
go run sample_runner.go tags example-db
```

### Node.js

``` text
npm start tags example-db
```

### Python

``` text
python tags.py example-db
```

### C\#

``` text
dotnet run tags example-db
```

### PHP

``` text
php tags.php example-db
```

**Tip:** For a full list of commands that can be used to access Spanner features with PGAdapter, see [PGAdapter session management commands](/spanner/docs/pgadapter-session-mgmt-commands) .

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Set the connection to read-only or use the `  SET TRANSACTION READ ONLY  ` SQL statement to execute a read-only transaction.

**Tip:** PGAdapter supports multiple additional SQL statements for executing specific types of transactions and batches, and for accessing specific Spanner features. For a full list of supported statements, see [PGAdapter session management commands](/spanner/docs/pgadapter-session-mgmt-commands)

The following shows how to run a query and perform a read in the same read-only transaction:

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

psql << SQL
  -- Begin a transaction.
  begin;
  -- Change the current transaction to a read-only transaction.
  -- This statement can only be executed at the start of a transaction.
  set transaction read only;

  -- The following two queries use the same read-only transaction.
  select singer_id, album_id, album_title
  from albums
  order by singer_id, album_id;

  select singer_id, album_id, album_title
  from albums
  order by album_title;

  -- Read-only transactions must also be committed or rolled back to mark
  -- the end of the transaction. There is no semantic difference between
  -- rolling back or committing a read-only transaction.
  commit;  
SQL
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

class ReadOnlyTransaction {
  static void readOnlyTransaction(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Set AutoCommit=false to enable transactions.
      connection.setAutoCommit(false);
      // This SQL statement instructs the JDBC driver to use
      // a read-only transaction.
      connection.createStatement().execute("set transaction read only");

      try (ResultSet resultSet =
          connection
              .createStatement()
              .executeQuery(
                  "SELECT singer_id, album_id, album_title "
                      + "FROM albums "
                      + "ORDER BY singer_id, album_id")) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %d %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getLong("album_id"),
              resultSet.getString("album_title"));
        }
      }
      try (ResultSet resultSet =
          connection
              .createStatement()
              .executeQuery(
                  "SELECT singer_id, album_id, album_title "
                      + "FROM albums "
                      + "ORDER BY album_title")) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %d %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getLong("album_id"),
              resultSet.getString("album_title"));
        }
      }
      // End the read-only transaction by calling commit().
      connection.commit();
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func ReadOnlyTransaction(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Start a read-only transaction by supplying additional transaction options.
    tx, err := conn.BeginTx(ctx, pgx.TxOptions{AccessMode: pgx.ReadOnly})

    albumsOrderedById, err := tx.Query(ctx, "SELECT singer_id, album_id, album_title FROM albums ORDER BY singer_id, album_id")
    defer albumsOrderedById.Close()
    if err != nil {
        return err
    }
    for albumsOrderedById.Next() {
        var singerId, albumId int64
        var title string
        err = albumsOrderedById.Scan(&singerId, &albumId, &title)
        if err != nil {
            return err
        }
        fmt.Printf("%v %v %v\n", singerId, albumId, title)
    }

    albumsOrderedTitle, err := tx.Query(ctx, "SELECT singer_id, album_id, album_title FROM albums ORDER BY album_title")
    defer albumsOrderedTitle.Close()
    if err != nil {
        return err
    }
    for albumsOrderedTitle.Next() {
        var singerId, albumId int64
        var title string
        err = albumsOrderedTitle.Scan(&singerId, &albumId, &title)
        if err != nil {
            return err
        }
        fmt.Printf("%v %v %v\n", singerId, albumId, title)
    }

    // End the read-only transaction by calling Commit().
    return tx.Commit(ctx)
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function readOnlyTransaction(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Start a transaction.
  await connection.query("begin");
  // This SQL statement instructs the PGAdapter to make it a read-only transaction.
  await connection.query("set transaction read only");

  const albumsOrderById = await connection.query(
      "SELECT singer_id, album_id, album_title "
      + "FROM albums "
      + "ORDER BY singer_id, album_id");
  for (const row of albumsOrderById.rows) {
    console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
  }
  const albumsOrderByTitle = await connection.query(
      "SELECT singer_id, album_id, album_title "
      + "FROM albums "
      + "ORDER BY album_title");
  for (const row of albumsOrderByTitle.rows) {
    console.log(`${row["singer_id"]} ${row["album_id"]} ${row["album_title"]}`);
  }
  // End the read-only transaction by executing commit.
  await connection.query("commit");

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def read_only_transaction(host: string, port: int, database: string):
    with (psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn):
        # Set autocommit=False to enable transactions.
        conn.autocommit = False

        with conn.cursor() as cur:
            # Change the current transaction to a read-only transaction.
            # This statement can only be executed at the start of a transaction.
            cur.execute("set transaction read only")

            # The following two queries use the same read-only transaction.
            cur.execute("select singer_id, album_id, album_title "
                        "from albums "
                        "order by singer_id, album_id")
            for album in cur:
                print(album)

            cur.execute("select singer_id, album_id, album_title "
                        "from albums "
                        "order by album_title")
            for album in cur:
                print(album)

        # Read-only transactions must also be committed or rolled back to mark
        # the end of the transaction. There is no semantic difference between
        # rolling back or committing a read-only transaction.
        conn.commit()
```

### C\#

``` csharp
using Npgsql;
using System.Data;

namespace dotnet_snippets;

public static class ReadOnlyTransactionSample
{
    public static void ReadOnlyTransaction(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Start a read-only transaction.
        // You must specify Serializable as the isolation level, as the npgsql driver
        // will otherwise automatically set the isolation level to read-committed.
        var transaction = connection.BeginTransaction(IsolationLevel.Serializable);
        using var cmd = connection.CreateCommand();
        cmd.Transaction = transaction;
        // This SQL statement instructs the npgsql driver to use
        // a read-only transaction.
        cmd.CommandText = "set transaction read only";
        cmd.ExecuteNonQuery();

        cmd.CommandText = "SELECT singer_id, album_id, album_title " +
                          "FROM albums " +
                          "ORDER BY singer_id, album_id";
        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine($"{reader["singer_id"]} {reader["album_id"]} {reader["album_title"]}");
            }
        }
        cmd.CommandText = "SELECT singer_id, album_id, album_title "
                          + "FROM albums "
                          + "ORDER BY album_title";
        using (var reader = cmd.ExecuteReader())
        {
            while (reader.Read())
            {
                Console.WriteLine($"{reader["singer_id"]} {reader["album_id"]} {reader["album_title"]}");
            }
        }
        // End the read-only transaction by calling commit().
        transaction.Commit();
    }
}
```

### PHP

``` php
function read_only_transaction(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Start a transaction.
    $connection->beginTransaction();
    // Change the current transaction to a read-only transaction.
    // This statement can only be executed at the start of a transaction.
    $connection->exec("set transaction read only");

    // The following two queries use the same read-only transaction.
    $statement = $connection->query(
        "select singer_id, album_id, album_title "
        ."from albums "
        ."order by singer_id, album_id"
    );
    $rows = $statement->fetchAll();
    foreach ($rows as $album)
    {
        printf("%s\t%s\t%s\n", $album["singer_id"], $album["album_id"], $album["album_title"]);
    }

    $statement = $connection->query(
        "select singer_id, album_id, album_title "
        ."from albums "
        ."order by album_title"
    );
    $rows = $statement->fetchAll();
    foreach ($rows as $album)
    {
        printf("%s\t%s\t%s\n", $album["singer_id"], $album["album_id"], $album["album_title"]);
    }

    # Read-only transactions must also be committed or rolled back to mark
    # the end of the transaction. There is no semantic difference between
    # rolling back or committing a read-only transaction.
    $connection->commit();

    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./read_only_transaction.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar readonlytransaction example-db
```

### Go

``` text
go run sample_runner.go readonlytransaction example-db
```

### Node.js

``` text
npm start readonlytransaction example-db
```

### Python

``` text
python read_only_transaction.py example-db
```

### C\#

``` text
dotnet run readonlytransaction example-db
```

### PHP

``` text
php read_only_transaction.php example-db
```

You should see output similar to:

``` text
    1 1 Total Junk
    1 2 Go, Go, Go
    2 1 Green
    2 2 Forever Hold Your Peace
    2 3 Terrified
    2 2 Forever Hold Your Peace
    1 2 Go, Go, Go
    2 1 Green
    2 3 Terrified
    1 1 Total Junk
```

### Partitioned queries and Data Boost

The [`  partitionQuery  `](/spanner/docs/reference/rest/v1/projects.instances.databases.sessions/partitionQuery) API divides a query into smaller pieces, or partitions, and uses multiple machines to fetch the partitions in parallel. Each partition is identified by a partition token. The PartitionQuery API has higher latency than the standard query API, because it is only intended for bulk operations such as exporting or scanning the whole database.

[Data Boost](/spanner/docs/databoost/databoost-overview) lets you execute analytics queries and data exports with near-zero impact to existing workloads on the provisioned Spanner instance. Data Boost only supports [partitioned queries](/spanner/docs/reads#read_data_in_parallel) .

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# 'set spanner.data_boost_enabled=true' enables Data Boost for
# all partitioned queries on this connection.

# 'run partitioned query' is a shortcut for partitioning the query
# that follows and executing each of the partitions that is returned
# by Spanner.

psql -c "set spanner.data_boost_enabled=true" \
     -c "run partitioned query
         select singer_id, first_name, last_name
         from singers"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;

class DataBoost {
  static void dataBoost(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // This enables Data Boost for all partitioned queries on this connection.
      connection.createStatement().execute("set spanner.data_boost_enabled=true");

      // Run a partitioned query. This query will use Data Boost.
      try (ResultSet resultSet =
          connection
              .createStatement()
              .executeQuery(
                  "run partitioned query "
                      + "select singer_id, first_name, last_name "
                      + "from singers")) {
        while (resultSet.next()) {
          System.out.printf(
              "%d %s %s\n",
              resultSet.getLong("singer_id"),
              resultSet.getString("first_name"),
              resultSet.getString("last_name"));
        }
      }
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func DataBoost(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // This enables Data Boost for all partitioned queries on this connection.
    _, _ = conn.Exec(ctx, "set spanner.data_boost_enabled=true")

    // Run a partitioned query. This query will use Data Boost.
    rows, err := conn.Query(ctx, "run partitioned query select singer_id, first_name, last_name from singers")
    defer rows.Close()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId int64
        var firstName, lastName string
        err = rows.Scan(&singerId, &firstName, &lastName)
        if err != nil {
            return err
        }
        fmt.Printf("%v %v %v\n", singerId, firstName, lastName)
    }

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function dataBoost(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // This enables Data Boost for all partitioned queries on this connection.
  await connection.query("set spanner.data_boost_enabled=true");

  // Run a partitioned query. This query will use Data Boost.
  const singers = await connection.query(
      "run partitioned query "
      + "select singer_id, first_name, last_name "
      + "from singers");
  for (const row of singers.rows) {
    console.log(`${row["singer_id"]} ${row["first_name"]} ${row["last_name"]}`);
  }

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def data_boost(host: string, port: int, database: string):
    with (psycopg.connect("host={host} port={port} dbname={database} "
                          "sslmode=disable".format(host=host,
                                                   port=port,
                                                   database=database)) as conn):
        # Set autocommit=True so each query uses a separate transaction.
        conn.autocommit = True

        with conn.cursor() as cur:
            # This enables Data Boost for all partitioned queries on this
            # connection.
            cur.execute("set spanner.data_boost_enabled=true")

            # Run a partitioned query. This query will use Data Boost.
            cur.execute("run partitioned query "
                        "select singer_id, first_name, last_name "
                        "from singers")
            for singer in cur:
                print(singer)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class DataBoostSample
{
    public static void DataBoost(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        using var cmd = connection.CreateCommand();
        // This enables Data Boost for all partitioned queries on this connection.
        cmd.CommandText = "set spanner.data_boost_enabled=true";
        cmd.ExecuteNonQuery();


        // Run a partitioned query. This query will use Data Boost.
        cmd.CommandText = "run partitioned query "
                          + "select singer_id, first_name, last_name "
                          + "from singers";
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            Console.WriteLine($"{reader["singer_id"]} {reader["first_name"]} {reader["last_name"]}");
        }
    }
}
```

### PHP

``` php
function data_boost(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // This enables Data Boost for all partitioned queries on this
    // connection.
    $connection->exec("set spanner.data_boost_enabled=true");

    // Run a partitioned query. This query will use Data Boost.
    $statement = $connection->query(
        "run partitioned query "
        ."select singer_id, first_name, last_name "
        ."from singers"
    );
    $rows = $statement->fetchAll();
    foreach ($rows as $singer) {
        printf("%s\t%s\t%s\n", $singer["singer_id"], $singer["first_name"], $singer["last_name"]);
    }

    $rows = null;
    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./data_boost.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar databoost example-db
```

### Go

``` text
go run sample_runner.go databoost example-db
```

### Node.js

``` text
npm start databoost example-db
```

### Python

``` text
python data_boost.py example-db
```

### C\#

``` text
dotnet run databoost example-db
```

### PHP

``` text
php data_boost.php example-db
```

For more information on running partitioned queries and using Data Boost with PGAdapter, see: [Data Boost and partitioned query statements](/spanner/docs/pgadapter-session-mgmt-commands#data_boost_and_partitioned_query_statements)

## Partitioned DML

[Partitioned Data Manipulation Language (DML)](/spanner/docs/dml-partitioned) is designed for the following types of bulk updates and deletes:

  - Periodic cleanup and garbage collection.
  - Backfilling new columns with default values.

### psql

``` bash
#!/bin/bash

export PGHOST="${PGHOST:-localhost}"
export PGPORT="${PGPORT:-5432}"
export PGDATABASE="${PGDATABASE:-example-db}"

# Change the DML mode that is used by this connection to Partitioned
# DML. Partitioned DML is designed for bulk updates and deletes.
# See https://cloud.google.com/spanner/docs/dml-partitioned for more
# information.
psql -c "set spanner.autocommit_dml_mode='partitioned_non_atomic'" \
     -c "update albums
         set marketing_budget=0
         where marketing_budget is null"

echo "Updated albums using Partitioned DML"
```

### Java

``` java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

class PartitionedDml {

  static void partitionedDml(String host, int port, String database) throws SQLException {
    String connectionUrl = String.format("jdbc:postgresql://%s:%d/%s", host, port, database);
    try (Connection connection = DriverManager.getConnection(connectionUrl)) {
      // Enable Partitioned DML on this connection.
      connection
          .createStatement()
          .execute("set spanner.autocommit_dml_mode='partitioned_non_atomic'");
      // Back-fill a default value for the MarketingBudget column.
      long lowerBoundUpdateCount =
          connection
              .createStatement()
              .executeUpdate("update albums set marketing_budget=0 where marketing_budget is null");
      System.out.printf("Updated at least %d albums\n", lowerBoundUpdateCount);
    }
  }
}
```

### Go

``` go
import (
    "context"
    "fmt"

    "github.com/jackc/pgx/v5"
)

func PartitionedDML(host string, port int, database string) error {
    ctx := context.Background()
    connString := fmt.Sprintf(
        "postgres://uid:pwd@%s:%d/%s?sslmode=disable",
        host, port, database)
    conn, err := pgx.Connect(ctx, connString)
    if err != nil {
        return err
    }
    defer conn.Close(ctx)

    // Enable Partitioned DML on this connection.
    if _, err := conn.Exec(ctx, "set spanner.autocommit_dml_mode='partitioned_non_atomic'"); err != nil {
        return err
    }
    // Back-fill a default value for the MarketingBudget column.
    tag, err := conn.Exec(ctx, "update albums set marketing_budget=0 where marketing_budget is null")
    if err != nil {
        return err
    }
    fmt.Printf("Updated at least %v albums\n", tag.RowsAffected())

    return nil
}
```

### Node.js

``` typescript
import { Client } from 'pg';

async function partitionedDml(host: string, port: number, database: string): Promise<void> {
  const connection = new Client({
    host: host,
    port: port,
    database: database,
  });
  await connection.connect();

  // Enable Partitioned DML on this connection.
  await connection.query("set spanner.autocommit_dml_mode='partitioned_non_atomic'");

  // Back-fill a default value for the MarketingBudget column.
  const lowerBoundUpdateCount = await connection.query(
      "update albums " +
      "set marketing_budget=0 " +
      "where marketing_budget is null");
  console.log(`Updated at least ${lowerBoundUpdateCount.rowCount} albums`);

  // Close the connection.
  await connection.end();
}
```

### Python

``` python
import string
import psycopg


def execute_partitioned_dml(host: string, port: int, database: string):
    with psycopg.connect("host={host} port={port} dbname={database} "
                         "sslmode=disable".format(host=host,
                                                  port=port,
                                                  database=database)) as conn:
        conn.autocommit = True
        with conn.cursor() as cur:
            # Change the DML mode that is used by this connection to Partitioned
            # DML. Partitioned DML is designed for bulk updates and deletes.
            # See https://cloud.google.com/spanner/docs/dml-partitioned for more
            # information.
            cur.execute(
                "set spanner.autocommit_dml_mode='partitioned_non_atomic'")

            # The following statement will use Partitioned DML.
            cur.execute("update albums "
                        "set marketing_budget=0 "
                        "where marketing_budget is null")
            print("Updated at least %d albums" % cur.rowcount)
```

### C\#

``` csharp
using Npgsql;

namespace dotnet_snippets;

public static class PartitionedDmlSample
{
    public static void PartitionedDml(string host, int port, string database)
    {
        var connectionString = $"Host={host};Port={port};Database={database};SSL Mode=Disable";
        using var connection = new NpgsqlConnection(connectionString);
        connection.Open();

        // Enable Partitioned DML on this connection.
        using var cmd = connection.CreateCommand();
        cmd.CommandText = "set spanner.autocommit_dml_mode='partitioned_non_atomic'";
        cmd.ExecuteNonQuery();

        // Back-fill a default value for the MarketingBudget column.
        cmd.CommandText = "update albums set marketing_budget=0 where marketing_budget is null";
        var lowerBoundUpdateCount = cmd.ExecuteNonQuery();

        Console.WriteLine($"Updated at least {lowerBoundUpdateCount} albums");
    }
}
```

### PHP

``` php
function execute_partitioned_dml(string $host, string $port, string $database): void
{
    $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
    $connection = new PDO($dsn);

    // Change the DML mode that is used by this connection to Partitioned
    // DML. Partitioned DML is designed for bulk updates and deletes.
    // See https://cloud.google.com/spanner/docs/dml-partitioned for more
    // information.
    $connection->exec("set spanner.autocommit_dml_mode='partitioned_non_atomic'");

    // The following statement will use Partitioned DML.
    $rowcount = $connection->exec(
        "update albums "
        ."set marketing_budget=0 "
        ."where marketing_budget is null"
    );
    printf("Updated at least %d albums\n", $rowcount);

    $statement = null;
    $connection = null;
}
```

Run the sample with the following command:

### psql

``` text
PGDATABASE=example-db ./partitioned_dml.sh
```

### Java

``` text
java -jar target/pgadapter-snippets/pgadapter-samples.jar partitioneddml example-db
```

### Go

``` text
go run sample_runner.go partitioneddml example-db
```

### Node.js

``` text
npm start partitioneddml example-db
```

### Python

``` text
python partitioned_dml.py example-db
```

### C\#

``` text
dotnet run datpartitioneddmlboost example-db
```

### PHP

``` text
php partitioned_dml.php example-db
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
