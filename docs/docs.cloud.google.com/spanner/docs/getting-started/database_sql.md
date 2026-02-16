## Objectives

This tutorial walks you through the following steps using the Spanner database/sql driver:

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

## Prepare your local database/sql environment

1.  Download and install [Go](https://go.dev/doc/install) on your development machine if it isn't already installed.

2.  Clone the sample repository to your local machine:
    
    ``` text
    git clone https://github.com/googleapis/go-sql-spanner.git
    ```

3.  Change to the directory that contains the Spanner sample code:
    
    ``` text
    cd go-sql-spanner/snippets
    ```

## Create an instance

When you first use Spanner, you must create an instance, which is an allocation of resources that are used by Spanner databases. When you create an instance, you choose an *instance configuration* , which determines where your data is stored, and also the number of nodes to use, which determines the amount of serving and storage resources in your instance.

See [Create an instance](/spanner/docs/create-manage-instances#create-instance) to learn how to create a Spanner instance using any of the following methods. You can name your instance `  test-instance  ` to use it with other topics in this document that reference an instance named `  test-instance  ` .

  - The Google Cloud CLI
  - The Google Cloud console
  - A client library (C++, C\#, Go, Java, Node.js, PHP, Python, or Ruby)

## Look through sample files

The samples repository contains a sample that shows how to use Spanner with database/sql.

Take a look through the `  getting_started_guide.go  ` file, which shows how to use Spanner. The code shows how to create and use a new database. The data uses the example schema shown in the [Schema and data model](/spanner/docs/schema-and-data-model#creating-interleaved-tables) page.

## Create a database

### GoogleSQL

``` text
gcloud spanner databases create example-db --instance=test-instance
```

### PostgreSQL

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

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func CreateTables(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Create two tables in one batch on Spanner.
    conn, err := db.Conn(ctx)
    defer conn.Close()

    // Start a DDL batch on the connection.
    // This instructs the connection to buffer all DDL statements until the
    // command `run batch` is executed.
    if _, err := conn.ExecContext(ctx, "start batch ddl"); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `CREATE TABLE Singers (
                SingerId   INT64 NOT NULL,
                FirstName  STRING(1024),
                LastName   STRING(1024),
                SingerInfo BYTES(MAX)
            ) PRIMARY KEY (SingerId)`); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `CREATE TABLE Albums (
                SingerId     INT64 NOT NULL,
                AlbumId      INT64 NOT NULL,
                AlbumTitle   STRING(MAX)
            ) PRIMARY KEY (SingerId, AlbumId),
            INTERLEAVE IN PARENT Singers ON DELETE CASCADE`); err != nil {
        return err
    }
    // `run batch` sends the DDL statements to Spanner and blocks until
    // all statements have finished executing.
    if _, err := conn.ExecContext(ctx, "run batch"); err != nil {
        return err
    }

    fmt.Fprintf(w, "Created Singers & Albums tables in database: [%s]\n", databaseName)
    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func CreateTablesPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Create two tables in one batch on Spanner PostgreSQL.
    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    defer func() { _ = conn.Close() }()

    // Start a DDL batch on the connection.
    // This instructs the connection to buffer all DDL statements until the
    // command `run batch` is executed.
    if _, err := conn.ExecContext(ctx, "start batch ddl"); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `create table singers (
                singer_id   bigint not null primary key,
                first_name  varchar(1024),
                last_name   varchar(1024),
                singer_info bytea
            )`); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `create table albums (
                singer_id     bigint not null,
                album_id      bigint not null,
                album_title   varchar,
                primary key (singer_id, album_id)
            )
            interleave in parent singers on delete cascade`); err != nil {
        return err
    }
    // `run batch` sends the DDL statements to Spanner and blocks until
    // all statements have finished executing.
    if _, err := conn.ExecContext(ctx, "run batch"); err != nil {
        return err
    }

    _, _ = fmt.Fprintf(w, "Created singers & albums tables in database: [%s]\n", databaseName)
    return nil
}
```

Run the sample with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go createtables projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go createtablespg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The next step is to write data to your database.

## Create a connection

Before you can do reads or writes, you must create a [`  sql.DB  `](https://pkg.go.dev/database/sql#DB) . `  sql.DB  ` contains a connection pool that can be used to interact with Spanner. The database name and other connection properties are specified in the database/sql data source name.

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func CreateConnection(ctx context.Context, w io.Writer, databaseName string) error {
    // The dataSourceName should start with a fully qualified Spanner database name
    // in the format `projects/my-project/instances/my-instance/databases/my-database`.
    // Additional properties can be added after the database name by
    // adding one or more `;name=value` pairs.

    dsn := fmt.Sprintf("%s;numChannels=8", databaseName)
    db, err := sql.Open("spanner", dsn)
    if err != nil {
        return err
    }
    defer db.Close()

    row := db.QueryRowContext(ctx, "select 'Hello world!' as hello")
    var msg string
    if err := row.Scan(&msg); err != nil {
        return err
    }
    fmt.Fprintf(w, "Greeting from Spanner: %s\n", msg)
    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func CreateConnectionPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    // The dataSourceName should start with a fully qualified Spanner database name
    // in the format `projects/my-project/instances/my-instance/databases/my-database`.
    // Additional properties can be added after the database name by
    // adding one or more `;name=value` pairs.

    dsn := fmt.Sprintf("%s;numChannels=8", databaseName)
    db, err := sql.Open("spanner", dsn)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // The Spanner database/sql driver supports both PostgreSQL-style query
    // parameters ($1, $2, ...) and positional query parameters (?, ?, ...).
    row := db.QueryRowContext(ctx, "select $1 as hello", "Hello world!")
    var msg string
    if err := row.Scan(&msg); err != nil {
        return err
    }
    _, _ = fmt.Fprintf(w, "Greeting from Spanner PostgreSQL: %s\n", msg)
    return nil
}
```

## Write data with DML

You can insert data using Data Manipulation Language (DML) in a read-write transaction.

You use the `  ExecContext  ` function to execute a DML statement.

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func WriteDataWithDml(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Add 4 rows in one statement.
    // The database/sql driver supports positional query parameters.
    res, err := db.ExecContext(ctx,
        "INSERT INTO Singers (SingerId, FirstName, LastName) "+
            "VALUES (?, ?, ?), (?, ?, ?), "+
            "       (?, ?, ?), (?, ?, ?)",
        12, "Melissa", "Garcia",
        13, "Russel", "Morales",
        14, "Jacqueline", "Long",
        15, "Dylan", "Shaw")
    if err != nil {
        return err
    }
    c, err := res.RowsAffected()
    if err != nil {
        return err
    }
    fmt.Fprintf(w, "%v records inserted\n", c)

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func WriteDataWithDmlPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Add 4 rows in one statement.
    // The database/sql driver supports positional query parameters.
    res, err := db.ExecContext(ctx,
        "insert into singers (singer_id, first_name, last_name) "+
            "values (?, ?, ?), (?, ?, ?), "+
            "       (?, ?, ?), (?, ?, ?)",
        12, "Melissa", "Garcia",
        13, "Russel", "Morales",
        14, "Jacqueline", "Long",
        15, "Dylan", "Shaw")
    if err != nil {
        return err
    }
    c, err := res.RowsAffected()
    if err != nil {
        return err
    }
    _, _ = fmt.Fprintf(w, "%v records inserted\n", c)

    return nil
}
```

Run the sample with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go dmlwrite projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go dmlwritepg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

``` text
4 records inserted.
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Write data with mutations

You can also insert data using [mutations](/spanner/docs/modify-mutation-api) .

A [`  Mutation  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Mutation) is a container for mutation operations. A `  Mutation  ` represents a sequence of inserts, updates, and deletes that Spanner applies atomically to different rows and tables in a Spanner database.

Use [`  Mutation.InsertOrUpdate()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#InsertOrUpdate) to construct an `  INSERT_OR_UPDATE  ` mutation, which adds a new row or updates column values if the row already exists. Alternatively, use the [`  Mutation.Insert()  `](https://pkg.go.dev/cloud.google.com/go/spanner/#Insert) method to construct an `  INSERT  ` mutation, which adds a new row.

Use the `  conn.Raw  ` function to get a reference to the underlying Spanner connection. The `  SpannerConn.Apply  ` function applies mutations atomically to the database.

The following code shows how to write the data using mutations:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func WriteDataWithMutations(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Get a connection so that we can get access to the Spanner specific
    // connection interface SpannerConn.
    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    defer conn.Close()

    singerColumns := []string{"SingerId", "FirstName", "LastName"}
    albumColumns := []string{"SingerId", "AlbumId", "AlbumTitle"}
    mutations := []*spanner.Mutation{
        spanner.Insert("Singers", singerColumns, []interface{}{int64(1), "Marc", "Richards"}),
        spanner.Insert("Singers", singerColumns, []interface{}{int64(2), "Catalina", "Smith"}),
        spanner.Insert("Singers", singerColumns, []interface{}{int64(3), "Alice", "Trentor"}),
        spanner.Insert("Singers", singerColumns, []interface{}{int64(4), "Lea", "Martin"}),
        spanner.Insert("Singers", singerColumns, []interface{}{int64(5), "David", "Lomond"}),
        spanner.Insert("Albums", albumColumns, []interface{}{int64(1), int64(1), "Total Junk"}),
        spanner.Insert("Albums", albumColumns, []interface{}{int64(1), int64(2), "Go, Go, Go"}),
        spanner.Insert("Albums", albumColumns, []interface{}{int64(2), int64(1), "Green"}),
        spanner.Insert("Albums", albumColumns, []interface{}{int64(2), int64(2), "Forever Hold Your Peace"}),
        spanner.Insert("Albums", albumColumns, []interface{}{int64(2), int64(3), "Terrified"}),
    }
    // Mutations can be written outside an explicit transaction using SpannerConn#Apply.
    if err := conn.Raw(func(driverConn interface{}) error {
        spannerConn, ok := driverConn.(spannerdriver.SpannerConn)
        if !ok {
            return fmt.Errorf("unexpected driver connection %v, expected SpannerConn", driverConn)
        }
        _, err = spannerConn.Apply(ctx, mutations)
        return err
    }); err != nil {
        return err
    }
    fmt.Fprintf(w, "Inserted %v rows\n", len(mutations))

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func WriteDataWithMutationsPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Get a connection so that we can get access to the Spanner specific
    // connection interface SpannerConn.
    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    defer func() { _ = conn.Close() }()

    singerColumns := []string{"singer_id", "first_name", "last_name"}
    albumColumns := []string{"singer_id", "album_id", "album_title"}
    mutations := []*spanner.Mutation{
        spanner.Insert("singers", singerColumns, []interface{}{int64(1), "Marc", "Richards"}),
        spanner.Insert("singers", singerColumns, []interface{}{int64(2), "Catalina", "Smith"}),
        spanner.Insert("singers", singerColumns, []interface{}{int64(3), "Alice", "Trentor"}),
        spanner.Insert("singers", singerColumns, []interface{}{int64(4), "Lea", "Martin"}),
        spanner.Insert("singers", singerColumns, []interface{}{int64(5), "David", "Lomond"}),
        spanner.Insert("albums", albumColumns, []interface{}{int64(1), int64(1), "Total Junk"}),
        spanner.Insert("albums", albumColumns, []interface{}{int64(1), int64(2), "Go, Go, Go"}),
        spanner.Insert("albums", albumColumns, []interface{}{int64(2), int64(1), "Green"}),
        spanner.Insert("albums", albumColumns, []interface{}{int64(2), int64(2), "Forever Hold Your Peace"}),
        spanner.Insert("albums", albumColumns, []interface{}{int64(2), int64(3), "Terrified"}),
    }
    // Mutations can be written outside an explicit transaction using SpannerConn#Apply.
    if err := conn.Raw(func(driverConn interface{}) error {
        spannerConn, ok := driverConn.(spannerdriver.SpannerConn)
        if !ok {
            return fmt.Errorf("unexpected driver connection %v, expected SpannerConn", driverConn)
        }
        _, err = spannerConn.Apply(ctx, mutations)
        return err
    }); err != nil {
        return err
    }
    _, _ = fmt.Fprintf(w, "Inserted %v rows\n", len(mutations))

    return nil
}
```

Run the following example using the `  write  ` argument:

### GoogleSQL

``` text
go run getting_started_guide.go write projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go writepg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

**Note:** There are limits to commit size. See [CRUD limit](/spanner/quotas#limits-for) for more information.

## Query data using SQL

Spanner supports a SQL interface for reading data, which you can access on the command line using the Google Cloud CLI or programmatically using the Spanner database/sql driver.

### On the command line

Execute the following SQL statement to read the values of all columns from the `  Albums  ` table:

### GoogleSQL

``` text
gcloud spanner databases execute-sql example-db --instance=test-instance \
    --sql='SELECT SingerId, AlbumId, AlbumTitle FROM Albums'
```

### PostgreSQL

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

### Use the Spanner database/sql driver

In addition to executing a SQL statement on the command line, you can issue the same SQL statement programmatically using the Spanner database/sql driver.

The following functions and structs are used to execute a SQL query:

  - The [`  QueryContext  `](https://pkg.go.dev/database/sql#DB.QueryContext) function in the `  DB  ` struct: use this to execute a SQL statement that returns rows, such as a query or a DML statement with a `  THEN RETURN  ` clause.
  - The [`  Rows  `](https://pkg.go.dev/database/sql#Rows) struct: use this to access the data returned by a SQL statement.

The following example uses the `  QueryContext  ` function:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryData(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    rows, err := db.QueryContext(ctx,
        `SELECT SingerId, AlbumId, AlbumTitle
        FROM Albums
        ORDER BY SingerId, AlbumId`)
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
        fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryDataPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    rows, err := db.QueryContext(ctx,
        `select singer_id, album_id, album_title
        from albums
        order by singer_id, album_id`)
    defer func() { _ = rows.Close() }()
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
        _, _ = fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go query projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go querypg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

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

The Spanner database/sql driver supports both positional and named query parameters. A `  ?  ` in a SQL statement indicates a positional query parameter. Pass the query parameter values as additional arguments to the `  QueryContext  ` function. For example:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryDataWithParameter(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    rows, err := db.QueryContext(ctx,
        `SELECT SingerId, FirstName, LastName
        FROM Singers
        WHERE LastName = ?`, "Garcia")
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
        fmt.Fprintf(w, "%v %v %v\n", singerId, firstName, lastName)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryDataWithParameterPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    rows, err := db.QueryContext(ctx,
        `select singer_id, first_name, last_name
        from singers
        where last_name = ?`, "Garcia")
    defer func() { _ = rows.Close() }()
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
        _, _ = fmt.Fprintf(w, "%v %v %v\n", singerId, firstName, lastName)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go querywithparameter projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go querywithparameterpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

``` text
12 Melissa Garcia
```

## Update the database schema

Assume you need to add a new column called `  MarketingBudget  ` to the `  Albums  ` table. Adding a new column to an existing table requires an update to your database schema. Spanner supports schema updates to a database while the database continues to serve traffic. Schema updates don't require taking the database offline and they don't lock entire tables or columns; you can continue writing data to the database during the schema update. Read more about supported schema updates and schema change performance in [Make schema updates](/spanner/docs/schema-updates) .

### Add a column

You can add a column on the command line using the Google Cloud CLI or programmatically using the Spanner database/sql driver.

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
    --ddl='alter table albums add column marketing_budget bigint'
```

You should see:

``` text
Schema updating...done.
```

#### Use the Spanner database/sql driver

Use the [`  ExecContext  `](https://pkg.go.dev/database/sql#DB.ExecContext) function to modify the schema:

### GoogleSQL

``` go
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
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func AddColumnPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    _, err = db.ExecContext(ctx,
        `alter table albums
            add column marketing_budget bigint`)
    if err != nil {
        return err
    }

    _, _ = fmt.Fprint(w, "Added marketing_budget column\n")
    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go addcolumn projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go addcolumnpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

``` text
Added MarketingBudget column.
```

### Execute a DDL batch

We recommend that you execute multiple schema modifications in one batch. Use the `  START BATCH DDL  ` and `  RUN BATCH  ` commands to execute a DDL batch. The following example creates two tables in one batch:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func DdlBatch(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Executing multiple DDL statements as one batch is
    // more efficient than executing each statement
    // individually.
    conn, err := db.Conn(ctx)
    defer conn.Close()

    if _, err := conn.ExecContext(ctx, "start batch ddl"); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `CREATE TABLE Venues (
            VenueId     INT64 NOT NULL,
            Name        STRING(1024),
            Description JSON,
        ) PRIMARY KEY (VenueId)`); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `CREATE TABLE Concerts (
            ConcertId INT64 NOT NULL,
            VenueId   INT64 NOT NULL,
            SingerId  INT64 NOT NULL,
            StartTime TIMESTAMP,
            EndTime   TIMESTAMP,
            CONSTRAINT Fk_Concerts_Venues FOREIGN KEY
                (VenueId) REFERENCES Venues (VenueId),
            CONSTRAINT Fk_Concerts_Singers FOREIGN KEY
                (SingerId) REFERENCES Singers (SingerId),
        ) PRIMARY KEY (ConcertId)`); err != nil {
        return err
    }
    // `run batch` sends the DDL statements to Spanner and blocks until
    // all statements have finished executing.
    if _, err := conn.ExecContext(ctx, "run batch"); err != nil {
        return err
    }

    fmt.Fprint(w, "Added Venues and Concerts tables\n")
    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func DdlBatchPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Executing multiple DDL statements as one batch is
    // more efficient than executing each statement
    // individually.
    conn, err := db.Conn(ctx)
    defer func() { _ = conn.Close() }()

    if _, err := conn.ExecContext(ctx, "start batch ddl"); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `create table venues (
            venue_id    bigint not null primary key,
            name        varchar(1024),
            description jsonb
        )`); err != nil {
        return err
    }
    if _, err := conn.ExecContext(ctx,
        `create table concerts (
            concert_id bigint not null primary key,
            venue_id   bigint not null,
            singer_id  bigint not null,
            start_time timestamptz,
            end_time   timestamptz,
            constraint fk_concerts_venues foreign key
                (venue_id) references venues (venue_id),
            constraint fk_concerts_singers foreign key
                (singer_id) references singers (singer_id)
        )`); err != nil {
        return err
    }
    // `run batch` sends the DDL statements to Spanner and blocks until
    // all statements have finished executing.
    if _, err := conn.ExecContext(ctx, "run batch"); err != nil {
        return err
    }

    _, _ = fmt.Fprint(w, "Added venues and concerts tables\n")
    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go ddlbatch projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go ddlbatchpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

``` text
Added Venues and Concerts tables.
```

### Write data to the new column

The following code writes data to the new column. It sets `  MarketingBudget  ` to `  100000  ` for the row keyed by `  Albums(1, 1)  ` and to `  500000  ` for the row keyed by `  Albums(2, 2)  ` .

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func UpdateDataWithMutations(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Get a connection so that we can get access to the Spanner specific
    // connection interface SpannerConn.
    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    defer conn.Close()

    cols := []string{"SingerId", "AlbumId", "MarketingBudget"}
    mutations := []*spanner.Mutation{
        spanner.Update("Albums", cols, []interface{}{1, 1, 100000}),
        spanner.Update("Albums", cols, []interface{}{2, 2, 500000}),
    }
    if err := conn.Raw(func(driverConn interface{}) error {
        spannerConn, ok := driverConn.(spannerdriver.SpannerConn)
        if !ok {
            return fmt.Errorf("unexpected driver connection %v, "+
                "expected SpannerConn", driverConn)
        }
        _, err = spannerConn.Apply(ctx, mutations)
        return err
    }); err != nil {
        return err
    }
    fmt.Fprintf(w, "Updated %v albums\n", len(mutations))

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func UpdateDataWithMutationsPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Get a connection so that we can get access to the Spanner specific
    // connection interface SpannerConn.
    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    defer func() { _ = conn.Close() }()

    cols := []string{"singer_id", "album_id", "marketing_budget"}
    mutations := []*spanner.Mutation{
        spanner.Update("albums", cols, []interface{}{1, 1, 100000}),
        spanner.Update("albums", cols, []interface{}{2, 2, 500000}),
    }
    if err := conn.Raw(func(driverConn interface{}) error {
        spannerConn, ok := driverConn.(spannerdriver.SpannerConn)
        if !ok {
            return fmt.Errorf("unexpected driver connection %v, "+
                "expected SpannerConn", driverConn)
        }
        _, err = spannerConn.Apply(ctx, mutations)
        return err
    }); err != nil {
        return err
    }
    _, _ = fmt.Fprintf(w, "Updated %v albums\n", len(mutations))

    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go update projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go updatepg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

``` text
Updated 2 albums
```

You can also execute a SQL query to fetch the values that you just wrote.

The following example uses the `  QueryContext  ` function to execute a query:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryNewColumn(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    rows, err := db.QueryContext(ctx,
        `SELECT SingerId, AlbumId, MarketingBudget
        FROM Albums
        ORDER BY SingerId, AlbumId`)
    defer rows.Close()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId, albumId int64
        var marketingBudget sql.NullInt64
        err = rows.Scan(&singerId, &albumId, &marketingBudget)
        if err != nil {
            return err
        }
        budget := "NULL"
        if marketingBudget.Valid {
            budget = fmt.Sprintf("%v", marketingBudget.Int64)
        }
        fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, budget)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func QueryNewColumnPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    rows, err := db.QueryContext(ctx,
        `select singer_id, album_id, marketing_budget
        from albums
        order by singer_id, album_id`)
    defer func() { _ = rows.Close() }()
    if err != nil {
        return err
    }
    for rows.Next() {
        var singerId, albumId int64
        var marketingBudget sql.NullInt64
        err = rows.Scan(&singerId, &albumId, &marketingBudget)
        if err != nil {
            return err
        }
        budget := "null"
        if marketingBudget.Valid {
            budget = fmt.Sprintf("%v", marketingBudget.Int64)
        }
        _, _ = fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, budget)
    }
    if rows.Err() != nil {
        return rows.Err()
    }
    return rows.Close()
}
```

To execute this query, run the following command:

### GoogleSQL

``` text
go run getting_started_guide.go querymarketingbudget projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go querymarketingbudgetpg projects/PROJECT_ID/instances/test-instance/databases/example-db
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

Call [`  DB.BeginTx  `](https://pkg.go.dev/database/sql#DB.BeginTx) to execute read-write transactions in database/sql.

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func WriteWithTransactionUsingDml(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Transfer marketing budget from one album to another. We do it in a
    // transaction to ensure that the transfer is atomic.
    tx, err := db.BeginTx(ctx, &sql.TxOptions{})
    if err != nil {
        return err
    }
    // The Spanner database/sql driver supports both positional and named
    // query parameters. This query uses named query parameters.
    const selectSql = "SELECT MarketingBudget " +
        "FROM Albums " +
        "WHERE SingerId = @singerId and AlbumId = @albumId"
    // Get the marketing_budget of singer 2 / album 2.
    row := tx.QueryRowContext(ctx, selectSql,
        sql.Named("singerId", 2), sql.Named("albumId", 2))
    var budget2 int64
    if err := row.Scan(&budget2); err != nil {
        tx.Rollback()
        return err
    }
    const transfer = 20000
    // The transaction will only be committed if this condition still holds
    // at the time of commit. Otherwise, the transaction will be aborted.
    if budget2 >= transfer {
        // Get the marketing_budget of singer 1 / album 1.
        row := tx.QueryRowContext(ctx, selectSql,
            sql.Named("singerId", 1), sql.Named("albumId", 1))
        var budget1 int64
        if err := row.Scan(&budget1); err != nil {
            tx.Rollback()
            return err
        }
        // Transfer part of the marketing budget of Album 2 to Album 1.
        budget1 += transfer
        budget2 -= transfer
        const updateSql = "UPDATE Albums " +
            "SET MarketingBudget = @budget " +
            "WHERE SingerId = @singerId and AlbumId = @albumId"
        // Start a DML batch and execute it as part of the current transaction.
        if _, err := tx.ExecContext(ctx, "start batch dml"); err != nil {
            tx.Rollback()
            return err
        }
        if _, err := tx.ExecContext(ctx, updateSql,
            sql.Named("singerId", 1),
            sql.Named("albumId", 1),
            sql.Named("budget", budget1)); err != nil {
            _, _ = tx.ExecContext(ctx, "abort batch")
            tx.Rollback()
            return err
        }
        if _, err := tx.ExecContext(ctx, updateSql,
            sql.Named("singerId", 2),
            sql.Named("albumId", 2),
            sql.Named("budget", budget2)); err != nil {
            _, _ = tx.ExecContext(ctx, "abort batch")
            tx.Rollback()
            return err
        }
        // `run batch` sends the DML statements to Spanner.
        // The result contains the total affected rows across the entire batch.
        result, err := tx.ExecContext(ctx, "run batch")
        if err != nil {
            tx.Rollback()
            return err
        }
        if affected, err := result.RowsAffected(); err != nil {
            tx.Rollback()
            return err
        } else if affected != 2 {
            // The batch should update 2 rows.
            tx.Rollback()
            return fmt.Errorf("unexpected number of rows affected: %v", affected)
        }
    }
    // Commit the current transaction.
    if err := tx.Commit(); err != nil {
        return err
    }

    fmt.Fprintln(w, "Transferred marketing budget from Album 2 to Album 1")

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func WriteWithTransactionUsingDmlPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Transfer marketing budget from one album to another. We do it in a
    // transaction to ensure that the transfer is atomic.
    tx, err := db.BeginTx(ctx, &sql.TxOptions{})
    if err != nil {
        return err
    }
    const selectSql = "select marketing_budget " +
        "from albums " +
        "where singer_id = $1 and album_id = $2"
    // Get the marketing_budget of singer 2 / album 2.
    row := tx.QueryRowContext(ctx, selectSql, 2, 2)
    var budget2 int64
    if err := row.Scan(&budget2); err != nil {
        _ = tx.Rollback()
        return err
    }
    const transfer = 20000
    // The transaction will only be committed if this condition still holds
    // at the time of commit. Otherwise, the transaction will be aborted.
    if budget2 >= transfer {
        // Get the marketing_budget of singer 1 / album 1.
        row := tx.QueryRowContext(ctx, selectSql, 1, 1)
        var budget1 int64
        if err := row.Scan(&budget1); err != nil {
            _ = tx.Rollback()
            return err
        }
        // Transfer part of the marketing budget of Album 2 to Album 1.
        budget1 += transfer
        budget2 -= transfer
        const updateSql = "update albums " +
            "set marketing_budget = $1 " +
            "where singer_id = $2 and album_id = $3"
        // Start a DML batch and execute it as part of the current transaction.
        if _, err := tx.ExecContext(ctx, "start batch dml"); err != nil {
            _ = tx.Rollback()
            return err
        }
        if _, err := tx.ExecContext(ctx, updateSql, budget1, 1, 1); err != nil {
            _, _ = tx.ExecContext(ctx, "abort batch")
            _ = tx.Rollback()
            return err
        }
        if _, err := tx.ExecContext(ctx, updateSql, budget2, 2, 2); err != nil {
            _, _ = tx.ExecContext(ctx, "abort batch")
            _ = tx.Rollback()
            return err
        }
        // `run batch` sends the DML statements to Spanner.
        // The result contains the total affected rows across the entire batch.
        result, err := tx.ExecContext(ctx, "run batch")
        if err != nil {
            _ = tx.Rollback()
            return err
        }
        if affected, err := result.RowsAffected(); err != nil {
            _ = tx.Rollback()
            return err
        } else if affected != 2 {
            // The batch should update 2 rows.
            _ = tx.Rollback()
            return fmt.Errorf("unexpected number of rows affected: %v", affected)
        }
    }
    // Commit the current transaction.
    if err := tx.Commit(); err != nil {
        return err
    }

    _, _ = fmt.Fprintln(w, "Transferred marketing budget from Album 2 to Album 1")

    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go writewithtransactionusingdml projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go writewithtransactionusingdmlpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### Transaction tags and request tags

Use [transaction tags and request tags](/spanner/docs/introspection/troubleshooting-with-tags) to troubleshoot transactions and queries in Spanner. You can pass additional transaction options to the `  spannerdriver.BeginReadWriteTransaction  ` function.

Use `  spannerdriver.ExecOptions  ` to pass additional query options for a SQL statement. For example:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func Tags(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Use the spannerdriver.BeginReadWriteTransaction function
    // to specify specific Spanner options, such as transaction tags.
    tx, err := spannerdriver.BeginReadWriteTransaction(ctx, db,
        spannerdriver.ReadWriteTransactionOptions{
            TransactionOptions: spanner.TransactionOptions{
                TransactionTag: "example-tx-tag",
            },
        })
    if err != nil {
        return err
    }

    // Pass in an argument of type spannerdriver.ExecOptions to supply
    // additional options for a statement.
    row := tx.QueryRowContext(ctx, "SELECT MarketingBudget "+
        "FROM Albums "+
        "WHERE SingerId=? and AlbumId=?",
        spannerdriver.ExecOptions{
            QueryOptions: spanner.QueryOptions{RequestTag: "query-marketing-budget"},
        }, 1, 1)
    var budget int64
    if err := row.Scan(&budget); err != nil {
        tx.Rollback()
        return err
    }

    // Reduce the marketing budget by 10% if it is more than 1,000.
    if budget > 1000 {
        budget = int64(float64(budget) - float64(budget)*0.1)
        if _, err := tx.ExecContext(ctx,
            `UPDATE Albums SET MarketingBudget=@budget 
               WHERE SingerId=@singerId AND AlbumId=@albumId`,
            spannerdriver.ExecOptions{
                QueryOptions: spanner.QueryOptions{RequestTag: "reduce-marketing-budget"},
            },
            sql.Named("budget", budget),
            sql.Named("singerId", 1),
            sql.Named("albumId", 1)); err != nil {
            tx.Rollback()
            return err
        }
    }
    // Commit the current transaction.
    if err := tx.Commit(); err != nil {
        return err
    }
    fmt.Fprintln(w, "Reduced marketing budget")

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func TagsPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Use the spannerdriver.BeginReadWriteTransaction function
    // to specify specific Spanner options, such as transaction tags.
    tx, err := spannerdriver.BeginReadWriteTransaction(ctx, db,
        spannerdriver.ReadWriteTransactionOptions{
            TransactionOptions: spanner.TransactionOptions{
                TransactionTag: "example-tx-tag",
            },
        })
    if err != nil {
        return err
    }

    // Pass in an argument of type spannerdriver.ExecOptions to supply
    // additional options for a statement.
    row := tx.QueryRowContext(ctx, "select marketing_budget "+
        "from albums "+
        "where singer_id=? and album_id=?",
        spannerdriver.ExecOptions{
            QueryOptions: spanner.QueryOptions{RequestTag: "query-marketing-budget"},
        }, 1, 1)
    var budget int64
    if err := row.Scan(&budget); err != nil {
        _ = tx.Rollback()
        return err
    }

    // Reduce the marketing budget by 10% if it is more than 1,000.
    if budget > 1000 {
        budget = int64(float64(budget) - float64(budget)*0.1)
        if _, err := tx.ExecContext(ctx,
            `update albums set marketing_budget=$1 
               where singer_id=$2 and album_id=$3`,
            spannerdriver.ExecOptions{
                QueryOptions: spanner.QueryOptions{RequestTag: "reduce-marketing-budget"},
            }, budget, 1, 1); err != nil {
            _ = tx.Rollback()
            return err
        }
    }
    // Commit the current transaction.
    if err := tx.Commit(); err != nil {
        return err
    }
    _, _ = fmt.Fprintln(w, "Reduced marketing budget")

    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go tags projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go tagspg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

## Retrieve data using read-only transactions

Suppose you want to execute more than one read at the same timestamp. [Read-only transactions](/spanner/docs/transactions#read-only_transactions) observe a consistent prefix of the transaction commit history, so your application always gets consistent data.Set the [`  TxOptions.ReadOnly  `](https://pkg.go.dev/database/sql#TxOptions) field to `  true  ` to execute a read-only transaction.

The following shows how to run a query and perform a read in the same read-only transaction:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func ReadOnlyTransaction(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Start a read-only transaction by supplying additional transaction options.
    tx, err := db.BeginTx(ctx, &sql.TxOptions{ReadOnly: true})

    albumsOrderedById, err := tx.QueryContext(ctx,
        `SELECT SingerId, AlbumId, AlbumTitle
        FROM Albums
        ORDER BY SingerId, AlbumId`)
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
        fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }

    albumsOrderedTitle, err := tx.QueryContext(ctx,
        `SELECT SingerId, AlbumId, AlbumTitle
        FROM Albums
        ORDER BY AlbumTitle`)
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
        fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }

    // End the read-only transaction by calling Commit.
    return tx.Commit()
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func ReadOnlyTransactionPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Start a read-only transaction by supplying additional transaction options.
    tx, err := db.BeginTx(ctx, &sql.TxOptions{ReadOnly: true})
    if err != nil {
        return err
    }

    albumsOrderedById, err := tx.QueryContext(ctx,
        `select singer_id, album_id, album_title
        from albums
        order by singer_id, album_id`)
    defer func() { _ = albumsOrderedById.Close() }()
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
        _, _ = fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }

    albumsOrderedTitle, err := tx.QueryContext(ctx,
        `select singer_id, album_id, album_title
        from albums
        order by album_title`)
    defer func() { _ = albumsOrderedTitle.Close() }()
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
        _, _ = fmt.Fprintf(w, "%v %v %v\n", singerId, albumId, title)
    }

    // End the read-only transaction by calling Commit.
    return tx.Commit()
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go readonlytransaction projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go readonlytransactionpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

The result shows:

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

The [`  partitionQuery  `](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.PartitionQuery) API divides a query into smaller pieces, or partitions, and uses multiple machines to fetch the partitions in parallel. Each partition is identified by a partition token. The partitionQuery API has higher latency than the standard [query API](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.ExecuteStreamingSql) , because it's only intended for bulk operations such as exporting or scanning the whole database.

[Data Boost](/spanner/docs/databoost/databoost-overview) lets you execute analytics queries and data exports with near-zero impact to existing workloads on the provisioned Spanner instance. Data Boost only supports [partitioned queries](/spanner/docs/reads#read_data_in_parallel) .

The following example shows how to execute a partitioned query with Data Boost with the database/sql driver:

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"
    "slices"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func DataBoost(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    // Run a partitioned query that uses Data Boost.
    rows, err := db.QueryContext(ctx,
        "SELECT SingerId, FirstName, LastName from Singers",
        spannerdriver.ExecOptions{
            PartitionedQueryOptions: spannerdriver.PartitionedQueryOptions{
                // AutoPartitionQuery instructs the Spanner database/sql driver to
                // automatically partition the query and execute each partition in parallel.
                // The rows are returned as one result set in undefined order.
                AutoPartitionQuery: true,
            },
            QueryOptions: spanner.QueryOptions{
                // Set DataBoostEnabled to true to enable DataBoost.
                // See https://cloud.google.com/spanner/docs/databoost/databoost-overview
                // for more information.
                DataBoostEnabled: true,
            },
        })
    defer rows.Close()
    if err != nil {
        return err
    }
    type Singer struct {
        SingerId  int64
        FirstName string
        LastName  string
    }
    var singers []Singer
    for rows.Next() {
        var singer Singer
        err = rows.Scan(&singer.SingerId, &singer.FirstName, &singer.LastName)
        if err != nil {
            return err
        }
        singers = append(singers, singer)
    }
    // Queries that use the AutoPartition option return rows in undefined order,
    // so we need to sort them in memory to guarantee the output order.
    slices.SortFunc(singers, func(a, b Singer) int {
        return int(a.SingerId - b.SingerId)
    })
    for _, s := range singers {
        fmt.Fprintf(w, "%v %v %v\n", s.SingerId, s.FirstName, s.LastName)
    }

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"
    "slices"

    "cloud.google.com/go/spanner"
    spannerdriver "github.com/googleapis/go-sql-spanner"
)

func DataBoostPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    // Run a partitioned query that uses Data Boost.
    rows, err := db.QueryContext(ctx,
        "select singer_id, first_name, last_name from singers",
        spannerdriver.ExecOptions{
            PartitionedQueryOptions: spannerdriver.PartitionedQueryOptions{
                // AutoPartitionQuery instructs the Spanner database/sql driver to
                // automatically partition the query and execute each partition in parallel.
                // The rows are returned as one result set in undefined order.
                AutoPartitionQuery: true,
            },
            QueryOptions: spanner.QueryOptions{
                // Set DataBoostEnabled to true to enable DataBoost.
                // See https://cloud.google.com/spanner/docs/databoost/databoost-overview
                // for more information.
                DataBoostEnabled: true,
            },
        })
    defer func() { _ = rows.Close() }()
    if err != nil {
        return err
    }
    type Singer struct {
        SingerId  int64
        FirstName string
        LastName  string
    }
    var singers []Singer
    for rows.Next() {
        var singer Singer
        err = rows.Scan(&singer.SingerId, &singer.FirstName, &singer.LastName)
        if err != nil {
            return err
        }
        singers = append(singers, singer)
    }
    // Queries that use the AutoPartition option return rows in undefined order,
    // so we need to sort them in memory to guarantee the output order.
    slices.SortFunc(singers, func(a, b Singer) int {
        return int(a.SingerId - b.SingerId)
    })
    for _, s := range singers {
        _, _ = fmt.Fprintf(w, "%v %v %v\n", s.SingerId, s.FirstName, s.LastName)
    }

    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go databoost projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go databoostpg projects/PROJECT_ID/instances/test-instance/databases/example-db
```

## Partitioned DML

[Partitioned Data Manipulation Language (DML)](/spanner/docs/dml-partitioned) is designed for the following types of bulk updates and deletes:

  - Periodic cleanup and garbage collection.
  - Backfilling new columns with default values.

### GoogleSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func PartitionedDml(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer db.Close()

    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    // Enable Partitioned DML on this connection.
    if _, err := conn.ExecContext(ctx, "SET AUTOCOMMIT_DML_MODE='PARTITIONED_NON_ATOMIC'"); err != nil {
        return fmt.Errorf("failed to change DML mode to Partitioned_Non_Atomic: %v", err)
    }
    // Back-fill a default value for the MarketingBudget column.
    res, err := conn.ExecContext(ctx, "UPDATE Albums SET MarketingBudget=0 WHERE MarketingBudget IS NULL")
    if err != nil {
        return err
    }
    affected, err := res.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get affected rows: %v", err)
    }

    // Partitioned DML returns the minimum number of records that were affected.
    fmt.Fprintf(w, "Updated at least %v albums\n", affected)

    // Closing the connection will return it to the connection pool. The DML mode will automatically be reset to the
    // default TRANSACTIONAL mode when the connection is returned to the pool, so we do not need to change it back
    // manually.
    _ = conn.Close()

    return nil
}
```

### PostgreSQL

``` go
import (
    "context"
    "database/sql"
    "fmt"
    "io"

    _ "github.com/googleapis/go-sql-spanner"
)

func PartitionedDmlPostgreSQL(ctx context.Context, w io.Writer, databaseName string) error {
    db, err := sql.Open("spanner", databaseName)
    if err != nil {
        return err
    }
    defer func() { _ = db.Close() }()

    conn, err := db.Conn(ctx)
    if err != nil {
        return err
    }
    // Enable Partitioned DML on this connection.
    if _, err := conn.ExecContext(ctx, "set autocommit_dml_mode='partitioned_non_atomic'"); err != nil {
        return fmt.Errorf("failed to change DML mode to Partitioned_Non_Atomic: %v", err)
    }
    // Back-fill a default value for the marketing_budget column.
    res, err := conn.ExecContext(ctx, "update albums set marketing_budget=0 where marketing_budget is null")
    if err != nil {
        return err
    }
    affected, err := res.RowsAffected()
    if err != nil {
        return fmt.Errorf("failed to get affected rows: %v", err)
    }

    // Partitioned DML returns the minimum number of records that were affected.
    _, _ = fmt.Fprintf(w, "Updated at least %v albums\n", affected)

    // Closing the connection will return it to the connection pool. The DML mode will automatically be reset to the
    // default TRANSACTIONAL mode when the connection is returned to the pool, so we do not need to change it back
    // manually.
    _ = conn.Close()

    return nil
}
```

Run the example with the following command:

### GoogleSQL

``` text
go run getting_started_guide.go pdml projects/PROJECT_ID/instances/test-instance/databases/example-db
```

### PostgreSQL

``` text
go run getting_started_guide.go pdmlpg projects/PROJECT_ID/instances/test-instance/databases/example-db
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
