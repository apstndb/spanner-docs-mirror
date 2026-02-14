## Objectives

This tutorial walks you through the following steps using the Spanner client library for Go:

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

## Prepare your local Go environment

1.  Install Go ( [download](https://golang.org/doc/install) ) on your development machine if it is not already installed.

2.  Configure the `  GOPATH  ` environment variable if it is not already configured, as described in [Test your installation](https://golang.org/doc/install#testing) .

3.  Download the samples to your machine.
    
    ``` text
    git clone https://github.com/GoogleCloudPlatform/golang-samples $GOPATH/src/github.com/GoogleCloudPlatform/golang-samples
    ```

4.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd $GOPATH/src/github.com/GoogleCloudPlatform/golang-samples/spanner/spanner_snippets
    ```

5.  Set the `  PROJECT_ID  ` environment variable to your Google Cloud project ID:
    
    ``` text
    export PROJECT_ID=[MY_PROJECT_ID]
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with Go.

Take a look through the `  snippet.go  ` file, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
go run snippet.go createdatabase projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgcreatedatabase projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see:

``` text
Created database [example-db]
```

The following code creates a database and two tables in the database.

**Note:** The subsequent code samples use these two tables. If you don't execute this code, then create the tables by using the Google Cloud console or the gcloud CLI. For more information, see the [example schema](/spanner/docs/schema-and-data-model#creating-interleaved-tables) .

### GoogleSQL

``` go
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
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"
 "regexp"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// pgCreateDatabase shows how to create a Spanner database that uses the
// PostgreSQL dialect.
func pgCreateDatabase(ctx context.Context, w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 matches := regexp.MustCompile("^(.*)/databases/(.*)$").FindStringSubmatch(db)
 if matches == nil || len(matches) != 3 {
     return fmt.Errorf("invalid database id %v", db)
 }

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // Databases with PostgreSQL dialect do not support extra DDL statements in the `CreateDatabase` call.
 req := &adminpb.CreateDatabaseRequest{
     Parent:          matches[1],
     DatabaseDialect: adminpb.DatabaseDialect_POSTGRESQL,
     // Note that PostgreSQL uses double quotes for quoting identifiers. This also
     // includes database names in the CREATE DATABASE statement.
     CreateStatement: `CREATE DATABASE "` + matches[2] + `"`,
 }
 opCreate, err := adminClient.CreateDatabase(ctx, req)
 if err != nil {
     return err
 }
 if _, err := opCreate.Wait(ctx); err != nil {
     return err
 }
 updateReq := &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         `CREATE TABLE Singers (
             SingerId   bigint NOT NULL PRIMARY KEY,
             FirstName  varchar(1024),
             LastName   varchar(1024),
             SingerInfo bytea
         )`,
         `CREATE TABLE Albums (
             AlbumId      bigint NOT NULL,
             SingerId     bigint NOT NULL REFERENCES Singers (SingerId),
             AlbumTitle   text,
                PRIMARY KEY(SingerId, AlbumId)
         )`,
         `CREATE TABLE Venues (
             VenueId  bigint NOT NULL PRIMARY KEY,
             Name     varchar(1024) NOT NULL
         )`,
     },
 }
 opUpdate, err := adminClient.UpdateDatabaseDdl(ctx, updateReq)
 if err != nil {
     return err
 }
 if err := opUpdate.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Created Spanner PostgreSQL database [%v]\n", db)
 return nil
}
```

The next step is to write data to your database.

## Create a database client

Before you can do reads or writes, you must create a [`  Client  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client) :

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
 database "cloud.google.com/go/spanner/admin/database/apiv1"
)

func createClients(w io.Writer, db string) error {
 ctx := context.Background()

 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 dataClient, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer dataClient.Close()

 _ = adminClient
 _ = dataClient

 return nil
}
```

You can think of a `  Client  ` as a database connection: all of your interactions with Spanner must go through a `  Client  ` . Typically you create a `  Client  ` when your application starts up, then you re-use that `  Client  ` to read, write, and execute transactions. Each client uses resources in Spanner.

If you create multiple clients in the same app, you should call [`  Client.Close()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.Close) to clean up the client's resources, including network connections, as soon as it is no longer needed.

Read more in the [`  Client  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client) reference.

The code in the previous example also shows how to create a [`  DatabaseAdminClient  `](https://pkg.go.dev/cloud.google.com/go/spanner//admin/database/apiv1#DatabaseAdminClient) , which is used to create a database.

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  Update()  ` method to execute a DML statement.

### GoogleSQL

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func writeUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT Singers (SingerId, FirstName, LastName) VALUES
             (12, 'Melissa', 'Garcia'),
             (13, 'Russell', 'Morales'),
             (14, 'Jacqueline', 'Long'),
             (15, 'Dylan', 'Shaw')`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return err
 })
 return err
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func pgWriteUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     stmt := spanner.Statement{
         SQL: `INSERT INTO Singers (SingerId, FirstName, LastName) VALUES
             (12, 'Melissa', 'Garcia'),
             (13, 'Russell', 'Morales'),
             (14, 'Jacqueline', 'Long'),
             (15, 'Dylan', 'Shaw')`,
     }
     rowCount, err := txn.Update(ctx, stmt)
     if err != nil {
         return err
     }
     fmt.Fprintf(w, "%d record(s) inserted.\n", rowCount)
     return err
 })
 return err
}
```

Run the sample using the `  dmlwrite  ` argument for Google SQL and the `  pgdmlwrite  ` argument for PostgreSQL:

### GoogleSQL

``` text
go run snippet.go dmlwrite projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgdmlwrite projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see:

``` text
4 record(s) inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

A [`  Mutation  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Mutation) is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a Spanner database.

Use [`  Mutation.InsertOrUpdate()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#InsertOrUpdate) to construct an `  INSERT_OR_UPDATE  ` mutation, which adds a new row or updates column values if the row already exists. Alternatively, use the [`  Mutation.Insert()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Insert) method to construct an `  INSERT  ` mutation, which adds a new row.

[`  Client.Apply()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.Apply) applies mutations atomically to a database.

This code shows how to write the data using mutations:

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func write(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

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
 _, err = client.Apply(ctx, m)
 return err
}
```

Run the sample using the `  write  ` argument:

``` text
go run snippet.go write projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see the command run successfully.

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Go.

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

### Use the Spanner client library for Go

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner client library for Go.

The following methods and types are used to run the SQL query:

  - [`  Client.Single()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.Single) : use this to read the value of one or more columns from one or more rows in a Spanner table. `  Client.Single  ` returns a [`  ReadOnlyTransaction  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadOnlyTransaction) , which is used for running a read or SQL statement.
  - [`  ReadOnlyTransaction.Query()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadOnlyTransaction.Query) : use this method to execute a query against a database.
  - The [`  Statement  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Statement) type: use this to construct a SQL string.
  - The [`  Row  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Row) type: use this to access the data returned by a SQL statement or read call.

Here's how to issue the query and access the data:

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func query(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

Run the sample using the `  query  ` argument.

``` text
go run snippet.go query projects/PROJECT_ID/instances/test-instance/databases/example-db
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

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryWithParameter(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT SingerId, FirstName, LastName FROM Singers
         WHERE LastName = @lastName`,
     Params: map[string]interface{}{
         "lastName": "Garcia",
     },
 }
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID int64
     var firstName, lastName string
     if err := row.Columns(&singerID, &firstName, &lastName); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", singerID, firstName, lastName)
 }
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

// pgQueryParameter shows how to execute a query with parameters on a Spanner
// PostgreSQL database. The PostgreSQL dialect uses positional parameters, as
// opposed to the named parameters of Cloud Spanner.
func pgQueryParameter(w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{
     SQL: `SELECT SingerId, FirstName, LastName FROM Singers
         WHERE LastName = $1`,
     Params: map[string]interface{}{
         "p1": "Garcia",
     },
 }
 type Singers struct {
     SingerID            int64
     FirstName, LastName string
 }
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var val Singers
     if err := row.ToStruct(&val); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s %s\n", val.SingerID, val.FirstName, val.LastName)
 }
}
```

Run the sample using the `  querywithparameter  ` argument for Google SQL and the `  pgqueryparameter  ` argument for PostgreSQL.

### GoogleSQL

``` text
go run snippet.go querywithparameter projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgqueryparameter projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see output similar to:

``` text
12 Melissa Garcia
```

## Read data using the read API

In addition to Spanner's SQL interface, Spanner also supports a read interface.

Use [`  ReadOnlyTransaction.Read()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadOnlyTransaction.Read) to read rows from the database. Use [`  KeySet  `](https://pkg.go.dev/cloud.google.com/go/spanner/#KeySet) to define a collection of keys and key ranges to read.

Here's how to read the data:

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func read(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 iter := client.Single().Read(ctx, "Albums", spanner.AllKeys(),
     []string{"SingerId", "AlbumId", "AlbumTitle"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

Run the sample using the `  read  ` argument.

``` text
go run snippet.go read projects/PROJECT_ID/instances/test-instance/databases/example-db
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

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner client library for Go.

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

#### Use the Spanner client library for Go

Use [`  DatabaseAdminClient.UpdateDatabaseDdl()  `](https://pkg.go.dev/cloud.google.com/go/spanner//admin/database/apiv1#DatabaseAdminClient.UpdateDatabaseDdl) to modify the schema:

### GoogleSQL

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addNewColumn(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Albums ADD COLUMN MarketingBudget INT64",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added MarketingBudget column\n")
 return nil
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func pgAddNewColumn(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "ALTER TABLE Albums ADD COLUMN MarketingBudget bigint",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added MarketingBudget column\n")
 return nil
}
```

Run the sample using the `  addnewcolumn  ` argument for Google SQL and the `  pgaddnewcolumn  ` argument for PostgreSQL.

### GoogleSQL

``` text
go run snippet.go addnewcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgaddnewcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see:

``` text
Added MarketingBudget column.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

``` go
import (
 "context"
 "io"

 "cloud.google.com/go/spanner"
)

func update(w io.Writer, db string) error {
 ctx := context.Background()

 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
 _, err = client.Apply(ctx, []*spanner.Mutation{
     spanner.Update("Albums", cols, []interface{}{1, 1, 100000}),
     spanner.Update("Albums", cols, []interface{}{2, 2, 500000}),
 })
 return err
}
```

Run the sample using the `  update  ` argument.

``` text
go run snippet.go update projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You can also execute a SQL query or a read call to fetch the values that you just wrote.

Here's the code to execute the query:

### GoogleSQL

``` go
import (
 "context"
 "fmt"
 "io"
 "strconv"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func queryNewColumn(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, MarketingBudget FROM Albums`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var marketingBudget spanner.NullInt64
     if err := row.ColumnByName("SingerId", &singerID); err != nil {
         return err
     }
     if err := row.ColumnByName("AlbumId", &albumID); err != nil {
         return err
     }
     if err := row.ColumnByName("MarketingBudget", &marketingBudget); err != nil {
         return err
     }
     budget := "NULL"
     if marketingBudget.Valid {
         budget = strconv.FormatInt(marketingBudget.Int64, 10)
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, budget)
 }
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"
 "strconv"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func pgQueryNewColumn(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, MarketingBudget FROM Albums`}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID, albumID int64
     var marketingBudget spanner.NullInt64
     if err := row.ColumnByName("singerid", &singerID); err != nil {
         return err
     }
     if err := row.ColumnByName("albumid", &albumID); err != nil {
         return err
     }
     if err := row.ColumnByName("marketingbudget", &marketingBudget); err != nil {
         return err
     }
     budget := "NULL"
     if marketingBudget.Valid {
         budget = strconv.FormatInt(marketingBudget.Int64, 10)
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, budget)
 }
}
```

To execute this query, run the sample using the `  querynewcolumn  ` argument for Google SQL and the `  pgquerynewcolumn  ` argument for PostgreSQL.

### GoogleSQL

``` text
go run snippet.go querynewcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgquerynewcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see:

``` text
1 1 100000
1 2 NULL
2 1 NULL
2 2 500000
2 3 NULL
```

## Update data

You can update data using DML in a read-write transaction.

You use the `  Update()  ` method to execute a DML statement.

### GoogleSQL

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func writeWithTransactionUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     // getBudget returns the budget for a record with a given albumId and singerId.
     getBudget := func(albumID, singerID int64) (int64, error) {
         key := spanner.Key{albumID, singerID}
         row, err := txn.ReadRow(ctx, "Albums", key, []string{"MarketingBudget"})
         if err != nil {
             return 0, err
         }
         var budget int64
         if err := row.Column(0, &budget); err != nil {
             return 0, err
         }
         return budget, nil
     }
     // updateBudget updates the budget for a record with a given albumId and singerId.
     updateBudget := func(singerID, albumID, albumBudget int64) error {
         stmt := spanner.Statement{
             SQL: `UPDATE Albums
                 SET MarketingBudget = @AlbumBudget
                 WHERE SingerId = @SingerId and AlbumId = @AlbumId`,
             Params: map[string]interface{}{
                 "SingerId":    singerID,
                 "AlbumId":     albumID,
                 "AlbumBudget": albumBudget,
             },
         }
         _, err := txn.Update(ctx, stmt)
         return err
     }

     // Transfer the marketing budget from one album to another. By keeping the actions
     // in a single transaction, it ensures the movement is atomic.
     const transferAmt = 200000
     album2Budget, err := getBudget(2, 2)
     if err != nil {
         return err
     }
     // The transaction will only be committed if this condition still holds at the time
     // of commit. Otherwise it will be aborted and the callable will be rerun by the
     // client library.
     if album2Budget >= transferAmt {
         album1Budget, err := getBudget(1, 1)
         if err != nil {
             return err
         }
         if err = updateBudget(1, 1, album1Budget+transferAmt); err != nil {
             return err
         }
         if err = updateBudget(2, 2, album2Budget-transferAmt); err != nil {
             return err
         }
         fmt.Fprintf(w, "Moved %d from Album2's MarketingBudget to Album1's.", transferAmt)
     }
     return nil
 })
 return err
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
)

func pgWriteWithTransactionUsingDML(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 _, err = client.ReadWriteTransaction(ctx, func(ctx context.Context, txn *spanner.ReadWriteTransaction) error {
     // getBudget returns the budget for a record with a given albumId and singerId.
     getBudget := func(albumID, singerID int64) (int64, error) {
         key := spanner.Key{albumID, singerID}
         row, err := txn.ReadRow(ctx, "Albums", key, []string{"MarketingBudget"})
         if err != nil {
             return 0, fmt.Errorf("error reading marketing budget for album_id=%v,singer_id=%v: %w",
                 albumID, singerID, err)
         }
         var budget int64
         if err := row.Column(0, &budget); err != nil {
             return 0, fmt.Errorf("error decoding marketing budget for album_id=%v,singer_id=%v: %w",
                 albumID, singerID, err)
         }
         return budget, nil
     }
     // updateBudget updates the budget for a record with a given albumId and singerId.
     updateBudget := func(singerID, albumID, albumBudget int64) error {
         stmt := spanner.Statement{
             SQL: `UPDATE Albums
                 SET MarketingBudget = $1
                 WHERE SingerId = $2 and AlbumId = $3`,
             Params: map[string]interface{}{
                 "p1": albumBudget,
                 "p2": singerID,
                 "p3": albumID,
             },
         }
         _, err := txn.Update(ctx, stmt)
         return err
     }

     // Transfer the marketing budget from one album to another. By keeping the actions
     // in a single transaction, it ensures the movement is atomic.
     const transferAmt = 200000
     album2Budget, err := getBudget(2, 2)
     if err != nil {
         return err
     }
     // The transaction will only be committed if this condition still holds at the time
     // of commit. Otherwise it will be aborted and the callable will be rerun by the
     // client library.
     if album2Budget >= transferAmt {
         album1Budget, err := getBudget(1, 1)
         if err != nil {
             return err
         }
         if err = updateBudget(1, 1, album1Budget+transferAmt); err != nil {
             return err
         }
         if err = updateBudget(2, 2, album2Budget-transferAmt); err != nil {
             return err
         }
         fmt.Fprintf(w, "Moved %d from Album2's MarketingBudget to Album1's.", transferAmt)
     }
     return nil
 })
 return err
}
```

Run the sample using the `  dmlwritetxn  ` argument.

``` text
go run snippet.go dmlwritetxn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see:

``` text
Moved 200000 from Album2's MarketingBudget to Album1's.
```

**Note:** You can also [update data using mutations](/spanner/docs/modify-mutation-api#updating_rows_in_a_table) .

## Use a secondary index

Suppose you wanted to fetch all rows of `  Albums  ` that have `  AlbumTitle  ` values in a certain range. You could read all values from the `  AlbumTitle  ` column using a SQL statement or a read call, and then discard the rows that don't meet the criteria, but doing this full table scan is expensive, especially for tables with a lot of rows. Instead you can speed up the retrieval of rows when searching by non-primary key columns by creating a [secondary index](/spanner/docs/secondary-indexes) on the table.

Adding a secondary index to an existing table requires a schema update. Like other schema updates, Spanner supports adding an index while the database continues to serve traffic. Spanner automatically backfills the index with your existing data. Backfills might take a few minutes to complete, but you don't need to take the database offline or avoid writing to the indexed table during this process. For more details, see [Add a secondary index](/spanner/docs/secondary-indexes#adding_an_index) .

After you add a secondary index, Spanner automatically uses it for SQL queries that are likely to run faster with the index. If you use the read interface, you must specify the index that you want to use.

### Add a secondary index

You can add an index on the command line using the gcloud CLI or programmatically using the Spanner client library for Go.

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

#### Using the Spanner client library for Go

Use [`  UpdateDatabaseDdl()  `](https://pkg.go.dev/cloud.google.com/go/spanner//admin/database/apiv1#DatabaseAdminClient.UpdateDatabaseDdl) to add an index:

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addIndex(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "CREATE INDEX AlbumsByAlbumTitle ON Albums(AlbumTitle)",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added index\n")
 return nil
}
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added index
```

### Read using the index

For SQL queries, Spanner automatically uses an appropriate index. In the read interface, you must specify the index in your request.

To use the index in the read interface, use [`  ReadOnlyTransaction.ReadUsingIndex()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadOnlyTransaction.ReadUsingIndex) , which reads zero or more rows from a database using an index.

The following code fetches all `  AlbumId  ` , and `  AlbumTitle  ` columns from the `  AlbumsByAlbumTitle  ` index.

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readUsingIndex(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 iter := client.Single().ReadUsingIndex(ctx, "Albums", "AlbumsByAlbumTitle", spanner.AllKeys(),
     []string{"AlbumId", "AlbumTitle"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var albumID int64
     var albumTitle string
     if err := row.Columns(&albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %s\n", albumID, albumTitle)
 }
}
```

Run the sample using the `  readindex  ` argument.

``` text
go run snippet.go readindex projects/PROJECT_ID/instances/test-instance/databases/example-db
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

#### Using the Spanner client library for Go

Use [`  UpdateDatabaseDdl()  `](https://pkg.go.dev/cloud.google.com/go/spanner//admin/database/apiv1#DatabaseAdminClient.UpdateDatabaseDdl) to add an index with a `  STORING  ` clause for GoogleSQL and `  INCLUDE  ` clause for PostgreSQL:

### GoogleSQL

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addStoringIndex(ctx context.Context, w io.Writer, db string) error {
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) STORING (MarketingBudget)",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 fmt.Fprintf(w, "Added storing index\n")
 return nil
}
```

### PostgreSQL

``` go
import (
 "context"
 "fmt"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

// pgAddStoringIndex shows how to create 'STORING' indexes on a Spanner
// PostgreSQL database. The PostgreSQL dialect uses INCLUDE keyword, as
// opposed to the STORING keyword of Cloud Spanner.
func pgAddStoringIndex(ctx context.Context, w io.Writer, db string) error {
 // db := "projects/my-project/instances/my-instance/databases/my-database"
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return fmt.Errorf("failed to initialize spanner database admin client: %w", err)
 }
 defer adminClient.Close()

 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "CREATE INDEX AlbumsByAlbumTitle2 ON Albums(AlbumTitle) INCLUDE (MarketingBudget)",
     },
 })
 if err != nil {
     return fmt.Errorf("failed to execute spanner database DDL request: %w", err)
 }
 if err := op.Wait(ctx); err != nil {
     return fmt.Errorf("failed to complete spanner database DDL request: %w", err)
 }
 fmt.Fprintf(w, "Added storing index\n")
 return nil
}
```

Run the sample using the `  addstoringindex  ` argument.

### GoogleSQL

``` text
go run snippet.go addstoringindex projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run snippet.go pgaddstoringindex projects/PROJECT_ID/instances/test-instance/databases/example-db
```

Adding an index can take a few minutes. After the index is added, you should see:

``` text
Added storing index
```

Now you can execute a read that fetches all `  AlbumId  ` , `  AlbumTitle  ` , and `  MarketingBudget  ` columns from the `  AlbumsByAlbumTitle2  ` index:

``` go
import (
 "context"
 "fmt"
 "io"
 "strconv"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readStoringIndex(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 iter := client.Single().ReadUsingIndex(ctx, "Albums", "AlbumsByAlbumTitle2", spanner.AllKeys(),
     []string{"AlbumId", "AlbumTitle", "MarketingBudget"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var albumID int64
     var marketingBudget spanner.NullInt64
     var albumTitle string
     if err := row.Columns(&albumID, &albumTitle, &marketingBudget); err != nil {
         return err
     }
     budget := "NULL"
     if marketingBudget.Valid {
         budget = strconv.FormatInt(marketingBudget.Int64, 10)
     }
     fmt.Fprintf(w, "%d %s %s\n", albumID, albumTitle, budget)
 }
}
```

Run the sample using the `  readstoringindex  ` argument.

``` text
go run snippet.go readstoringindex projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see output similar to:

``` text
2 Forever Hold Your Peace 300000
2 Go, Go, Go NULL
1 Green NULL
3 Terrified NULL
1 Total Junk 300000
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data. Use the [`  ReadOnlyTransaction  `](https://pkg.go.dev/cloud.google.com/go/spanner/#ReadOnlyTransaction) type for executing read-only transactions. Use [`  Client.ReadOnlyTransaction()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Client.ReadOnlyTransaction) to get a `  ReadOnlyTransaction  ` .

The following shows how to run a query and perform a read in the same read-only transaction:

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readOnlyTransaction(w io.Writer, db string) error {
 ctx := context.Background()
 client, err := spanner.NewClient(ctx, db)
 if err != nil {
     return err
 }
 defer client.Close()

 ro := client.ReadOnlyTransaction()
 defer ro.Close()
 stmt := spanner.Statement{SQL: `SELECT SingerId, AlbumId, AlbumTitle FROM Albums`}
 iter := ro.Query(ctx, stmt)
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     var singerID int64
     var albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }

 iter = ro.Read(ctx, "Albums", spanner.AllKeys(), []string{"SingerId", "AlbumId", "AlbumTitle"})
 defer iter.Stop()
 for {
     row, err := iter.Next()
     if err == iterator.Done {
         return nil
     }
     if err != nil {
         return err
     }
     var singerID int64
     var albumID int64
     var albumTitle string
     if err := row.Columns(&singerID, &albumID, &albumTitle); err != nil {
         return err
     }
     fmt.Fprintf(w, "%d %d %s\n", singerID, albumID, albumTitle)
 }
}
```

Run the sample using the `  readonlytransaction  ` argument.

``` text
go run snippet.go readonlytransaction projects/PROJECT_ID/instances/test-instance/databases/example-db
```

You should see output similar to:

``` text
2 2 Forever Hold Your Peace
1 2 Go, Go, Go
2 1 Green
2 3 Terrified
1 1 Total Junk
1 1 Total Junk
1 2 Go, Go, Go
2 1 Green
2 2 Forever Hold Your Peace
2 3 Terrified
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
