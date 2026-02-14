This page shows how to get started with the Cloud Client Libraries for the Cloud Spanner API. Client libraries make it easier to access Google Cloud APIs from a supported language. Although you can use Google Cloud APIs directly by making raw requests to the server, client libraries provide simplifications that significantly reduce the amount of code you need to write.

Read more about the Cloud Client Libraries and the older Google API Client Libraries in [Client libraries explained](/apis/docs/client-libraries-explained) .

Spanner provides client libraries for popular programming languages, including C++, C\#, Go, Java, Node.js, PHP, Python, and Ruby. The client libraries connect to the Spanner service using gRPC over a TLS-encrypted HTTP/2 connection, so you don't need to manually configure SSL or TLS.

The Spanner client libraries are supported on Compute Engine, App Engine flexible environment, Google Kubernetes Engine, and Cloud Run functions. The Spanner client library for Java is also supported on App Engine standard environment with Java 8.

If you are using the App Engine standard environment with Go, PHP, or Python, use the [REST interface](/spanner/docs/reference/rest) to access Spanner.

**Note:** [Additional client libraries](#more-libraries) are available for Java applications.

## Install the client library

### C++

For more information about installing the C++ library, see the [Setting up a C++ development environment](/cpp/docs/setup) guide.

### C\#

To install the `  Google.Cloud.Spanner.Data  ` client library in Visual Studio, do the following:

1.  Right-click your solution in Visual Studio and select **Manage Nuget packages for solution** .

2.  Select the **Include prerelease** checkbox.

3.  Search for and install the package named `  Google.Cloud.Spanner.Data  ` .
    
    For more information, see [Setting Up a C\# Development Environment](/dotnet/docs/setup) .

### Go

``` text
go get cloud.google.com/go/spanner/...
```

For more information, see [Setting Up a Go Development Environment](/go/docs/setup) .

### Java

**Note:** If your application uses the Spring Framework, a [Spring Data module](/spanner/docs/adding-spring) is also available.

If you are using [Maven](https://maven.apache.org/) , add the following to your `  pom.xml  ` file. For more information about BOMs, see [The Google Cloud Platform Libraries BOM](https://cloud.google.com/java/docs/bom) .

``` markdown
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.google.cloud</groupId>
      <artifactId>libraries-bom</artifactId>
      <version>26.74.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>com.google.cloud</groupId>
    <artifactId>google-cloud-spanner</artifactId>
  </dependency>
```

If you are using [Gradle](https://gradle.org/) , add the following to your dependencies:

``` markdown
implementation platform('com.google.cloud:libraries-bom:26.75.0')

implementation 'com.google.cloud:google-cloud-spanner'
```

If you are using [sbt](https://www.scala-sbt.org/) , add the following to your dependencies:

``` markdown
libraryDependencies += "com.google.cloud" % "google-cloud-spanner" % "6.109.0"
```

If you're using Visual Studio Code or IntelliJ, you can add client libraries to your project using the following IDE plugins:

  - [Cloud Code for VS Code](/code/docs/vscode/client-libraries)
  - [Cloud Code for IntelliJ](/code/docs/intellij/client-libraries)

The plugins provide additional functionality, such as key management for service accounts. Refer to each plugin's documentation for details.

**Note:** Cloud Java client libraries do not currently support Android.

For more information, see [Setting Up a Java Development Environment](/java/docs/setup) .

### Node.js

``` text
npm install @google-cloud/spanner
```

For more information, see [Setting Up a Node.js Development Environment](/nodejs/docs/setup) .

### PHP

``` text
composer require google/cloud-spanner
```

**Note:** The Spanner Client Library requires that you install and enable the [gRPC extension](/php/grpc) for PHP.

For more information, see [Using PHP on Google Cloud](/php/docs) .

### Python

``` text
pip install --upgrade google-cloud-spanner
```

For more information, see [Setting Up a Python Development Environment](/python/docs/setup) .

### Ruby

``` text
gem install google-cloud-spanner
```

For more information, see [Setting Up a Ruby Development Environment](/ruby/docs/setup) .

## Set up authentication

To authenticate calls to Google Cloud APIs, client libraries support [Application Default Credentials (ADC)](/docs/authentication/application-default-credentials) ; the libraries look for credentials in a set of defined locations and use those credentials to authenticate requests to the API. With ADC, you can make credentials available to your application in a variety of environments, such as local development or production, without needing to modify your application code.

For production environments, the way you set up ADC depends on the service and context. For more information, see [Set up Application Default Credentials](/docs/authentication/provide-credentials-adc) .

For a local development environment, you can set up ADC with the credentials that are associated with your Google Account:

1.  [Install](/sdk/docs/install) the Google Cloud CLI. After installation, [initialize](/sdk/docs/initializing) the Google Cloud CLI by running the following command:
    
    ``` text
    gcloud init
    ```
    
    If you're using an external identity provider (IdP), you must first [sign in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .

2.  If you're using a local shell, then create local authentication credentials for your user account:
    
    ``` text
    gcloud auth application-default login
    ```
    
    You don't need to do this if you're using Cloud Shell.
    
    If an authentication error is returned, and you are using an external identity provider (IdP), confirm that you have [signed in to the gcloud CLI with your federated identity](/iam/docs/workforce-log-in-gcloud) .
    
    A sign-in screen appears. After you sign in, your credentials are stored in the [local credential file used by ADC](/docs/authentication/application-default-credentials#personal) .

## Use the client library

The following example shows how to use the client library.

### C++

``` cpp
#include "google/cloud/spanner/client.h"
void Quickstart(std::string const& project_id, std::string const& instance_id,
                std::string const& database_id) {
  namespace spanner = ::google::cloud::spanner;

  auto database = spanner::Database(project_id, instance_id, database_id);
  auto connection = spanner::MakeConnection(database);
  auto client = spanner::Client(connection);

  auto rows =
      client.ExecuteQuery(spanner::SqlStatement("SELECT 'Hello World'"));
  using RowType = std::tuple<std::string>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << std::get<0>(*row) << "\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System;
using System.Threading.Tasks;

namespace GoogleCloudSamples.Spanner
{
    public class QuickStart
    {
        static async Task MainAsync()
        {
            string projectId = "YOUR-PROJECT-ID";
            string instanceId = "my-instance";
            string databaseId = "my-database";
            string connectionString =
                $"Data Source=projects/{projectId}/instances/{instanceId}/"
                + $"databases/{databaseId}";
            // Create connection to Cloud Spanner.
            using (var connection = new SpannerConnection(connectionString))
            {
                // Execute a simple SQL statement.
                var cmd = connection.CreateSelectCommand(
                    @"SELECT ""Hello World"" as test");
                using (var reader = await cmd.ExecuteReaderAsync())
                {
                    while (await reader.ReadAsync())
                    {
                        Console.WriteLine(
                            reader.GetFieldValue<string>("test"));
                    }
                }
            }
        }
        public static void Main(string[] args)
        {
            MainAsync().Wait();
        }
    }
}
```

### Go

``` go
// Sample spanner_quickstart is a basic program that uses Cloud Spanner.
package main

import (
 "context"
 "fmt"
 "log"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func main() {
 ctx := context.Background()

 // This database must exist.
 databaseName := "projects/your-project-id/instances/your-instance-id/databases/your-database-id"

 client, err := spanner.NewClient(ctx, databaseName)
 if err != nil {
     log.Fatalf("Failed to create client %v", err)
 }
 defer client.Close()

 stmt := spanner.Statement{SQL: "SELECT 1"}
 iter := client.Single().Query(ctx, stmt)
 defer iter.Stop()

 for {
     row, err := iter.Next()
     if err == iterator.Done {
         fmt.Println("Done")
         return
     }
     if err != nil {
         log.Fatalf("Query failed with %v", err)
     }

     var i int64
     if row.Columns(&i) != nil {
         log.Fatalf("Failed to parse row %v", err)
     }
     fmt.Printf("Got value %v\n", i)
 }
}
```

### Java

``` java
// Imports the Google Cloud client library
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.Statement;

/**
 * A quick start code for Cloud Spanner. It demonstrates how to setup the Cloud Spanner client and
 * execute a simple query using it against an existing database.
 */
public class QuickstartSample {
  public static void main(String... args) throws Exception {

    if (args.length != 2) {
      System.err.println("Usage: QuickStartSample <instance_id> <database_id>");
      return;
    }
    // Instantiates a client
    SpannerOptions options = SpannerOptions.newBuilder().build();
    Spanner spanner = options.getService();

    // Name of your instance & database.
    String instanceId = args[0];
    String databaseId = args[1];
    try {
      // Creates a database client
      DatabaseClient dbClient =
          spanner.getDatabaseClient(DatabaseId.of(options.getProjectId(), instanceId, databaseId));
      // Queries the database
      ResultSet resultSet = dbClient.singleUse().executeQuery(Statement.of("SELECT 1"));

      System.out.println("\n\nResults:");
      // Prints the results
      while (resultSet.next()) {
        System.out.printf("%d\n\n", resultSet.getLong(0));
      }
    } finally {
      // Closes the client which will free up the resources used
      spanner.close();
    }
  }
}
```

### Node.js

``` javascript
// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// Creates a client
const spanner = new Spanner({projectId});

// Gets a reference to a Cloud Spanner instance and database
const instance = spanner.instance(instanceId);
const database = instance.database(databaseId);

// The query to execute
const query = {
  sql: 'SELECT 1',
};

// Execute a simple SQL statement
const [rows] = await database.run(query);
console.log(`Query: ${rows.length} found.`);
rows.forEach(row => console.log(row));
```

### PHP

``` php
# Includes the autoloader for libraries installed with composer
require __DIR__ . '/vendor/autoload.php';

# Imports the Google Cloud client library
use Google\Cloud\Spanner\SpannerClient;

# Your Google Cloud Platform project ID
$projectId = 'YOUR_PROJECT_ID';

# Instantiates a client
$spanner = new SpannerClient([
    'projectId' => $projectId
]);

# Your Cloud Spanner instance ID.
$instanceId = 'your-instance-id';

# Get a Cloud Spanner instance by ID.
$instance = $spanner->instance($instanceId);

# Your Cloud Spanner database ID.
$databaseId = 'your-database-id';

# Get a Cloud Spanner database by ID.
$database = $instance->database($databaseId);

# Execute a simple SQL statement.
$results = $database->execute('SELECT "Hello World" as test');

foreach ($results as $row) {
    print($row['test'] . PHP_EOL);
}
```

### Python

``` python
# Imports the Google Cloud Client Library.
from google.cloud import spanner

# Your Cloud Spanner instance ID.
# instance_id = "my-instance-id"
#
# Your Cloud Spanner database ID.
# database_id = "my-database-id"
# Instantiate a client.
spanner_client = spanner.Client()

# Get a Cloud Spanner instance by ID.
instance = spanner_client.instance(instance_id)

# Get a Cloud Spanner database by ID.
database = instance.database(database_id)

# Execute a simple SQL statement.
with database.snapshot() as snapshot:
    results = snapshot.execute_sql("SELECT 1")

    for row in results:
        print(row)
```

### Ruby

``` ruby
# Imports the Google Cloud client library
require "google/cloud/spanner"

# Your Google Cloud Platform project ID
project_id = "YOUR_PROJECT_ID"

# Instantiates a client
spanner = Google::Cloud::Spanner.new project: project_id

# Your Cloud Spanner instance ID
instance_id = "my-instance"

# Your Cloud Spanner database ID
database_id = "my-database"

# Gets a reference to a Cloud Spanner instance database
database_client = spanner.client instance_id, database_id

# Execute a simple SQL statement
results = database_client.execute_query "SELECT 1"
results.rows.each do |row|
  puts row
end
```

## Use the client library for administrator operations

The following list contains links to all the administrator operations you can use in the Spanner client library:

  - [Create and manage instance configuration](/spanner/docs/create-manage-configurations)
  - [Create and manage instances](/spanner/docs/create-manage-instances)
  - [Create and manage databases](/spanner/docs/create-manage-databases)
  - [Secure a database with customer-managed encryption keys (CMEK)](/spanner/docs/use-cmek)
  - [Configure fine-grained access control](/spanner/docs/configure-fgac)
  - [Backup and restore databases using client libraries](/spanner/docs/backup/libraries)
  - [Recover data using point-in-time recovery (PITR)](/spanner/docs/use-pitr#client-libraries)
  - [Prevent accidental database deletion](/spanner/docs/prevent-database-deletion#client-libraries)
  - [Create and manage sequences](/spanner/docs/sequence-tasks)
  - [Modify the leader region of a database](/spanner/docs/modifying-leader-region#client-libraries)
  - Work with [NUMERIC](/spanner/docs/working-with-numerics) , [JSON](/spanner/docs/working-with-json) , and [JSONB](/spanner/docs/working-with-jsonb) data types

## Use Spanner Graph with client libraries

You can use client libraries to setup, modify, and query property graphs in Spanner Graph. The following list contains links to learn about and get started with Spanner Graph using client libraries:

  - [Learn about Spanner Graph](/spanner/docs/graph/overview)
  - [Set up and query Spanner Graph](/spanner/docs/graph/set-up)
  - [Insert, update, or delete Spanner Graph data](/spanner/docs/graph/insert-update-delete-data)

## Additional resources

### C++

The following list contains links to more resources related to the client library for C++:

  - [API reference](/cpp/docs/reference/spanner/latests)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/google-cloud-cpp/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D%5Bc%2B%2B%5D)
  - [Source code](https://github.com/googleapis/google-cloud-cpp)
  - [Getting started in C++](/spanner/docs/getting-started/cpp)

### C\#

The following list contains links to more resources related to the client library for C\#:

  - [API reference](/dotnet/docs/reference)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/google-cloud-dotnet/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bc%23%5D)
  - [Source code](https://github.com/googleapis/google-cloud-dotnet)
  - [Getting started in C\#](/spanner/docs/getting-started/csharp)

### Go

The following list contains links to more resources related to the client library for Go:

  - [API reference](https://pkg.go.dev/cloud.google.com/go/spanner/)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/google-cloud-go/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bgo%5D)
  - [Source code](https://github.com/googleapis/google-cloud-go)
  - [Getting started in Go](/spanner/docs/getting-started/go)

### Java

The following list contains links to more resources related to the client library for Java:

  - [API reference](https://cloud.google.com/java/docs/reference/google-cloud-spanner/latest/overview)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/java-spanner/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bjava%5D)
  - [Source code](https://github.com/googleapis/java-spanner)
  - [Getting started in Java](/spanner/docs/getting-started/java)

### Node.js

The following list contains links to more resources related to the client library for Node.js:

  - [API reference](https://googleapis.dev/nodejs/spanner/latest)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/nodejs-spanner/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bnode.js%5D)
  - [Source code](https://github.com/googleapis/nodejs-spanner)
  - [Getting started in Node.js](/spanner/docs/getting-started/nodejs)

### PHP

The following list contains links to more resources related to the client library for PHP:

  - [API reference](/php/docs/reference/cloud-spanner/latest/readme)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/google-cloud-php/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bphp%5D)
  - [Source code](https://github.com/googleapis/google-cloud-php)
  - [Getting started in PHP](/spanner/docs/getting-started/php)

### Python

The following list contains links to more resources related to the client library for Python:

  - [API reference](/python/docs/reference/spanner/latest)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/python-spanner/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bpython%5D)
  - [Source code](https://github.com/googleapis/python-spanner)
  - [Getting started in Python](/spanner/docs/getting-started/python)

### Ruby

The following list contains links to more resources related to the client library for Ruby:

  - [API reference](/ruby/docs/reference/google-cloud-spanner/latest)
  - [Client libraries best practices](/apis/docs/client-libraries-best-practices)
  - [Issue tracker](https://github.com/googleapis/google-cloud-ruby/issues)
  - [`  google-cloud-spanner  ` on Stack Overflow](https://stackoverflow.com/search?q=%5Bgoogle-cloud-spanner%5D+%5Bruby%5D)
  - [Source code](https://github.com/googleapis/google-cloud-ruby)
  - [Getting started in Ruby](/spanner/docs/getting-started/ruby)

## Additional client libraries

In addition to the libraries shown above, a [Spring Data](https://spring.io/projects/spring-data) module is available for Java applications. Spring Data Spanner helps you use Spanner in any application that's built with the [Spring Framework](https://spring.io/projects/spring-framework) .

To get started, learn how to [add Spring Data Spanner to your application](/spanner/docs/adding-spring) .
