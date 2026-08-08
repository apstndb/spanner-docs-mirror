---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-create-database
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-create-database
title: Create database
description: Create a database.
data_source: docs.cloud.google.com
---

Create a database.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Create and manage databases](https://docs.cloud.google.com/spanner/docs/create-manage-databases)
  - [Getting started with Spanner and PGAdapter](https://docs.cloud.google.com/spanner/docs/getting-started/pgadapter)
  - [Getting started with Spanner in ADO.NET](https://docs.cloud.google.com/spanner/docs/getting-started/ado_net)
  - [Getting started with Spanner in C\#](https://docs.cloud.google.com/spanner/docs/getting-started/csharp)
  - [Getting started with Spanner in C++](https://docs.cloud.google.com/spanner/docs/getting-started/cpp)
  - [Getting started with Spanner in Go](https://docs.cloud.google.com/spanner/docs/getting-started/go)
  - [Getting started with Spanner in Go database/sql](https://docs.cloud.google.com/spanner/docs/getting-started/database_sql)
  - [Getting started with Spanner in Java](https://docs.cloud.google.com/spanner/docs/getting-started/java)
  - [Getting started with Spanner in JDBC](https://docs.cloud.google.com/spanner/docs/getting-started/jdbc)
  - [Getting started with Spanner in Node.js](https://docs.cloud.google.com/spanner/docs/getting-started/nodejs)
  - [Getting started with Spanner in PHP](https://docs.cloud.google.com/spanner/docs/getting-started/php)
  - [Getting started with Spanner in Python](https://docs.cloud.google.com/spanner/docs/getting-started/python)
  - [Getting started with Spanner in Ruby](https://docs.cloud.google.com/spanner/docs/getting-started/ruby)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void CreateDatabase(google::cloud::spanner_admin::DatabaseAdminClient client,
                        std::string const& project_id,
                        std::string const& instance_id,
                        std::string const& database_id) {
      google::cloud::spanner::Database database(project_id, instance_id,
                                                database_id);
      google::spanner::admin::database::v1::CreateDatabaseRequest request;
      request.set_parent(database.instance().FullName());
      request.set_create_statement("CREATE DATABASE `" + database.database_id() +
                                   "`");
      request.add_extra_statements(R"""(
          CREATE TABLE Singers (
              SingerId   INT64 NOT NULL,
              FirstName  STRING(1024),
              LastName   STRING(1024),
              SingerInfo BYTES(MAX),
              FullName   STRING(2049)
                  AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED
          ) PRIMARY KEY (SingerId))""");
      request.add_extra_statements(R"""(
          CREATE TABLE Albums (
              SingerId     INT64 NOT NULL,
              AlbumId      INT64 NOT NULL,
              AlbumTitle   STRING(MAX)
          ) PRIMARY KEY (SingerId, AlbumId),
              INTERLEAVE IN PARENT Singers ON DELETE CASCADE)""");
      auto db = client.CreateDatabase(request).get();
      if (!db) throw std::move(db).status();
      std::cout << "Database " << db->name() << " created.\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Data;
    using System.Threading.Tasks;
    
    public class CreateDatabaseAsyncSample
    {
        public async Task CreateDatabaseAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}";
    
            using var connection = new SpannerConnection(connectionString);
            var createDatabase = $"CREATE DATABASE `{databaseId}`";
            // Define create table statement for table #1.
            var createSingersTable =
                @"CREATE TABLE Singers (
                    SingerId INT64 NOT NULL,
                    FirstName STRING(1024),
                    LastName STRING(1024),
                    ComposerInfo BYTES(MAX),
                    FullName STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], "" "")) STORED
                ) PRIMARY KEY (SingerId)";
            // Define create table statement for table #2.
            var createAlbumsTable =
                @"CREATE TABLE Albums (
                    SingerId INT64 NOT NULL,
                    AlbumId INT64 NOT NULL,
                    AlbumTitle STRING(MAX)
                ) PRIMARY KEY (SingerId, AlbumId),
                INTERLEAVE IN PARENT Singers ON DELETE CASCADE";
    
            using var createDbCommand = connection.CreateDdlCommand(createDatabase, createSingersTable, createAlbumsTable);
            await createDbCommand.ExecuteNonQueryAsync();
        }
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import (
     "context"
     "fmt"
     "io"
     "regexp"
    
     database "cloud.google.com/go/spanner/admin/database/apiv1"
     adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
    )
    
    func createDatabase(ctx context.Context, w io.Writer, db string) error {
     matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
     if matches == nil || len(matches) != 3 {
         return fmt.Errorf("Invalid database id %s", db)
     }
    
     adminClient, err := database.NewDatabaseAdminClient(ctx)
     if err != nil {
         return err
     }
     defer adminClient.Close()
    
     op, err := adminClient.CreateDatabase(ctx, &adminpb.CreateDatabaseRequest{
         Parent:          matches[1],
         CreateStatement: "CREATE DATABASE `" + matches[2] + "`",
         ExtraStatements: []string{
             `CREATE TABLE Singers (
                 SingerId   INT64 NOT NULL,
                 FirstName  STRING(1024),
                 LastName   STRING(1024),
                 SingerInfo BYTES(MAX),
                 FullName   STRING(2048) AS (
                     ARRAY_TO_STRING([FirstName, LastName], " ")
                 ) STORED
             ) PRIMARY KEY (SingerId)`,
             `CREATE TABLE Albums (
                 SingerId     INT64 NOT NULL,
                 AlbumId      INT64 NOT NULL,
                 AlbumTitle   STRING(MAX)
             ) PRIMARY KEY (SingerId, AlbumId),
             INTERLEAVE IN PARENT Singers ON DELETE CASCADE`,
         },
     })
     if err != nil {
         return err
     }
     if _, err := op.Wait(ctx); err != nil {
         return err
     }
     fmt.Fprintf(w, "Created database [%s]\n", db)
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    /**
     * TODO(developer): Uncomment the following lines before running the sample.
     */
    // const projectId = 'my-project-id';
    // const instanceId = 'my-instance';
    // const databaseId = 'my-database';
    
    // Imports the Google Cloud client library
    const {Spanner} = require('@google-cloud/spanner');
    
    // creates a client
    const spanner = new Spanner({
      projectId: projectID,
    });
    
    const databaseAdminClient = spanner.getDatabaseAdminClient();
    
    const createSingersTableStatement = `
    CREATE TABLE Singers (
      SingerId    INT64 NOT NULL,
      FirstName   STRING(1024),
      LastName    STRING(1024),
      SingerInfo  BYTES(MAX),
      FullName    STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED,
    ) PRIMARY KEY (SingerId)`;
    const createAlbumsTableStatement = `
    CREATE TABLE Albums (
      SingerId    INT64 NOT NULL,
      AlbumId     INT64 NOT NULL,
      AlbumTitle  STRING(MAX)
    ) PRIMARY KEY (SingerId, AlbumId),
    INTERLEAVE IN PARENT Singers ON DELETE CASCADE`;
    
    // Creates a new database
    try {
      const [operation] = await databaseAdminClient.createDatabase({
        createStatement: 'CREATE DATABASE `' + databaseID + '`',
        extraStatements: [
          createSingersTableStatement,
          createAlbumsTableStatement,
        ],
        parent: databaseAdminClient.instancePath(projectID, instanceID),
      });
    
      console.log(`Waiting for creation of ${databaseID} to complete...`);
      await operation.promise();
    
      console.log(`Created database ${databaseID} on instance ${instanceID}.`);
    } catch (err) {
      console.error('Failed to create database:', err.message || err);
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Rust

    use google_cloud_lro::Poller;
    use google_cloud_spanner_admin_database_v1::client::DatabaseAdmin;
    
    pub async fn sample(
        admin_client: &DatabaseAdmin,
        instance_name: &str,
        database_id: &str,
    ) -> anyhow::Result<()> {
        let create_statement = format!("CREATE DATABASE `{database_id}`");
        let extra_statements = vec![
            r#"CREATE TABLE Singers (
                SingerId INT64 NOT NULL,
                FirstName STRING(1024),
                LastName STRING(1024),
                SingerInfo BYTES(MAX),
                FullName STRING(2048) AS (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED
             ) PRIMARY KEY (SingerId)"#
                .to_string(),
            r#"CREATE TABLE Albums (
                SingerId INT64 NOT NULL,
                AlbumId INT64 NOT NULL,
                AlbumTitle STRING(MAX)
             ) PRIMARY KEY (SingerId, AlbumId),
             INTERLEAVE IN PARENT Singers ON DELETE CASCADE"#
                .to_string(),
        ];
    
        println!("Creating database {database_id}...");
        let database = admin_client
            .create_database()
            .set_parent(instance_name.to_string())
            .set_create_statement(create_statement)
            .set_extra_statements(extra_statements)
            .poller()
            .until_done()
            .await?;
    
        println!("Created database [{}] successfully.", database.name);
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
