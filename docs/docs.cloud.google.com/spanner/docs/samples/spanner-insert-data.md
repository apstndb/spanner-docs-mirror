---
name: documents/docs.cloud.google.com/spanner/docs/samples/spanner-insert-data
uri: https://docs.cloud.google.com/spanner/docs/samples/spanner-insert-data
title: Mutations write data
description: Insert several rows of data into a table by using mutations.
data_source: docs.cloud.google.com
---

Insert several rows of data into a table by using mutations.

## Explore further

For detailed documentation that includes this code sample, see the following:

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
  - [Insert, update, and delete data using mutations](https://docs.cloud.google.com/spanner/docs/modify-mutation-api)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    void InsertData(google::cloud::spanner::Client client) {
      namespace spanner = ::google::cloud::spanner;
      auto insert_singers = spanner::InsertMutationBuilder(
                                "Singers", {"SingerId", "FirstName", "LastName"})
                                .EmplaceRow(1, "Marc", "Richards")
                                .EmplaceRow(2, "Catalina", "Smith")
                                .EmplaceRow(3, "Alice", "Trentor")
                                .EmplaceRow(4, "Lea", "Martin")
                                .EmplaceRow(5, "David", "Lomond")
                                .Build();
    
      auto insert_albums = spanner::InsertMutationBuilder(
                               "Albums", {"SingerId", "AlbumId", "AlbumTitle"})
                               .EmplaceRow(1, 1, "Total Junk")
                               .EmplaceRow(1, 2, "Go, Go, Go")
                               .EmplaceRow(2, 1, "Green")
                               .EmplaceRow(2, 2, "Forever Hold Your Peace")
                               .EmplaceRow(2, 3, "Terrified")
                               .Build();
    
      auto commit_result =
          client.Commit(spanner::Mutations{insert_singers, insert_albums});
      if (!commit_result) throw std::move(commit_result).status();
      std::cout << "Insert was successful [spanner_insert_data]\n";
    }

### C\#

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    struct Singer
    {
        internal long SingerId;
        internal string FirstName;
        internal string LastName;
    }
    
    struct Album
    {
        internal long SingerId;
        internal long AlbumId;
        internal string Title;
    }
    
    public static async Task WriteDataWithMutations(string connectionString)
    {
        await using var connection = new SpannerConnection(connectionString);
        await connection.OpenAsync();
    
        Singer[] singers =
        [
            new() {SingerId=1, FirstName = "Marc", LastName = "Richards"},
            new() {SingerId=2, FirstName = "Catalina", LastName = "Smith"},
            new() {SingerId=3, FirstName = "Alice", LastName = "Trentor"},
            new() {SingerId=4, FirstName = "Lea", LastName = "Martin"},
            new() {SingerId=5, FirstName = "David", LastName = "Lomond"},
        ];
        Album[] albums =
        [
            new() {SingerId = 1, AlbumId = 1, Title = "Total Junk"},
            new() {SingerId = 1, AlbumId = 2, Title = "Go, Go, Go"},
            new() {SingerId = 2, AlbumId = 1, Title = "Green"},
            new() {SingerId = 2, AlbumId = 2, Title = "Forever Hold Your Peace"},
            new() {SingerId = 2, AlbumId = 3, Title = "Terrified"},
        ];
        var batch = connection.CreateBatch();
        foreach (var singer in singers)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("Singers");
            command.AddParameter("SingerId", singer.SingerId);
            command.AddParameter("FirstName", singer.FirstName);
            command.AddParameter("LastName", singer.LastName);
            batch.BatchCommands.Add(command);
        }
        foreach (var album in albums)
        {
            // The name of a parameter must correspond with a column name.
            var command = batch.CreateInsertCommand("Albums");
            command.AddParameter("SingerId", album.SingerId);
            command.AddParameter("AlbumId", album.AlbumId);
            command.AddParameter("AlbumTitle", album.Title);
            batch.BatchCommands.Add(command);
        }
        var affected = await batch.ExecuteNonQueryAsync();
        Console.WriteLine($"Inserted {affected} rows.");
    }

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    func write(ctx context.Context, w io.Writer, client *spanner.Client) error {
     singerColumns := []string{"SingerId", "FirstName", "LastName"}
     albumColumns := []string{"SingerId", "AlbumId", "AlbumTitle"}
     m := []*spanner.Mutation{
         spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{1, "Marc", "Richards"}),
         spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{2, "Catalina", "Smith"}),
         spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{3, "Alice", "Trentor"}),
         spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{4, "Lea", "Martin"}),
         spanner.InsertOrUpdate("Singers", singerColumns, []interface{}{5, "David", "Lomond"}),
         spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{1, 1, "Total Junk"}),
         spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{1, 2, "Go, Go, Go"}),
         spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 1, "Green"}),
         spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 2, "Forever Hold Your Peace"}),
         spanner.InsertOrUpdate("Albums", albumColumns, []interface{}{2, 3, "Terrified"}),
     }
     _, err := client.Apply(ctx, m)
     return err
    }

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

    /** The list of Singers to insert. */
    static final List<Singer> SINGERS =
        Arrays.asList(
            new Singer(1, "Marc", "Richards"),
            new Singer(2, "Catalina", "Smith"),
            new Singer(3, "Alice", "Trentor"),
            new Singer(4, "Lea", "Martin"),
            new Singer(5, "David", "Lomond"));
    
    /** The list of Albums to insert. */
    static final List<Album> ALBUMS =
        Arrays.asList(
            new Album(1, 1, "Total Junk"),
            new Album(1, 2, "Go, Go, Go"),
            new Album(2, 1, "Green"),
            new Album(2, 2, "Forever Hold Your Peace"),
            new Album(2, 3, "Terrified"));
    
    static void writeDataWithMutations(
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
    
        List<Mutation> mutations = new ArrayList<>();
        for (Singer singer : SINGERS) {
          mutations.add(
              Mutation.newInsertBuilder("Singers")
                  .set("SingerId")
                  .to(singer.singerId)
                  .set("FirstName")
                  .to(singer.firstName)
                  .set("LastName")
                  .to(singer.lastName)
                  .build());
        }
        for (Album album : ALBUMS) {
          mutations.add(
              Mutation.newInsertBuilder("Albums")
                  .set("SingerId")
                  .to(album.singerId)
                  .set("AlbumId")
                  .to(album.albumId)
                  .set("AlbumTitle")
                  .to(album.albumTitle)
                  .build());
        }
        // Apply the mutations atomically to Spanner.
        cloudSpannerJdbcConnection.write(mutations);
        System.out.printf("Inserted %d rows.\n", mutations.size());
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
    
    // Instantiate Spanner table objects
    const singersTable = database.table('Singers');
    const albumsTable = database.table('Albums');
    
    // Inserts rows into the Singers table
    // Note: Cloud Spanner interprets Node.js numbers as FLOAT64s, so
    // they must be converted to strings before being inserted as INT64s
    try {
      await singersTable.insert([
        {SingerId: '1', FirstName: 'Marc', LastName: 'Richards'},
        {SingerId: '2', FirstName: 'Catalina', LastName: 'Smith'},
        {SingerId: '3', FirstName: 'Alice', LastName: 'Trentor'},
        {SingerId: '4', FirstName: 'Lea', LastName: 'Martin'},
        {SingerId: '5', FirstName: 'David', LastName: 'Lomond'},
      ]);
    
      await albumsTable.insert([
        {SingerId: '1', AlbumId: '1', AlbumTitle: 'Total Junk'},
        {SingerId: '1', AlbumId: '2', AlbumTitle: 'Go, Go, Go'},
        {SingerId: '2', AlbumId: '1', AlbumTitle: 'Green'},
        {SingerId: '2', AlbumId: '2', AlbumTitle: 'Forever Hold your Peace'},
        {SingerId: '2', AlbumId: '3', AlbumTitle: 'Terrified'},
      ]);
    
      console.log('Inserted data.');
    } catch (err) {
      console.error('ERROR:', err);
    } finally {
      await database.close();
    }

### PHP

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](https://docs.cloud.google.com/docs/authentication/set-up-adc-local-dev-environment) .

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

### Rust

    use google_cloud_spanner::client::DatabaseClient;
    use google_cloud_spanner::mutation::Mutation;
    
    pub async fn sample(client: &DatabaseClient) -> anyhow::Result<()> {
        let mutations = vec![
            Mutation::new_insert_builder("Singers")
                .set("SingerId")
                .to(&1)
                .set("FirstName")
                .to(&"Marc")
                .set("LastName")
                .to(&"Richards")
                .build(),
            Mutation::new_insert_builder("Singers")
                .set("SingerId")
                .to(&2)
                .set("FirstName")
                .to(&"Catalina")
                .set("LastName")
                .to(&"Smith")
                .build(),
            Mutation::new_insert_builder("Singers")
                .set("SingerId")
                .to(&3)
                .set("FirstName")
                .to(&"Alice")
                .set("LastName")
                .to(&"Trentor")
                .build(),
            Mutation::new_insert_builder("Singers")
                .set("SingerId")
                .to(&4)
                .set("FirstName")
                .to(&"Lea")
                .set("LastName")
                .to(&"Martin")
                .build(),
            Mutation::new_insert_builder("Singers")
                .set("SingerId")
                .to(&5)
                .set("FirstName")
                .to(&"David")
                .set("LastName")
                .to(&"Lomond")
                .build(),
            Mutation::new_insert_builder("Albums")
                .set("SingerId")
                .to(&1)
                .set("AlbumId")
                .to(&1)
                .set("AlbumTitle")
                .to(&"Total Junk")
                .build(),
            Mutation::new_insert_builder("Albums")
                .set("SingerId")
                .to(&1)
                .set("AlbumId")
                .to(&2)
                .set("AlbumTitle")
                .to(&"Go, Go, Go")
                .build(),
            Mutation::new_insert_builder("Albums")
                .set("SingerId")
                .to(&2)
                .set("AlbumId")
                .to(&1)
                .set("AlbumTitle")
                .to(&"Green")
                .build(),
            Mutation::new_insert_builder("Albums")
                .set("SingerId")
                .to(&2)
                .set("AlbumId")
                .to(&2)
                .set("AlbumTitle")
                .to(&"Forever Hold Your Peace")
                .build(),
            Mutation::new_insert_builder("Albums")
                .set("SingerId")
                .to(&2)
                .set("AlbumId")
                .to(&3)
                .set("AlbumTitle")
                .to(&"Terrified")
                .build(),
        ];
    
        println!("Inserting initial data into Singers & Albums...");
        let write_transaction = client.write_only_transaction().build();
        write_transaction.write(mutations).await?;
        println!("Inserted data successfully.");
    
        Ok(())
    }

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](https://docs.cloud.google.com/docs/samples?product=cloudspanner) .
