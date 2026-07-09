---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-update-data
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-update-data
title: Mutations update data
description: Update data by using mutations.
data_source: docs.cloud.google.com
---

Update data by using mutations.

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

    void UpdateData(google::cloud::spanner::Client client) {
      //! [commit-with-mutations]
      namespace spanner = ::google::cloud::spanner;
      auto commit_result = client.Commit(spanner::Mutations{
          spanner::UpdateMutationBuilder("Albums",
                                         {"SingerId", "AlbumId", "MarketingBudget"})
              .EmplaceRow(1, 1, 100000)
              .EmplaceRow(2, 2, 500000)
              .Build()});
      if (!commit_result) throw std::move(commit_result).status();
      //! [commit-with-mutations]
      std::cout << "Update was successful [spanner_update_data]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    public static async Task UpdateDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        (long SingerId, long AlbumId, long MarketingBudget)[] albums = [
            (1L, 1L, 100000L),
            (2L, 2L, 500000L),
        ];
        // Use a batch to update two rows in one round-trip.
        var batch = connection.CreateBatch();
        foreach (var album in albums)
        {
            // This creates a command that will use a mutation to update the row.
            var command = batch.CreateUpdateCommand("Albums");
            command.AddParameter("SingerId", album.SingerId);
            command.AddParameter("AlbumId", album.AlbumId);
            command.AddParameter("MarketingBudget", album.MarketingBudget);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Updated {affected} albums.");
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    func update(ctx context.Context, w io.Writer, client *spanner.Client) error {
     cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
     _, err := client.Apply(ctx, []*spanner.Mutation{
         spanner.Update("Albums", cols, []interface{}{1, 1, 100000}),
         spanner.Update("Albums", cols, []interface{}{2, 2, 500000}),
     })
     return err
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    static void updateDataWithMutations(
        final String project,
        final String instance,
        final String database,
        final Properties properties) throws SQLException {
      try (Connection connection =
          DriverManager.getConnection(
              String.format(
                  "jdbc:cloudspanner:/projects/%s/instances/%s/databases/%s",
                  project, instance, database),
              properties)) {
        // Unwrap the CloudSpannerJdbcConnection interface
        // from the java.sql.Connection.
        CloudSpannerJdbcConnection cloudSpannerJdbcConnection =
            connection.unwrap(CloudSpannerJdbcConnection.class);
    
        final long marketingBudgetAlbum1 = 100000L;
        final long marketingBudgetAlbum2 = 500000L;
        // Mutation can be used to update/insert/delete a single row in a table.
        // Here we use newUpdateBuilder to create update mutations.
        List<Mutation> mutations =
            Arrays.asList(
                Mutation.newUpdateBuilder("Albums")
                    .set("SingerId")
                    .to(1)
                    .set("AlbumId")
                    .to(1)
                    .set("MarketingBudget")
                    .to(marketingBudgetAlbum1)
                    .build(),
                Mutation.newUpdateBuilder("Albums")
                    .set("SingerId")
                    .to(2)
                    .set("AlbumId")
                    .to(2)
                    .set("MarketingBudget")
                    .to(marketingBudgetAlbum2)
                    .build());
        // This writes all the mutations to Cloud Spanner atomically.
        cloudSpannerJdbcConnection.write(mutations);
        System.out.println("Updated albums");
      }
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    // Imports the Google Cloud client library
    const {Spanner} = require('@google-cloud/spanner');
    
    /**
     * TODO(developer): Uncomment the following lines before running the sample.
     */
    // const projectId = 'my-project-id';
    // const instanceId = 'my-instance';
    // const databaseId = 'my-database';
    
    // Creates a client
    const spanner = new Spanner({
      projectId: projectId,
    });
    
    // Gets a reference to a Cloud Spanner instance and database
    const instance = spanner.instance(instanceId);
    const database = instance.database(databaseId);
    
    // Update a row in the Albums table
    // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so they
    // must be converted to strings before being inserted as INT64s
    const albumsTable = database.table('Albums');
    
    try {
      await albumsTable.update([
        {SingerId: '1', AlbumId: '1', MarketingBudget: '100000'},
        {SingerId: '2', AlbumId: '2', MarketingBudget: '500000'},
      ]);
      console.log('Updated data.');
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      // Close the database when finished.
      await database.close();
    }

### Node.js

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Ruby

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::mutation::Mutation;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let mutations = vec![
            Mutation::new_update_builder("Albums")
                .set("SingerId")
                .to(&1)
                .set("AlbumId")
                .to(&1)
                .set("MarketingBudget")
                .to(&100000)
                .build(),
            Mutation::new_update_builder("Albums")
                .set("SingerId")
                .to(&2)
                .set("AlbumId")
                .to(&2)
                .set("MarketingBudget")
                .to(&500000)
                .build(),
        ];
    
        println!("Updating MarketingBudget on Albums...");
        let write_transaction = client.write_only_transaction().build();
        write_transaction.write(mutations).await?;
        println!("Updated budget successfully.");
    
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
