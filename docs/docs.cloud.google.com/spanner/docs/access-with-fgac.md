This page explains how to access a Spanner database when you are a fine-grained access control user.

To learn about fine-grained access control, see [About fine-grained access control](/spanner/docs/fgac-about) .

As a fine-grained access control user, you must select a database role to use to execute SQL statements and queries, and to perform row operations on a database. Your role selection persists throughout your session until you change the role.

When you submit a query, DML, or row operation, Spanner checks authorization by using the following rules:

  - Google Cloud console  
    Spanner first checks if you have [database-level Identity and Access Management (IAM) permissions](/spanner/docs/iam#databases) . If so, the Google Cloud console doesn't show a database role selector, and your session proceeds with your database-level permissions.

<!-- end list -->

  - If you have only fine-grained access control privileges and no IAM database-level permissions, you must have been granted access to the `  spanner_sys_reader  ` system role or one of its member roles. Select a role on the database **Overview** page so that your Google Cloud console session proceeds with the required privileges.
    
    **Note:** To interact with Spanner resources in the Google Cloud console, you must have the `  roles/spanner.viewer  ` IAM role.

<!-- end list -->

  - Google Cloud SDK  
    If you specify a database role when you submit a query, DML, or a row operation, Spanner checks fine-grained access control privileges. If the check fails, Spanner does not check for database-level IAM permissions, and the operation fails.
    
    If you don't specify a database role, Spanner checks database-level IAM permissions, and if the checks succeed, your session proceeds with your database-level permissions.

Use these methods to specify a database role when accessing a Spanner database:

### Console

1.  Select a database, and then on the database **Overview** page, click the **Change database role** (pencil) icon adjacent to the **Current role** field.
    
    By default, when a fine-grained access control user logs in, this field has the value `  public  ` . For information about the `  public  ` system role, see [Fine-grained access control system roles](/spanner/docs/fgac-system-roles) .
    
    **Note:** As stated earlier, if a fine-grained access control user has [database-level IAM permissions](/spanner/docs/iam#databases) , the **Current role** field isn't shown.

2.  In the **Change database role** dialog, select another role from the list of available roles.

3.  Click **Update** .
    
    The **Current role** field shows the new role.

### gcloud

  - Add the `  --database-role  ` option to the [`  gcloud spanner databases execute-sql  `](/sdk/gcloud/reference/spanner/databases/execute-sql) command, as follows:
    
    ``` text
    gcloud spanner databases execute-sql DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --sql="SELECT * from TABLE_NAME;" \
    --database-role=ROLE_NAME
    ```

### Client libraries

### C++

``` cpp
void ReadDataWithDatabaseRole(std::string const& project_id,
                              std::string const& instance_id,
                              std::string const& database_id,
                              std::string const& role) {
  namespace spanner = ::google::cloud::spanner;
  auto client = spanner::Client(spanner::MakeConnection(
      spanner::Database(project_id, instance_id, database_id),
      google::cloud::Options{}.set<spanner::SessionCreatorRoleOption>(role)));
  spanner::SqlStatement select_star("SELECT * FROM Singers");
  auto rows = client.ExecuteQuery(std::move(select_star));
  using RowType =
      std::tuple<std::int64_t, std::string, std::string, spanner::Bytes>;
  for (auto& row : spanner::StreamOf<RowType>(rows)) {
    if (!row) throw std::move(row).status();
    std::cout << "SingerId: " << std::get<0>(*row) << ", "
              << "FirstName: " << std::get<1>(*row) << ", "
              << "LastName: " << std::get<2>(*row) << "\n";
  }
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Collections.Generic;
using System.Threading.Tasks;

public class ReadDataWithDatabaseRoleAsyncSample
{
    public class Singer
    {
        public int SingerId { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    }

    public async Task<List<Singer>> ReadDataWithDatabaseRoleAsync(string projectId, string instanceId, string databaseId, string databaseRole)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string tableName = "Singers";

        var spannerConnectionStringBuilder = new SpannerConnectionStringBuilder
        {
            ConnectionString = connectionString,
            DatabaseRole = databaseRole
        };
        using var connection = new SpannerConnection(spannerConnectionStringBuilder);
        var createSelectCmd = connection.CreateSelectCommand($"SELECT * FROM {tableName}");
        using var reader = await createSelectCmd.ExecuteReaderAsync();
        var singers = new List<Singer>();
        while (await reader.ReadAsync())
        {
            singers.Add(new Singer
            {
                SingerId = reader.GetFieldValue<int>("SingerId"),
                FirstName = reader.GetFieldValue<string>("FirstName"),
                LastName = reader.GetFieldValue<string>("LastName"),
            });
        }
        return singers;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/spanner"
 "google.golang.org/api/iterator"
)

func readDataWithDatabaseRole(w io.Writer, db string, databaseRole string) error {
 // databaseRole = "parent"
 ctx := context.Background()
 cfg := spanner.ClientConfig{
     DatabaseRole: databaseRole,
 }
 client, err := spanner.NewClientWithConfig(ctx, db, cfg)
 if err != nil {
     return err
 }
 defer client.Close()

 // Read all albums.
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

### Java

``` java
import com.google.cloud.spanner.DatabaseClient;
import com.google.cloud.spanner.DatabaseId;
import com.google.cloud.spanner.KeySet;
import com.google.cloud.spanner.ResultSet;
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import java.util.Arrays;

public class ReadDataWithDatabaseRole {

  static void readDataWithDatabaseRole() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String role = "my-role";
    readDataWithDatabaseRole(projectId, instanceId, databaseId, role);
  }

  static void readDataWithDatabaseRole(
      String projectId, String instanceId, String databaseId, String role) {
    try (Spanner spannerWithRole =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .setDatabaseRole(role)
            .build()
            .getService()) {
      DatabaseClient dbClient =
          spannerWithRole.getDatabaseClient(DatabaseId.of(projectId, instanceId, databaseId));
      ResultSet resultSet =
          dbClient
              .singleUse()
              .read(
                  "Singers",
                  KeySet.all(),
                  Arrays.asList("SingerId", "FirstName", "LastName"));
      while (resultSet.next()) {
        System.out.printf("SingerId: %d\n", resultSet.getLong(0));
        System.out.printf("FirstName: %s\n", resultSet.getString(1));
        System.out.printf("LastName: %s\n", resultSet.getString(2));
      }
    }
  }
}
```

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';
// Imports the Google Cloud Spanner client library
const {Spanner} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function readDataWithDatabaseRole() {
  // Gets a reference to a Cloud Spanner instance and database.
  const instance = spanner.instance(instanceId);
  // Connect to a database using the 'parent' database role. This means that the connection will only have the permissions that have explicitly been granted to the 'parent' role.
  const options = {
    databaseRole: 'parent',
  };
  const database = instance.database(databaseId, options);

  try {
    const query = {
      sql: 'SELECT SingerId, FirstName, LastName FROM Singers',
    };
    const [rows] = await database.run(query);

    for (const row of rows) {
      const json = row.toJSON();

      console.log(
        `SingerId: ${json.SingerId}, FirstName: ${json.FirstName}, LastName: ${json.LastName}`,
      );
    }
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the database when finished.
    await database.close();
  }
}
readDataWithDatabaseRole();
```

### PHP

```` php
use Google\Cloud\Spanner\SpannerClient;

/**
 * Read database with a database role.
 * Example:
 * ```
 * read_data_with_database_role($instanceId, $databaseId);
 * ```
 *
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function read_data_with_database_role(string $instanceId, string $databaseId): void
{
    $spanner = new SpannerClient();
    $databaseRole = 'new_parent';
    $instance = $spanner->instance($instanceId);
    $database = $instance->database($databaseId, ['databaseRole' => $databaseRole]);
    $results = $database->execute('SELECT * FROM Singers');

    foreach ($results as $row) {
        printf('SingerId: %s, Firstname: %s, LastName: %s' . PHP_EOL, $row['SingerId'], $row['FirstName'], $row['LastName']);
    }
}
````

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
spanner_client = spanner.Client()
instance = spanner_client.instance(instance_id)
role = "new_parent"
database = instance.database(database_id, database_role=role)

with database.snapshot() as snapshot:
    results = snapshot.execute_sql("SELECT * FROM Singers")
    for row in results:
        print("SingerId: {}, FirstName: {}, LastName: {}".format(*row))
```

### Ruby

``` ruby
require "google/cloud/spanner"

def spanner_read_data_with_database_role project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  role = "new_parent"
  spanner = Google::Cloud::Spanner.new project: project_id
  client = spanner.client instance_id, database_id, database_role: role

  result = client.execute_sql "SELECT * FROM Singers"

  result.rows.each do |row|
    puts "SingerId: #{row[:SingerId]}"
    puts "FirstName: #{row[:FirstName]}"
    puts "LastName: #{row[:LastName]}"
  end
end
```
