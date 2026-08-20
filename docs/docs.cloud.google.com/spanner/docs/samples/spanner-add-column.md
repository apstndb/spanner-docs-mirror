---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-add-column
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-add-column
title: Add column
description: Update a schema by adding a column.
data_source: docs.cloud.google.com
---

Update a schema by adding a column.

## Explore further

For detailed documentation that includes this code sample, see the following:

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

    void AddColumn(google::cloud::spanner_admin::DatabaseAdminClient client,
                   std::string const& project_id, std::string const& instance_id,
                   std::string const& database_id) {
      google::cloud::spanner::Database database(project_id, instance_id,
                                                database_id);
      auto metadata =
          client
              .UpdateDatabaseDdl(
                  database.FullName(),
                  {"ALTER TABLE Albums ADD COLUMN MarketingBudget INT64"})
              .get();
      google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
          client, database, metadata.status());              //! TODO(#4758)
      if (!metadata) throw std::move(metadata).status();
      std::cout << "Added MarketingBudget column\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    using Google.Cloud.Spanner.Data;
    using System;
    using System.Threading.Tasks;
    
    public class AddColumnAsyncSample
    {
        public async Task AddColumnAsync(string projectId, string instanceId, string databaseId)
        {
            string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
            string alterStatement = "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64";
    
            using var connection = new SpannerConnection(connectionString);
            using var updateCmd = connection.CreateDdlCommand(alterStatement);
            await updateCmd.ExecuteNonQueryAsync();
            Console.WriteLine("Added the MarketingBudget column.");
        }
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    import (
     "context"
     "database/sql"
     "fmt"
     "io"
    
     _ "github.com/googleapis/go-sql-spanner"
    )
    
    func AddColumn(ctx context.Context, w io.Writer, databaseName string) error {
     db, err := sql.Open("spanner", databaseName)
     if err != nil {
         return err
     }
     defer db.Close()
    
     _, err = db.ExecContext(ctx,
         `ALTER TABLE Albums
             ADD COLUMN MarketingBudget INT64`)
     if err != nil {
         return err
     }
    
     fmt.Fprint(w, "Added MarketingBudget column\n")
     return nil
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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
      projectId: projectId,
    });
    
    const databaseAdminClient = spanner.getDatabaseAdminClient();
    
    // Creates a new index in the database
    try {
      const [operation] = await databaseAdminClient.updateDatabaseDdl({
        database: databaseAdminClient.databasePath(
          projectId,
          instanceId,
          databaseId
        ),
        statements: ['ALTER TABLE Albums ADD COLUMN MarketingBudget INT64'],
      });
    
      console.log('Waiting for operation to complete...');
      await operation.promise();
    
      console.log('Added the MarketingBudget column.');
    } catch (err) {
      console.error('Failed to add column:', err.message || err);
    } finally {
      // Close the spanner client when finished.
      // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
      spanner.close();
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    function add_column(string $host, string $port, string $database): void
    {
        $dsn = sprintf("pgsql:host=%s;port=%s;dbname=%s", $host, $port, $database);
        $connection = new PDO($dsn);
    
        $connection->exec("ALTER TABLE albums ADD COLUMN marketing_budget bigint");
        print("Added marketing_budget column\n");
    
        $connection = null;
    }

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Rust

    use google_cloud_lro::Poller;
    use google_cloud_spanner_admin_database_v1::client::DatabaseAdmin;
    
    pub async fn sample(admin_client: &DatabaseAdmin, database_name: &str) -> anyhow::Result<()> {
        let statements = vec!["ALTER TABLE Albums ADD COLUMN MarketingBudget INT64"];
    
        admin_client
            .update_database_ddl()
            .set_database(database_name)
            .set_statements(statements)
            .poller()
            .until_done()
            .await?;
    
        println!("Added MarketingBudget column");
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
