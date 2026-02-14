Create a database using a property graph.

## Explore further

For detailed documentation that includes this code sample, see the following:

  - [Set up and query Spanner Graph](/spanner/docs/graph/set-up)

## Code sample

### C++

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` cpp
void CreateDatabaseWithPropertyGraph(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  google::spanner::admin::database::v1::CreateDatabaseRequest request;
  request.set_parent(database.instance().FullName());
  request.set_create_statement("CREATE DATABASE `" + database.database_id() +
                               "`");
  request.add_extra_statements(R"""(
    CREATE TABLE Person (
      id               INT64 NOT NULL,
      name             STRING(MAX),
      birthday         TIMESTAMP,
      country          STRING(MAX),
      city             STRING(MAX),
    ) PRIMARY KEY (id))""");
  request.add_extra_statements(R"""(
    CREATE TABLE Account (
      id               INT64 NOT NULL,
      create_time      TIMESTAMP,
      is_blocked       BOOL,
      nick_name        STRING(MAX),
    ) PRIMARY KEY (id))""");
  request.add_extra_statements(R"""(
    CREATE TABLE PersonOwnAccount (
      id               INT64 NOT NULL,
      account_id       INT64 NOT NULL,
      create_time      TIMESTAMP,
      FOREIGN KEY (account_id)
      REFERENCES Account (id)
    ) PRIMARY KEY (id, account_id),
      INTERLEAVE IN PARENT Person ON DELETE CASCADE)""");
  request.add_extra_statements(R"""(
    CREATE TABLE AccountTransferAccount (
      id               INT64 NOT NULL,
      to_id            INT64 NOT NULL,
      amount           FLOAT64,
      create_time      TIMESTAMP NOT NULL,
      order_number     STRING(MAX),
      FOREIGN KEY (to_id) REFERENCES Account (id)
    ) PRIMARY KEY (id, to_id, create_time),
      INTERLEAVE IN PARENT Account ON DELETE CASCADE)""");
  request.add_extra_statements(R"""(
    CREATE OR REPLACE PROPERTY GRAPH FinGraph
      NODE TABLES (Account, Person)
      EDGE TABLES (
        PersonOwnAccount
          SOURCE KEY(id) REFERENCES Person(id)
          DESTINATION KEY(account_id) REFERENCES Account(id)
          LABEL Owns,
        AccountTransferAccount
          SOURCE KEY(id) REFERENCES Account(id)
          DESTINATION KEY(to_id) REFERENCES Account(id)
          LABEL Transfers))""");
  auto db = client.CreateDatabase(request).get();
  if (!db) throw std::move(db).status();
  std::cout << "Database " << db->name() << " created with property graph.\n";
}
```

### Go

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "google.golang.org/genproto/googleapis/spanner/admin/database/v1"
)

func createDatabaseWithPropertyGraph(ctx context.Context, w io.Writer, dbId string) error {
 // dbId is of the form:
 //   projects/YOUR_PROJECT_ID/instances/YOUR_INSTANCE_ID/databases/YOUR_DATABASE_NAME
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(dbId)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("Invalid database id %s", dbId)
 }

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 var instance = matches[1]
 var dbName = matches[2]

 // The schema defintion for a database with a property graph comprises table
 // definitions one or more `CREATE PROPERTY GRAPH` statements to define the
 // property graph(s).
 //
 // Here, tables are created for 'Person's and 'Account's. The property graph
 // definition says that these entities form nodes in the graph. Similarly,
 // there are 'PersonOwnAccount' and 'AccountTransferAccount' relationship
 // tables defined. The property graph definition maps these to edges in the graph.
 var schema_statements = []string{
     `CREATE TABLE Person (
         id               INT64 NOT NULL,
         name             STRING(MAX),
         birthday         TIMESTAMP,
         country          STRING(MAX),
         city             STRING(MAX),
     ) PRIMARY KEY (id)`,
     `CREATE TABLE Account (
         id               INT64 NOT NULL,
         create_time      TIMESTAMP,
         is_blocked       BOOL,
         nick_name        STRING(MAX),
     ) PRIMARY KEY (id)`,
     `CREATE TABLE PersonOwnAccount (
         id               INT64 NOT NULL,
         account_id       INT64 NOT NULL,
         create_time      TIMESTAMP,
         FOREIGN KEY (account_id)
             REFERENCES Account (id)
     ) PRIMARY KEY (id, account_id),
     INTERLEAVE IN PARENT Person ON DELETE CASCADE`,
     `CREATE TABLE AccountTransferAccount (
         id               INT64 NOT NULL,
         to_id            INT64 NOT NULL,
         amount           FLOAT64,
         create_time      TIMESTAMP NOT NULL,
         order_number     STRING(MAX),
         FOREIGN KEY (to_id) REFERENCES Account (id)
     ) PRIMARY KEY (id, to_id, create_time),
     INTERLEAVE IN PARENT Account ON DELETE CASCADE`,
     `CREATE OR REPLACE PROPERTY GRAPH FinGraph
         NODE TABLES (Account, Person)
         EDGE TABLES (
             PersonOwnAccount
                 SOURCE KEY(id) REFERENCES Person(id)
                 DESTINATION KEY(account_id) REFERENCES Account(id)
                 LABEL Owns,
             AccountTransferAccount
                 SOURCE KEY(id) REFERENCES Account(id)
                 DESTINATION KEY(to_id) REFERENCES Account(id)
                 LABEL Transfers)`,
 }

 op, err := adminClient.CreateDatabase(ctx, &adminpb.CreateDatabaseRequest{
     Parent:          instance,
     CreateStatement: "CREATE DATABASE `" + dbName + "`",
     ExtraStatements: schema_statements,
 })
 if err != nil {
     return err
 }
 if _, err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Created database [%s]\n", dbId)
 return nil
}
```

### Java

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` java
static void createDatabaseWithPropertyGraph(
    DatabaseAdminClient dbAdminClient, InstanceName instanceName, String databaseId) {
  CreateDatabaseRequest createDatabaseRequest =
      CreateDatabaseRequest.newBuilder()
          .setCreateStatement("CREATE DATABASE `" + databaseId + "`")
          .setParent(instanceName.toString())
          .addAllExtraStatements(
              Arrays.asList(
                  "CREATE TABLE Person ("
                      + "  id               INT64 NOT NULL,"
                      + "  name             STRING(MAX),"
                      + "  birthday         TIMESTAMP,"
                      + "  country          STRING(MAX),"
                      + "  city             STRING(MAX),"
                      + ") PRIMARY KEY (id)",
                  "CREATE TABLE Account ("
                      + "  id               INT64 NOT NULL,"
                      + "  create_time      TIMESTAMP,"
                      + "  is_blocked       BOOL,"
                      + "  nick_name        STRING(MAX),"
                      + ") PRIMARY KEY (id)",
                  "CREATE TABLE PersonOwnAccount ("
                      + "  id               INT64 NOT NULL,"
                      + "  account_id       INT64 NOT NULL,"
                      + "  create_time      TIMESTAMP,"
                      + "  FOREIGN KEY (account_id)"
                      + "  REFERENCES Account (id)"
                      + ") PRIMARY KEY (id, account_id),"
                      + "INTERLEAVE IN PARENT Person ON DELETE CASCADE",
                  "CREATE TABLE AccountTransferAccount ("
                      + "  id               INT64 NOT NULL,"
                      + "  to_id            INT64 NOT NULL,"
                      + "  amount           FLOAT64,"
                      + "  create_time      TIMESTAMP NOT NULL,"
                      + "  order_number     STRING(MAX),"
                      + "  FOREIGN KEY (to_id) REFERENCES Account (id)"
                      + ") PRIMARY KEY (id, to_id, create_time),"
                      + "INTERLEAVE IN PARENT Account ON DELETE CASCADE",
                  "CREATE OR REPLACE PROPERTY GRAPH FinGraph "
                      + "NODE TABLES (Account, Person)"
                      + "EDGE TABLES ("
                      + "  PersonOwnAccount"
                      + "    SOURCE KEY(id) REFERENCES Person(id)"
                      + "    DESTINATION KEY(account_id) REFERENCES Account(id)"
                      + "    LABEL Owns,"
                      + "  AccountTransferAccount"
                      + "    SOURCE KEY(id) REFERENCES Account(id)"
                      + "    DESTINATION KEY(to_id) REFERENCES Account(id)"
                      + "    LABEL Transfers)"))
          .build();
  try {
    // Initiate the request which returns an OperationFuture.
    com.google.spanner.admin.database.v1.Database db =
        dbAdminClient.createDatabaseAsync(createDatabaseRequest).get();
    System.out.println("Created database [" + db.getName() + "]");
  } catch (ExecutionException e) {
    // If the operation failed during execution, expose the cause.
    System.out.println("Encountered exception" + e.getCause());
    throw (SpannerException) e.getCause();
  } catch (InterruptedException e) {
    // Throw when a thread is waiting, sleeping, or otherwise occupied,
    // and the thread is interrupted, either before or during the activity.
    throw SpannerExceptionFactory.propagateInterrupt(e);
  }
}
```

### Python

To learn how to install and use the client library for Spanner, see [Spanner client libraries](/spanner/docs/reference/libraries) .

To authenticate to Spanner, set up Application Default Credentials. For more information, see [Set up authentication for a local development environment](/docs/authentication/set-up-adc-local-dev-environment) .

``` python
def create_database_with_property_graph(instance_id, database_id):
    """Creates a database, tables and a property graph for sample data."""
    from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

    spanner_client = spanner.Client()
    database_admin_api = spanner_client.database_admin_api

    request = spanner_database_admin.CreateDatabaseRequest(
        parent=database_admin_api.instance_path(spanner_client.project, instance_id),
        create_statement=f"CREATE DATABASE `{database_id}`",
        extra_statements=[
            """CREATE TABLE Person (
            id               INT64 NOT NULL,
            name             STRING(MAX),
            birthday         TIMESTAMP,
            country          STRING(MAX),
            city             STRING(MAX),
        ) PRIMARY KEY (id)""",
            """CREATE TABLE Account (
            id               INT64 NOT NULL,
            create_time      TIMESTAMP,
            is_blocked       BOOL,
            nick_name        STRING(MAX),
        ) PRIMARY KEY (id)""",
            """CREATE TABLE PersonOwnAccount (
            id               INT64 NOT NULL,
            account_id       INT64 NOT NULL,
            create_time      TIMESTAMP,
            FOREIGN KEY (account_id)
                REFERENCES Account (id)
        ) PRIMARY KEY (id, account_id),
        INTERLEAVE IN PARENT Person ON DELETE CASCADE""",
            """CREATE TABLE AccountTransferAccount (
            id               INT64 NOT NULL,
            to_id            INT64 NOT NULL,
            amount           FLOAT64,
            create_time      TIMESTAMP NOT NULL,
            order_number     STRING(MAX),
            FOREIGN KEY (to_id) REFERENCES Account (id)
        ) PRIMARY KEY (id, to_id, create_time),
        INTERLEAVE IN PARENT Account ON DELETE CASCADE""",
            """CREATE OR REPLACE PROPERTY GRAPH FinGraph
            NODE TABLES (Account, Person)
            EDGE TABLES (
                PersonOwnAccount
                    SOURCE KEY(id) REFERENCES Person(id)
                    DESTINATION KEY(account_id) REFERENCES Account(id)
                    LABEL Owns,
                AccountTransferAccount
                    SOURCE KEY(id) REFERENCES Account(id)
                    DESTINATION KEY(to_id) REFERENCES Account(id)
                    LABEL Transfers)""",
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

## What's next

To search and filter code samples for other Google Cloud products, see the [Google Cloud sample browser](/docs/samples?product=spanner) .
