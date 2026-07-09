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

    public static async Task CreateTables(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        // Create two tables in one batch on Spanner.
        var batch = connection.CreateBatch();
        batch.BatchCommands.Add("CREATE TABLE Singers (" +
                                "  SingerId   INT64 NOT NULL, " +
                                "  FirstName  STRING(1024), " +
                                "  LastName   STRING(1024), " +
                                "  SingerInfo BYTES(MAX) " +
                                ") PRIMARY KEY (SingerId)");
        batch.BatchCommands.Add("CREATE TABLE Albums ( " +
                                "  SingerId     INT64 NOT NULL, " +
                                "  AlbumId      INT64 NOT NULL, " +
                                "  AlbumTitle   STRING(MAX)" +
                                ") PRIMARY KEY (SingerId, AlbumId), " +
                                "INTERLEAVE IN PARENT Singers ON DELETE CASCADE");
        await batch.ExecuteNonQueryAsync();
        Console.WriteLine("Created Singers & Albums tables");
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    func createDatabase(ctx context.Context, w io.Writer, adminClient *database.DatabaseAdminClient, db string) error {
     matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
     if matches == nil || len(matches) != 3 {
         return fmt.Errorf("Invalid database id %s", db)
     }
     op, err := adminClient.CreateDatabase(ctx, &adminpb.CreateDatabaseRequest{
         Parent:          matches[1],
         CreateStatement: "CREATE DATABASE `" + matches[2] + "`",
         ExtraStatements: []string{
             `CREATE TABLE Singers (
                 SingerId   INT64 NOT NULL,
                 FirstName  STRING(1024),
                 LastName   STRING(1024),
                 SingerInfo BYTES(MAX)
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

    static void createDatabase(
        final DatabaseAdminClient dbAdminClient,
        final InstanceName instanceName,
        final String databaseId,
        final Properties properties) throws SQLException {
      // Use the Spanner admin client to create a database.
      CreateDatabaseRequest createDatabaseRequest =
          CreateDatabaseRequest.newBuilder()
              .setCreateStatement("CREATE DATABASE `" + databaseId + "`")
              .setParent(instanceName.toString())
              .build();
      try {
        dbAdminClient.createDatabaseAsync(createDatabaseRequest).get();
      } catch (ExecutionException e) {
        throw SpannerExceptionFactory.asSpannerException(e.getCause());
      } catch (InterruptedException e) {
        throw SpannerExceptionFactory.propagateInterrupt(e);
      }
    
      // Connect to the database with the JDBC driver and create two test tables.
      String projectId = instanceName.getProject();
      String instanceId = instanceName.getInstance();
      try (Connection connection =
          DriverManager.getConnection(
              String.format(
                  "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
                  projectId, instanceId, databaseId),
              properties)) {
        try (Statement statement = connection.createStatement()) {
          // Create the tables in one batch.
          statement.addBatch(
              "CREATE TABLE Singers ("
                  + "  SingerId   INT64 NOT NULL,"
                  + "  FirstName  STRING(1024),"
                  + "  LastName   STRING(1024),"
                  + "  SingerInfo BYTES(MAX),"
                  + "  FullName STRING(2048) AS "
                  + "  (ARRAY_TO_STRING([FirstName, LastName], \" \")) STORED"
                  + ") PRIMARY KEY (SingerId)");
          statement.addBatch(
              "CREATE TABLE Albums ("
                  + "  SingerId     INT64 NOT NULL,"
                  + "  AlbumId      INT64 NOT NULL,"
                  + "  AlbumTitle   STRING(MAX)"
                  + ") PRIMARY KEY (SingerId, AlbumId),"
                  + "  INTERLEAVE IN PARENT Singers ON DELETE CASCADE");
          statement.executeBatch();
        }
      }
      System.out.printf(
          "Created database [%s]\n",
          DatabaseName.of(projectId, instanceId, databaseId));
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

    use Google\Cloud\Spanner\SpannerClient;
    
    /**
     * Creates a database and tables for sample data.
     * Example:
     * ```
     * create_database($instanceId, $databaseId);
     * ```
     *
     * @param string $instanceId The Spanner instance ID.
     * @param string $databaseId The Spanner database ID.
     */
    function create_database(string $instanceId, string $databaseId): void
    {
        $spanner = new SpannerClient();
        $instance = $spanner->instance($instanceId);
    
        if (!$instance->exists()) {
            throw new \LogicException("Instance $instanceId does not exist");
        }
    
        $operation = $instance->createDatabase($databaseId, ['statements' => [
            'CREATE TABLE Singers (
                SingerId     INT64 NOT NULL,
                FirstName    STRING(1024),
                LastName     STRING(1024),
                SingerInfo   BYTES(MAX),
                FullName     STRING(2048) AS
                (ARRAY_TO_STRING([FirstName, LastName], " ")) STORED
            ) PRIMARY KEY (SingerId)',
            'CREATE TABLE Albums (
                SingerId     INT64 NOT NULL,
                AlbumId      INT64 NOT NULL,
                AlbumTitle   STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE'
        ]]);
    
        print('Waiting for operation to complete...' . PHP_EOL);
        $operation->pollUntilComplete();
    
        printf('Created database %s on instance %s' . PHP_EOL,
            $databaseId, $instanceId);
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    def create_database(instance_id, database_id):
        """Creates a database and tables for sample data."""
        spanner_client = spanner.Client()
        instance = spanner_client.instance(instance_id)
    
        database = instance.database(
            database_id,
            ddl_statements=[
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
    
        operation = database.create()
    
        print("Waiting for operation to complete...")
        operation.result(OPERATION_TIMEOUT_SECONDS)
    
        print("Created database {} on instance {}".format(database_id, instance_id))

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
