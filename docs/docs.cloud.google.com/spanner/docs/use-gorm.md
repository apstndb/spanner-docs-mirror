GORM is an object-relational mapping tool for the Go programming language. It provides a framework for mapping an object-oriented domain model to a relational database.

You can integrate GoogleSQL-dialect databases with GORM using the open source [Spanner Dialect](https://github.com/googleapis/go-gorm-spanner) ( `  SpannerDialect  ` ).

## Set up GORM with Spanner GoogleSQL-dialect databases

To use the GoogleSQL GORM dialect in your application, add the following import statement to the file where GORM is initialized:

```` python
  import (
    "fmt"

    "gorm.io/gorm"
    _ "github.com/googleapis/go-sql-spanner"
    spannergorm "github.com/googleapis/go-gorm-spanner"
  )

  dsn := fmt.Sprintf("projects/%s/instances/%s/databases/%s", projectId, instanceId, databaseId),
  db, err := gorm.Open(spannergorm.New(spannergorm.Config{DriverName: "spanner", DSN: dsn}), &gorm.Config{})
  ```

See the

[GORM with GoogleSQL documentation][go-driver-documentation]

for more connection options for Spanner.

## Use GORM with Spanner GoogleSQL-dialect databases

For more information about the features and recommendations for using
GORM with Spanner, consult the [reference documentation][spanner-gorm-github]
on GitHub.

## What's next {: #whats-next}

*   Checkout the [sample application][gorm-gsql-sample-application] using
    GORM with GoogleSQL and Spanner.
*   Learn more about [GORM][gorm].
*   [File a GitHub issue][spanner-gorm-issue] to report a bug or ask a question
    about using GORM with Spanner with GoogleSQL.
*   Learn more about [Integrate Spanner with GORM (
    PostgreSQL dialect)](/spanner/docs/use-gorm-postgresql).









  

  
    
  
````
