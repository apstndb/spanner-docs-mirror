This page explains how to configure fine-grained access control for Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

To learn about fine-grained access control, see [About fine-grained access control](/spanner/docs/fgac-about) .

Configure fine-grained access control by following these steps:

1.  [Create database roles and grant privileges](#create-fgac-role) .

2.  [Optional: Create a hierarchy of roles with inheritance](#role-hierarchy) .

3.  [Grant database roles to principals](#grant-fgac-role) .

4.  [Inform users and developers to use database roles](#inform-users) .

Fine-grained access control users must then specify a database role to perform queries, DML, or row operations against the database.

## Before you begin

Ensure that each principal who is to be a fine-grained access control user is granted the `  Cloud Spanner Viewer  ` Identity and Access Management (IAM) role ( `  roles/spanner.viewer  ` ). This role is recommended at the project level for users who must interact with Spanner resources in the Google Cloud console.

For instructions, see [Grant permissions to principals](/spanner/docs/grant-permissions#grant_permissions_to_principals) .

## Create database roles and grant privileges

A database role is a collection of fine-grained access privileges. You can create up to 100 database roles for each database.

Decide on roles and role hierarchies in your database and encode them in DDL. As with other schema changes in Spanner, we strongly recommend issuing schema changes in a batch rather than separately. For more information, see [Limit the frequency of schema updates](/spanner/docs/schema-updates#frequency) .

### Console

To create a database role and grant fine-grained access privileges to it, follow these steps:

1.  Go to the **Instances** page in the Google Cloud console.

2.  Select the instance containing the database for which you want to add the role.

3.  Select the database.

4.  On the **Overview** page, click **Spanner Studio** .

5.  On the **Spanner Studio** page, for each database role that you want to create and grant privileges to, follow these steps:
    
    1.  To create the role, enter the following statement:
        
        ``` text
        CREATE ROLE ROLE_NAME;
        ```
        
        Don't click **Submit** yet.
    
    2.  To grant privileges to the role, enter a `  GRANT  ` statement on the next line after the `  CREATE ROLE  ` statement.
        
        For syntax details for the `  GRANT  ` statement, see [GoogleSQL data definition language](/spanner/docs/reference/standard-sql/data-definition-language#grant_statement) . For information about privileges, see [Fine-grained access control privileges](/spanner/docs/fgac-privileges) .
        
        For example, to grant `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` on tables `  employees  ` and `  contractors  ` to the database role `  hr_manager  ` , enter the following statement:
        
        ### GoogleSQL
        
        ``` text
        GRANT SELECT, INSERT, UPDATE ON TABLE employees, contractors TO ROLE hr_manager;
        ```
        
        ### PostgreSQL
        
        ``` text
        GRANT SELECT, INSERT, UPDATE ON TABLE employees, contractors TO hr_manager;
        ```
        
        **Note:** Use the same case for schema object names that you used when you created the objects. For example, if you created a table named `  Customers  ` , you must use the name `  Customers  ` (with a capital C) in the `  GRANT  ` statement.
        
        You can use a DDL template for the `  GRANT  ` statement. In the **Explorer** pane, navigate to the role for which you want to grant privileges. Click more\_vert **View actions** and select the type of privilege you want to grant access to for this role. The `  GRANT  ` template statement populates in a new editor tab.

6.  Click **Submit** .
    
    If there are errors in your DDL, the Google Cloud console returns an error.

### gcloud

To create a database role and grant fine-grained access privileges to it, use the [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) command with `  CREATE ROLE  ` and `  GRANT  ` statements.

For syntax details on the `  CREATE ROLE  ` and `  GRANT  ` statements, see [GoogleSQL data definition language](/spanner/docs/reference/standard-sql/data-definition-language) .

For example, use the following command to create a database role and grant privileges to it on one or more tables.

### GoogleSQL

``` text
gcloud spanner databases ddl update DATABASE_NAME --instance=INSTANCE_NAME \
--ddl='CREATE ROLE ROLE_NAME; GRANT PRIVILEGES ON TABLE TABLES TO ROLE ROLE_NAME;'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update DATABASE_NAME --instance=INSTANCE_NAME \
--ddl='CREATE ROLE ROLE_NAME; GRANT PRIVILEGES ON TABLE TABLES TO ROLE_NAME;'
```

Replace the following:

  - `  PRIVILEGES  ` is a comma-delimited list of fine-grained access control privileges. For information about privileges, see [Fine-grained access control privileges](/spanner/docs/fgac-privileges) .

  - `  TABLES  ` is a comma-delimited list of tables.

For example, to grant `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` on the `  employees  ` and `  contractors  ` tables to the database role `  hr_analyst  ` in the database `  hrdb1  ` in the instance `  hr  ` , enter the following statement:

### GoogleSQL

``` text
gcloud spanner databases ddl update hrdb1 --instance=hr \
--ddl='CREATE ROLE hr_analyst; GRANT SELECT, INSERT, UPDATE ON TABLE employees, contractors TO ROLE hr_analyst;'
```

### PostgreSQL

``` text
gcloud spanner databases ddl update hrdb1 --instance=hr \
--ddl='CREATE ROLE hr_analyst; GRANT SELECT, INSERT, UPDATE ON TABLE employees, contractors TO hr_analyst;'
```

**Note:** Use the same case for schema object names that you used when you created the objects. For example, if you created a table named `  Customers  ` , you must use the name `  Customers  ` (with capital C) in the `  GRANT  ` statement.

### Client libraries

These code samples both create and drop a database role.

### C++

``` cpp
void AddAndDropDatabaseRole(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& role_parent,
    std::string const& role_child) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> grant_statements = {
      "CREATE ROLE " + role_parent,
      "GRANT SELECT ON TABLE Singers TO ROLE " + role_parent,
      "CREATE ROLE " + role_child,
      "GRANT ROLE " + role_parent + " TO ROLE " + role_child,
  };
  auto metadata =
      client.UpdateDatabaseDdl(database.FullName(), grant_statements).get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Created roles " << role_parent << " and " << role_child
            << " and granted privileges\n";

  std::vector<std::string> revoke_statements = {
      "REVOKE ROLE " + role_parent + " FROM ROLE " + role_child,
      "DROP ROLE " + role_child,
  };
  metadata =
      client.UpdateDatabaseDdl(database.FullName(), revoke_statements).get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Revoked privileges and dropped role " << role_child << "\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class AddAndDropDatabaseRoleAsyncSample
{
    public async Task AddDatabaseRoleAsync(string projectId, string instanceId, string databaseId, string databaseRole)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string createRoleStatement = $"CREATE ROLE {databaseRole}";

        // Creates the given database role.
        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(createRoleStatement);
        await updateCmd.ExecuteNonQueryAsync();
    }

    public async Task DropDatabaseRoleAsync(string projectId, string instanceId, string databaseId, string databaseRole)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string deleteRoleStatement = $"DROP ROLE {databaseRole}";

        // Drops the given database role.
        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(deleteRoleStatement);
        await updateCmd.ExecuteNonQueryAsync();
    }
}
```

### Go

``` go
import (
 "context"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addAndDropDatabaseRole(w io.Writer, db string) error {
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // Set up database roles and membership. After database roles are created,
 // users can be granted roles by setting IAM policies.
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "CREATE ROLE parent",
         "GRANT SELECT ON TABLE Albums TO ROLE parent",
         "CREATE ROLE child",
         "GRANT ROLE parent TO ROLE child",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }

 // Delete role and membership.
 op, err = adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "REVOKE ROLE parent FROM ROLE child",
         "DROP ROLE child",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class AddAndDropDatabaseRole {

  static void addAndDropDatabaseRole() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String parentRole = "parent_role";
    String childRole = "child_role";
    addAndDropDatabaseRole(projectId, instanceId, databaseId, parentRole, childRole, "Albums");
  }

  static void addAndDropDatabaseRole(
      String projectId, String instanceId, String databaseId,
      String parentRole, String childRole, String... tables) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      System.out.println("Waiting for role create operation to complete...");
      List<String> roleStatements = new ArrayList<>(ImmutableList.of(
          String.format("CREATE ROLE %s", parentRole),
          String.format("CREATE ROLE %s", childRole),
          String.format("GRANT ROLE %s TO ROLE %s", parentRole, childRole)));
      for (String table : tables) {
        roleStatements.add(String.format("GRANT SELECT ON TABLE %s TO ROLE %s", table, parentRole));
      }
      databaseAdminClient.updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId), roleStatements)
          .get(5, TimeUnit.MINUTES);
      System.out.printf(
          "Created roles %s and %s and granted privileges%n", parentRole, childRole);
      // Delete role and membership.
      System.out.println("Waiting for role revoke & drop operation to complete...");
      databaseAdminClient.updateDatabaseDdlAsync(
          DatabaseName.of(projectId, instanceId, databaseId),
          ImmutableList.of(
              String.format("REVOKE ROLE %s FROM ROLE %s", parentRole, childRole),
              String.format("DROP ROLE %s", childRole))).get(5, TimeUnit.MINUTES);
      System.out.printf("Revoked privileges and dropped role %s%n", childRole);
    } catch (ExecutionException | TimeoutException e) {
      System.out.printf(
          "Error: AddAndDropDatabaseRole failed with error message %s\n", e.getMessage());
      e.printStackTrace();
    } catch (InterruptedException e) {
      System.out.println(
          "Error: Waiting for AddAndDropDatabaseRole operation to finish was interrupted");
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function addAndDropNewDatabaseRole() {
  // Creates a new user defined role and grant permissions
  try {
    const request = [
      'CREATE ROLE parent',
      'GRANT SELECT ON TABLE Singers TO ROLE parent',
      'CREATE ROLE child',
      'GRANT ROLE parent TO ROLE child',
    ];
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log('Created roles child and parent and granted privileges');
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Revoke permissions and drop child role.
  // A role can't be dropped until all its permissions are revoked.
  try {
    const request = ['REVOKE ROLE parent FROM ROLE child', 'DROP ROLE child'];
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log('Revoked privileges and dropped role child');
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the spanner client when finished.
    // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
    spanner.close();
  }
}
addAndDropNewDatabaseRole();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds and drops roles to the Singers table in the example database.
 * Example:
 * ```
 * add_drop_database_role($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_drop_database_role(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            'CREATE ROLE new_parent',
            'GRANT SELECT ON TABLE Singers TO ROLE new_parent',
            'CREATE ROLE new_child',
            'GRANT ROLE new_parent TO ROLE new_child'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    printf('Waiting for create role and grant operation to complete...%s', PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created roles %s and %s and granted privileges%s', 'new_parent', 'new_child', PHP_EOL);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            'REVOKE ROLE new_parent FROM ROLE new_child',
            'DROP ROLE new_child'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    printf('Waiting for revoke role and drop role operation to complete...%s', PHP_EOL);
    $operation->pollUntilComplete();

    printf('Revoked privileges and dropped role %s%s', 'new_child', PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

spanner_client = spanner.Client()
database_admin_api = spanner_client.database_admin_api

role_parent = "new_parent"
role_child = "new_child"

request = spanner_database_admin.UpdateDatabaseDdlRequest(
    database=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    statements=[
        "CREATE ROLE {}".format(role_parent),
        "GRANT SELECT ON TABLE Singers TO ROLE {}".format(role_parent),
        "CREATE ROLE {}".format(role_child),
        "GRANT ROLE {} TO ROLE {}".format(role_parent, role_child),
    ],
)
operation = database_admin_api.update_database_ddl(request)

operation.result(OPERATION_TIMEOUT_SECONDS)
print(
    "Created roles {} and {} and granted privileges".format(role_parent, role_child)
)

request = spanner_database_admin.UpdateDatabaseDdlRequest(
    database=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    statements=[
        "REVOKE ROLE {} FROM ROLE {}".format(role_parent, role_child),
        "DROP ROLE {}".format(role_child),
    ],
)
operation = database_admin_api.update_database_ddl(request)

operation.result(OPERATION_TIMEOUT_SECONDS)
print("Revoked privileges and dropped role {}".format(role_child))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

def spanner_add_and_drop_database_role project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  admin_client = Google::Cloud::Spanner::Admin::Database::V1::DatabaseAdmin::Client.new
  role_parent = "new_parent"
  role_child = "new_child"

  db_path = admin_client.database_path project: project_id, instance: instance_id, database: database_id

  job = admin_client.update_database_ddl database: db_path, statements: [
    "CREATE ROLE #{role_parent}",
    "GRANT SELECT ON TABLE Singers TO ROLE #{role_parent}",
    "CREATE ROLE #{role_child}",
    "GRANT ROLE #{role_parent} TO ROLE #{role_child}"
  ]

  job.wait_until_done!
  puts "Created roles #{role_parent} and #{role_child} and granted privileges"


  job = admin_client.update_database_ddl database: db_path, statements: [
    "REVOKE ROLE #{role_parent} FROM ROLE #{role_child}",
    "DROP ROLE #{role_child}"
  ]

  job.wait_until_done!
  puts "Revoked privileges and dropped role #{role_child}"
end
```

## Create a hierarchy of roles with inheritance

You can create a hierarchy of database roles by granting one database role to another. Child roles (known as *member roles* ) inherit privileges from the parent.

To grant a database role to another database role, use the following statement:

### GoogleSQL

``` text
GRANT ROLE role1 TO ROLE role2;
```

### PostgreSQL

``` text
GRANT role1 TO role2;
```

For more information, see [Database role hierarchies and inheritance](/spanner/docs/fgac-about#role_hierarchy) .

## Grant database roles to principals

To access Spanner resources, a principal must be granted the necessary database role using one of the following options:

### Console

1.  On the database **Overview** page, click **SHOW INFO PANEL** if the **Info panel** is not already open.

2.  Click **ADD PRINCIPAL** .

3.  Under **Add principals** , in **New principals** , enter one or more principals.

4.  Under **Assign roles** , in **Select a role** , select **Cloud Spanner** \> **Cloud Spanner Fine-grained Access User** .
    
    You need to grant this role only once to each principal. It makes the principal a fine-grained access control user.

5.  Click **ADD ANOTHER ROLE** .

6.  In **Select a role** , select **Cloud Spanner** \> **Cloud Spanner Database Role User** .
    
    **Caution:** Avoid granting the Cloud Spanner Database User . role. It provides excessive permissions. For details, see [Cloud Spanner Database User](/spanner/docs/iam#spanner.databaseUser) . .

7.  Follow these steps to create the IAM condition that specifies the roles to grant.
    
    **Caution:** If you don't add a condition, the principal has access to all database roles.
    
    1.  Next to the Cloud Spanner Database Role User role, click **ADD IAM CONDITION** .
    
    2.  In the **Add condition** panel, enter a title and optional description for the condition.
        
        If you're granting a single database role, you typically include the role name in the condition title. If you're granting multiple roles, you can indicate something about the set of roles.
    
    3.  Click **Condition editor** .
    
    4.  In the **Expression** field, enter the following code:
        
        ``` text
        resource.type == "spanner.googleapis.com/DatabaseRole" &&
        resource.name.endsWith("/ROLE")
        ```
        
        Replace `  ROLE  ` with your role name.
        
        To grant more than one role to the principal, add more conditions with the **or** ( `  ||  ` ) operator, as shown in the following example:
        
        ``` text
        resource.type == "spanner.googleapis.com/DatabaseRole" &&
        (resource.name.endsWith("/ROLE1") || resource.name.endsWith("/ROLE2"))
        ```
        
        This code grants two roles. Replace `  ROLE1  ` and `  ROLE2  ` with your role names. To grant more than two roles, add more **or** conditions.
        
        You can use any condition expression that's supported by IAM. For more information, see [Overview of IAM conditions](https://cloud.google.com/iam/docs/conditions-overview) .
    
    5.  Click **Save** .
    
    6.  Verify that the condition appears in the **IAM Condition** column next to the **Role** field.
    
    7.  Click **Save** .
        
        Back on the **Info panel** , under **Role/Principal** , notice that **Cloud Spanner Database Role User** appears for each defined condition.
        
        The number in parentheses next to the condition indicates the number of principals who are granted the database role by that condition. You can click the expander arrow to view the list of principals.
    
    8.  To correct errors in database role names or conditions, or to add additional database roles for a principal, follow these steps:
        
        1.  Expand the **Cloud Spanner Database Role User** entry that lists the condition that you want.
        
        2.  Click the **Edit** (pencil) icon next to a principal.
        
        3.  On the **Edit access to *database\_name*** panel, do one of the following:
            
              - Click **ADD ANOTHER ROLE** .
            
              - To edit the condition, click the **Edit** (pencil) icon adjacent to the condition name. Then on the **Edit condition** page, click **Condition editor** , make corrections, and click **Save** twice.

### gcloud

1.  Enable fine-grained access control for the principal by using the [`  gcloud spanner databases add-iam-policy-binding  `](/sdk/gcloud/reference/spanner/databases/add-iam-policy-binding) command as follows:
    
    ``` text
    gcloud spanner databases add-iam-policy-binding DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --role=roles/spanner.fineGrainedAccessUser \
    --member=MEMBER_NAME \
    --condition=None
    ```
    
      - `  MEMBER_NAME  ` is the identifier for the principal. It must use one of the following syntaxes: `  user|group|serviceAccount: email  ` or `  domain: domain  ` .
    
      - This command makes the principal a fine-grained access control user. Submit this command only once for each principal.
    
      - If successful, the command outputs the entire policy for the database.

2.  Grant permission to use one or more database roles by using the [`  gcloud spanner databases add-iam-policy-binding  `](/sdk/gcloud/reference/spanner/databases/add-iam-policy-binding) command as follows:
    
    ``` text
    gcloud spanner databases add-iam-policy-binding DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --role=roles/spanner.databaseRoleUser \
    --member=MEMBER_NAME \
    --condition=CONDITION
    ```
    
      - `  MEMBER_NAME  ` is the identifier for the principal. It must use one of the following syntaxes: `  user|group|serviceAccount: email  ` or `  domain: domain  ` .
    
      - `  CONDITION  ` is an IAM condition expression that specifies the roles to grant to the principal.
        
        `  CONDITION  ` has the following form:
        
        ``` text
        --condition='expression=(resource.type == "spanner.googleapis.com/DatabaseRole" && resource.name.endsWith("/ROLE1")),title=TITLE,description=DESCRIPTION'
        ```
        
        Or, to grant access to the principal to more than one role, add more conditions with the **or** ( `  ||  ` ) operator, as shown in the following example:
        
        ``` text
        --condition='expression=(resource.type == "spanner.googleapis.com/DatabaseRole" && (resource.name.endsWith("/ROLE1") || resource.name.endsWith("/ROLE2"))),title=TITLE,description=DESCRIPTION'
        ```
        
        This code grants two roles. Replace `  ROLE1  ` and `  ROLE2  ` with your role names. To grant more than two roles, add more **or** conditions with the `  ||  ` operator.
        
        You can use any condition expression that's supported by IAM. For more information, see [Overview of IAM conditions](https://cloud.google.com/iam/docs/conditions-overview) .
    
    **Caution:** If you don't specify a condition, the principal has access to all database roles.
    
    If successful, the command outputs the entire policy for the database.
    
    The following example grants the database roles `  hr_rep  ` and `  hr_manager  ` to the principal `  jsmith@example.com  ` .
    
    ``` text
    gcloud spanner databases add-iam-policy-binding myDatabase \
      --instance=myInstance \
      --role=roles/spanner.databaseRoleUser \
      --member=user:jsmith@example.com \
      --condition='expression=(resource.type == "spanner.googleapis.com/DatabaseRole" && (resource.name.endsWith("/hr_rep") || resource.name.endsWith("/hr_manager"))),title=HR roles,description=Grant permissions on HR roles'
    ```

### Client libraries

### C++

``` cpp
void EnableFineGrainedAccess(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& iam_member,
    std::string const& database_role, std::string const& title) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);

  google::iam::v1::GetIamPolicyRequest request;
  request.set_resource(database.FullName());
  request.mutable_options()->set_requested_policy_version(3);
  auto policy = client.GetIamPolicy(request);
  if (!policy) throw std::move(policy).status();
  if (policy->version() < 3) policy->set_version(3);

  auto& binding = *policy->add_bindings();
  binding.set_role("roles/spanner.fineGrainedAccessUser");
  binding.add_members(iam_member);
  auto& condition = *binding.mutable_condition();
  condition.set_expression("resource.name.endsWith(\"/databaseRoles/" +
                           database_role + "\")");
  condition.set_title(title);

  auto new_policy =
      client.SetIamPolicy(database.FullName(), *std::move(policy));
  if (!new_policy) throw std::move(new_policy).status();
  std::cout << "Enabled fine-grained access in IAM. New policy has version "
            << new_policy->version() << "\n";
}
```

### C\#

``` csharp
using Google.Api.Gax;
using Google.Cloud.Iam.V1;
using Google.Cloud.Spanner.Admin.Database.V1;

public class EnableFineGrainedAccessSample
{
    public Policy EnableFineGrainedAccess(
        string projectId, string instanceId, string databaseId, 
        string databaseRole, string iamMember)
    {
        var resourceName = new UnparsedResourceName($"projects/{projectId}/instances/{instanceId}/databases/{databaseId}");

        var client = new DatabaseAdminClientBuilder().Build();

        // Request policy version 3 as earlier versions do not support condition field in role binding.
        // For more information see https://cloud.google.com/iam/docs/policies#versions.

        GetIamPolicyRequest getIamPolicyRequest = new GetIamPolicyRequest
        {
            ResourceAsResourceName = resourceName,
            Options = new GetPolicyOptions
            {
                RequestedPolicyVersion = 3
            }
        };

        var policy = client.GetIamPolicy(getIamPolicyRequest);

        // Gives the given IAM member access to the all the database roles
        // with resource name ending in ../databaseRoles/{databaseRole}.
        // For more information see https://cloud.google.com/iam/docs/conditions-overview.
        Binding newBinding = new Binding
        {
            Role = "roles/spanner.fineGrainedAccessUser",
            Members = { iamMember },
            Condition = new Google.Type.Expr
            {
                Title = "DatabaseRoleBindingTitle",
                Expression = $"resource.name.endsWith('/databaseRoles/{databaseRole}')"
            }
        };

        policy.Bindings.Add(newBinding);
        if (policy.Version < 3)
        {
            policy.Version = 3;
        }
        SetIamPolicyRequest setIamPolicyRequest = new SetIamPolicyRequest
        {
            Policy = policy,
            ResourceAsResourceName = resourceName,
        };
        var updatedPolicy = client.SetIamPolicy(setIamPolicyRequest);
        return updatedPolicy;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"

 "cloud.google.com/go/iam/apiv1/iampb"
 database "cloud.google.com/go/spanner/admin/database/apiv1"
 expr "google.golang.org/genproto/googleapis/type/expr"
)

func enableFineGrainedAccess(w io.Writer, db string, iamMember string, databaseRole string, title string) error {
 // iamMember = "user:alice@example.com"
 // databaseRole = "parent"
 // title = "condition title"
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 policy, err := adminClient.GetIamPolicy(ctx, &iampb.GetIamPolicyRequest{
     Resource: db,
     Options: &iampb.GetPolicyOptions{
         // IAM conditions need at least version 3
         RequestedPolicyVersion: 3,
     },
 })
 if err != nil {
     return err
 }

 // IAM conditions need at least version 3
 if policy.Version < 3 {
     policy.Version = 3
 }
 policy.Bindings = append(policy.Bindings, []*iampb.Binding{
     {
         Role:    "roles/spanner.fineGrainedAccessUser",
         Members: []string{iamMember},
     },
     {
         Role:    "roles/spanner.databaseRoleUser",
         Members: []string{iamMember},
         Condition: &expr.Expr{
             Expression: fmt.Sprintf(`resource.name.endsWith("/databaseRoles/%s")`, databaseRole),
             Title:      title,
         },
     },
 }...)
 _, err = adminClient.SetIamPolicy(ctx, &iampb.SetIamPolicyRequest{
     Resource: db,
     Policy:   policy,
 })
 if err != nil {
     return err
 }

 fmt.Fprintf(w, "Enabled fine-grained access in IAM.\n")
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.iam.v1.Binding;
import com.google.iam.v1.GetIamPolicyRequest;
import com.google.iam.v1.GetPolicyOptions;
import com.google.iam.v1.Policy;
import com.google.iam.v1.SetIamPolicyRequest;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.type.Expr;

public class EnableFineGrainedAccess {

  static void enableFineGrainedAccess() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String iamMember = "user:alice@example.com";
    String role = "my-role";
    String title = "my-condition-title";
    enableFineGrainedAccess(projectId, instanceId, databaseId, iamMember, title, role);
  }

  static void enableFineGrainedAccess(
      String projectId,
      String instanceId,
      String databaseId,
      String iamMember,
      String title,
      String role) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      final GetPolicyOptions options =
          GetPolicyOptions.newBuilder().setRequestedPolicyVersion(3).build();
      final GetIamPolicyRequest getRequest =
          GetIamPolicyRequest.newBuilder()
              .setResource(DatabaseName.of(projectId, instanceId, databaseId).toString())
              .setOptions(options).build();
      final Policy policy = databaseAdminClient.getIamPolicy(getRequest);
      int policyVersion = policy.getVersion();
      // The policy in the response from getDatabaseIAMPolicy might use the policy version
      // that you specified, or it might use a lower policy version. For example, if you
      // specify version 3, but the policy has no conditional role bindings, the response
      // uses version 1. Valid values are 0, 1, and 3.
      if (policy.getVersion() < 3) {
        // conditional role bindings work with policy version 3
        policyVersion = 3;
      }

      Binding binding1 =
          Binding.newBuilder()
              .setRole("roles/spanner.fineGrainedAccessUser")
              .addAllMembers(ImmutableList.of(iamMember))
              .build();

      Binding binding2 =
          Binding.newBuilder()
              .setRole("roles/spanner.databaseRoleUser")
              .setCondition(
                  Expr.newBuilder().setDescription(title).setExpression(
                      String.format("resource.name.endsWith(\"/databaseRoles/%s\")", role)
                  ).setTitle(title).build())
              .addAllMembers(ImmutableList.of(iamMember))
              .build();
      ImmutableList<Binding> bindings =
          ImmutableList.<Binding>builder()
              .addAll(policy.getBindingsList())
              .add(binding1)
              .add(binding2)
              .build();
      Policy policyWithConditions =
          Policy.newBuilder()
              .setVersion(policyVersion)
              .setEtag(policy.getEtag())
              .addAllBindings(bindings)
              .build();
      final SetIamPolicyRequest setRequest =
          SetIamPolicyRequest.newBuilder()
              .setResource(DatabaseName.of(projectId, instanceId, databaseId).toString())
              .setPolicy(policyWithConditions).build();
      final Policy response = databaseAdminClient.setIamPolicy(setRequest);
      System.out.printf(
          "Enabled fine-grained access in IAM with version %d%n", response.getVersion());
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';
// iamMember = 'user:alice@example.com';
// databaseRole = 'parent';
// title = 'condition title';
// Imports the Google Cloud Spanner client library
const {Spanner, protos} = require('@google-cloud/spanner');

// Instantiates a client
const spanner = new Spanner({
  projectId: projectId,
});

async function enableFineGrainedAccess() {
  // Gets a reference to a Cloud Spanner Database Admin Client object
  const databaseAdminClient = spanner.getDatabaseAdminClient();

  const [policy] = await databaseAdminClient.getIamPolicy({
    resource: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    options: (protos.google.iam.v1.GetPolicyOptions = {
      requestedPolicyVersion: 3,
    }),
  });
  if (policy.version < 3) {
    policy.version = 3;
  }

  const newBinding = {
    role: 'roles/spanner.fineGrainedAccessUser',
    members: [`user:${iamMember}`],
    condition: {
      title: title,
      expression: `resource.name.endsWith("/databaseRoles/${databaseRole}")`,
    },
  };
  policy.bindings.push(newBinding);
  await databaseAdminClient.setIamPolicy({
    resource: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    policy: policy,
  });
  // Requested Policy Version is Optional. The maximum policy version that will be used to format the policy.
  // Valid values are 0, 1, and 3. Requests specifying an invalid value will be rejected.
  const newPolicy = await databaseAdminClient.getIamPolicy({
    resource: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
    options: (protos.google.iam.v1.GetPolicyOptions = {
      requestedPolicyVersion: 3,
    }),
  });
  console.log(newPolicy);
}
enableFineGrainedAccess();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use \Google\Cloud\Iam\V1\Binding;
use \Google\Type\Expr;
use Google\Cloud\Iam\V1\GetIamPolicyRequest;
use Google\Cloud\Iam\V1\SetIamPolicyRequest;
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;

/**
 * Enable Fine Grained Access.
 * Example:
 * ```
 * enable_fine_grained_access($projectId, $instanceId, $databaseId, $iamMember, $databaseRole, $title);
 * ```
 *
 * @param string $projectId The Google cloud project ID
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 * @param string $iamMember The IAM member. Eg: `user:{emailid}`,
 *        `serviceAccount:{emailid}`, `group:{emailid}`, `domain:{domain}`
 * @param string $databaseRole The database role bound to
 *        the IAM member.
 * @param string $title Condition title.
 */
function enable_fine_grained_access(
    string $projectId,
    string $instanceId,
    string $databaseId,
    string $iamMember,
    string $databaseRole,
    string $title
): void {
    $adminClient = new DatabaseAdminClient();
    $resource = $adminC>lient-databaseName($projectId, $instanceId, $databaseId);
    $getIamPolicyRequest = (new GetIamPolicyRequest())
   >     -setResource($resource);
    $policy = $adminC>lient-getIamPolicy($getIamPolicyRequest);

    // IAM conditions need at least version 3
    if ($p>olicy-getVersion() != 3) {
        $p>olicy-setVersion(3);
    }

    $binding = new Binding([
        'r>ole' = 'roles/spanner.fineGrainedAccessUser',
        'memb>ers' = [$iamMember],
        'condit>ion' = new Expr([
            'ti>tle' = $title,
            'express>ion' = sprintf("resource.name.endsWith('/databaseRoles/%s')", $databaseRole)
        ])
    ]);
    $p>olicy-setBindings([$binding]);
    $setIamPolicyRequest = (new SetIamPolicyRequest())
   >     -setResource($resource)
   >     -setPolicy($policy);
    $adminC>lient-setIamPolicy($setIamPolicyRequest);

    printf('Enabled fine-grained access in IAM' . PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
# iam_member = "user:alice@example.com"
# database_role = "new_parent"
# title = "condition title"

from google.iam.v1 import iam_policy_pb2, options_pb2, policy_pb2
from google.type import expr_pb2

spanner_client = spanner.Client()
database_admin_api = spanner_client.database_admin_api

# The policy in the response from getDatabaseIAMPolicy might use the policy version
# that you specified, or it might use a lower policy version. For example, if you
# specify version 3, but the policy has no conditional role bindings, the response
# uses version 1. Valid values are 0, 1, and 3.
request = iam_policy_pb2.GetIamPolicyRequest(
    resource=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    options=options_pb2.GetPolicyOptions(requested_policy_version=3),
)
policy = database_admin_api.get_iam_policy(request=request)
if policy.version < 3:
    policy.version = 3

new_binding = policy_pb2.Binding(
    role="roles/spanner.fineGrainedAccessUser",
    members=[iam_member],
    condition=expr_pb2.Expr(
        title=title,
        expression=f'resource.name.endsWith("/databaseRoles/{database_role}")',
    ),
)

policy.version = 3
policy.bindings.append(new_binding)
set_request = iam_policy_pb2.SetIamPolicyRequest(
    resource=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    policy=policy,
)
database_admin_api.set_iam_policy(set_request)

new_policy = database_admin_api.get_iam_policy(request=request)
print(
    f"Enabled fine-grained access in IAM. New policy has version {new_policy.version}"
)
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

def spanner_enable_fine_grained_access project_id:, instance_id:, database_id:, iam_member:, database_role:, title:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"
  # iam_member = "user:alice@example.com"
  # database_role = "new_parent"
  # title = "condition title"

  admin_client = Google::Cloud::Spanner::Admin::Database::V1::DatabaseAdmin::Client.new
  db_path = admin_client.database_path project: project_id, instance: instance_id, database: database_id

  policy = admin_client.get_iam_policy resource: db_path, options: { requested_policy_version: 3 }

  policy.version = 3 if policy.version < 3

  binding = Google::Iam::V1::Binding.new(
    role: "roles/spanner.fineGrainedAccessUser",
    members: [iam_member],
    condition: Google::Type::Expr.new(
      title: title,
      expression: "resource.name.endsWith('/databaseRoles/#{database_role}')"
    )
  )

  policy.bindings << binding
  result = admin_client.set_iam_policy resource: db_path, policy: policy

  puts "Enabled fine-grained access in IAM."
end
```

## Inform users and developers to start using database roles

After initial fine-grained access control configuration is complete, inform users and application developers that they must begin using database roles.

  - Fine-grained access control users must begin specifying a database role when they access Spanner databases through the Google Cloud console or the Google Cloud CLI.

  - Applications that use fine-grained access control must specify a database role when accessing the database.

For more information, see [Access a database with fine-grained access control](/spanner/docs/access-with-fgac) .

## Transition a principal to fine-grained access control

To transition a principal from database-level access control to fine-grained access control, follow these steps:

1.  Enable fine-grained access control for the principal and grant access to all required database roles, as described in [Grant database roles to principals](#grant-fgac-role) .

2.  Update all applications that run as this principal. Specify the appropriate database roles in calls to client library methods.

3.  Revoke all IAM database-level roles from the principal. This way, access for the principal is governed by only one method.
    
    Exception: To interact with Spanner resources in the Google Cloud console, all users must have the `  roles/spanner.viewer  ` IAM role.
    
    To revoke IAM database-level roles, follow the instructions in [Remove database-level permissions](/spanner/docs/grant-permissions#remove-iam-db-role) .

## List database roles

**Note:** Spanner Studio (formerly labeled **Query** in the Google Cloud console) supports SQL, DML, and DDL operations in a single editor. For more information, see [Manage your data using the Google Cloud console](/spanner/docs/manage-data-using-console) .

You can list the database roles associated with a database.

### Console

To list database roles, enter the following query on the Spanner Studio page for the database:

### GoogleSQL

``` text
SELECT * FROM INFORMATION_SCHEMA.ROLES;
```

### PostgreSQL

``` text
SELECT * FROM information_schema.enabled_roles;
```

The response includes the current role and roles whose privileges the current role can use through inheritance. To fetch all roles, use the gcloud CLI command.

### gcloud

To obtain an unfiltered list of database roles, enter the following command. It requires the `  spanner.databaseRoles.list  ` permission.

``` text
gcloud spanner databases roles list --database=DATABASE_NAME --instance=INSTANCE_NAME
```

### Client libraries

### C++

``` cpp
void ListDatabaseRoles(google::cloud::spanner_admin::DatabaseAdminClient client,
                       std::string const& project_id,
                       std::string const& instance_id,
                       std::string const& database_id) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::cout << "Database Roles are:\n";
  for (auto& role : client.ListDatabaseRoles(database.FullName())) {
    if (!role) throw std::move(role).status();
    std::cout << role->name() << "\n";
  }
}
```

### C\#

``` csharp
using Google.Api.Gax;
using Google.Cloud.Spanner.Admin.Database.V1;
using System;

public class ListDatabaseRolesSample
{
    public PagedEnumerable<ListDatabaseRolesResponse, DatabaseRole> ListDatabaseRoles(string projectId, string instanceId, string databaseId)
    {
        string parent = $"projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        var client = DatabaseAdminClient.Create();
        PagedEnumerable<ListDatabaseRolesResponse, DatabaseRole> databaseRoles = client.ListDatabaseRoles(parent);
        foreach (var dbRole in databaseRoles)
        {
            Console.WriteLine($"Database Role: {dbRole.DatabaseRoleName}");
        }
        return databaseRoles;
    }
}
```

### Go

``` go
import (
 "context"
 "fmt"
 "io"
 "strings"

 "google.golang.org/api/iterator"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func listDatabaseRoles(w io.Writer, db string) error {
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 iter := adminClient.ListDatabaseRoles(ctx, &adminpb.ListDatabaseRolesRequest{
     Parent: db,
 })
 rolePrefix := db + "/databaseRoles/"
 for {
     role, err := iter.Next()
     if err == iterator.Done {
         break
     }
     if err != nil {
         return err
     }
     if !strings.HasPrefix(role.Name, rolePrefix) {
         return fmt.Errorf("Role %v does not have prefix %v", role.Name, rolePrefix)
     }
     fmt.Fprintf(w, "%s\n", strings.TrimPrefix(role.Name, rolePrefix))
 }
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient.ListDatabaseRolesPage;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient.ListDatabaseRolesPagedResponse;
import com.google.spanner.admin.database.v1.DatabaseName;
import com.google.spanner.admin.database.v1.DatabaseRole;

public class ListDatabaseRoles {

  static void listDatabaseRoles() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    listDatabaseRoles(projectId, instanceId, databaseId);
  }

  static void listDatabaseRoles(String projectId, String instanceId, String databaseId) {
    try (Spanner spanner =
        SpannerOptions.newBuilder().setProjectId(projectId).build().getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      DatabaseName databaseName = DatabaseName.of(projectId, instanceId, databaseId);
      ListDatabaseRolesPagedResponse response
          = databaseAdminClient.listDatabaseRoles(databaseName);
      System.out.println("List of Database roles");
      for (ListDatabaseRolesPage page : response.iteratePages()) {
        for (DatabaseRole role : page.iterateAll()) {
          System.out.printf("Obtained role %s%n", role.getName());
        }
      }
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function getDatabaseRoles() {
  // Fetching database roles
  const [databaseRoles] = await databaseAdminClient.listDatabaseRoles({
    parent: databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    ),
  });
  console.log(
    `Roles for Database: ${databaseAdminClient.databasePath(
      projectId,
      instanceId,
      databaseId,
    )}`,
  );
  databaseRoles.forEach(role => {
    console.log(`Role: ${role.name}`);
  });
}
getDatabaseRoles();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\ListDatabaseRolesRequest;

/**
 * List Database roles in the given database.
 * Example:
 * ```
 * list_database_roles($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google cloud project ID
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function list_database_roles(
    string $projectId,
    string $instanceId,
    string $databaseId
): void {
    $adminClient = new DatabaseAdminClient();
    $resource = $adminClient->databaseName($projectId, $instanceId, $databaseId);
    $listDatabaseRolesRequest = (new ListDatabaseRolesRequest())
        ->setParent($resource);

    $roles = $adminClient->listDatabaseRoles($listDatabaseRolesRequest);
    printf('List of Database roles:' . PHP_EOL);
    foreach ($roles as $role) {
        printf($role->getName() . PHP_EOL);
    }
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"
from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

spanner_client = spanner.Client()
database_admin_api = spanner_client.database_admin_api

request = spanner_database_admin.ListDatabaseRolesRequest(
    parent=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    )
)
# List database roles.
print("Database Roles are:")
for role in database_admin_api.list_database_roles(request):
    print(role.name.split("/")[-1])
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

def spanner_list_database_roles project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  admin_client = Google::Cloud::Spanner::Admin::Database::V1::DatabaseAdmin::Client.new

  db_path = admin_client.database_path project: project_id, instance: instance_id, database: database_id

  result = admin_client.list_database_roles parent: db_path

  puts "List of Database roles:"
  result.each do |role|
    puts role.name
  end
end
```

## View the privileges granted to a database role

To view the privileges granted to a role, run the following queries:

### GoogleSQL

``` text
SELECT * FROM INFORMATION_SCHEMA.TABLE_PRIVILEGES WHERE grantee = 'ROLE_NAME';
SELECT * FROM INFORMATION_SCHEMA.COLUMN_PRIVILEGES WHERE grantee = 'ROLE_NAME';
SELECT * FROM INFORMATION_SCHEMA.CHANGE_STREAM_PRIVILEGES WHERE grantee = 'ROLE_NAME';
```

`  INFORMATION_SCHEMA.TABLE_PRIVILEGES  ` returns privileges on both tables and views. The `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` privileges in `  TABLE_PRIVILEGES  ` are also shown in `  COLUMN_PRIVILEGES  ` .

**Note:** If all privileges on a table are restricted to a subset of columns, then the table isn't included in the result set of the `  TABLE_PRIVILEGES  ` query.

### PostgreSQL

``` text
SELECT * FROM information_schema.table_privileges WHERE grantee = 'ROLE_NAME';
SELECT * FROM information_schema.column_privileges WHERE grantee = 'ROLE_NAME';
SELECT * FROM information_schema.change_stream_privileges WHERE grantee = 'ROLE_NAME';
```

`  information_schema.table_privileges  ` returns privileges on both tables and views. The `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` privileges in `  table_privileges  ` are also shown in `  column_privileges  ` .

**Note:** If all privileges on a table are restricted to a subset of columns, then the table is not included in the result set of the `  table_privileges  ` query.

## View fine-grained access control users

To view a list of principals that are fine-grained access control users, run the following command. To run the command, you must have the Cloud Asset API enabled on your project, and you must have the [`  cloudasset.assets.searchAllIamPolicies  `](/asset-inventory/docs/access-control#required_permissions) IAM permission.

``` text
gcloud asset search-all-iam-policies \
--scope=projects/PROJECT_NAME \
--query='roles=roles/spanner.fineGrainedAccessUser AND resource=//spanner.googleapis.com/projects/PROJECT_NAME/instances/INSTANCE_NAME/databases/DATABASE_NAME' \
--flatten=policy.bindings[].members[] \
--format='table(policy.bindings.members)'
```

Output is similar to the following:

``` text
MEMBERS
user:222larabrown@gmail.com
user:baklavainthebalkans@gmail.com
serviceAccount:cs-fgac-sa-1@cloud-spanner-demo.google.com.iam.gserviceaccount.com
serviceAccount:cs-fgac-sa-2@cloud-spanner-demo.google.com.iam.gserviceaccount.com
```

For more information, see [Enabling an API in your Google Cloud project](/endpoints/docs/openapi/enable-api) .

## View principals with database access

To view a list of principals who have been granted access to a particular database, run the following commands. To run these commands, you must have the Cloud Asset API enabled on your project, and you must have the [`  cloudasset.assets.searchAllIamPolicies  `](/asset-inventory/docs/access-control#required_permissions) IAM permission.

``` text
gcloud asset search-all-iam-policies \
--scope=projects/PROJECT_NAME \
--query='roles=roles/spanner.databaseRoleUser AND policy:"resource.name" AND policy:/ROLE_NAME AND resource=//spanner.googleapis.com/projects/PROJECT_NAME/instances/INSTANCE_NAME/databases/DATABASE_NAME' \
--flatten=policy.bindings[].members[] \
--format='table(policy.bindings.members)'
```

Output is similar to the following:

``` text
MEMBERS
222larabrown@gmail.com
```

## View the IAM conditions for a principal

To view a list of IAM conditions that were specified when granting the role **Cloud Spanner Database Role User** to a principal, run the following command:

``` text
gcloud asset search-all-iam-policies \
--scope=projects/PROJECT_NAME \
--query='roles=roles/spanner.databaseRoleUser AND policy:resource.name AND policy:"PRINCIPAL_IDENTIFIER" AND resource=//spanner.googleapis.com/projects/PROJECT_NAME/instances/INSTANCE_NAME/databases/DATABASE_NAME' \
--flatten=policy.bindings[] \
--format='table(policy.bindings.condition.expression)'
```

where PRINCIPAL\_IDENTIFIER is:

``` text
  { user:user-account-name | serviceAccount:service-account-name }
```

PRINCIPAL\_IDENTIFIER examples:

``` text
user:222larabrown@gmail.com
serviceAccount:cs-fgac-sa-1@cloud-spanner-demo.google.com.iam.gserviceaccount.com
```

The following sample output shows two condition expressions.

``` text
EXPRESSION
resource.type == "spanner.googleapis.com/DatabaseRole" &&
resource.name.endsWith("/hr_analyst")
resource.type == "spanner.googleapis.com/DatabaseRole" &&
resource.name.endsWith("/hr_manager")
```

## Check IAM policies for missing database role conditions

After you have granted access to database roles to principals, we recommend that you ensure that each IAM binding has a condition specified.

To perform this check, run the following command:

``` text
gcloud asset search-all-iam-policies \
--scope=projects/PROJECT_NAME \
--query='roles:roles/spanner.databaseRoleUser AND resource=//spanner.googleapis.com/projects/PROJECT_NAME/instances/INSTANCE_NAME/databases/DATABASE_NAME'
--flatten=policy.bindings[].members[]
```

The output is similar to the following:

``` text
ROLE                              MEMBERS                         EXPRESSION
roles/spanner.databaseRoleUser    serviceAccount:cs-fgac-sa-1@...
roles/spanner.databaseRoleUser    serviceAccount:cs-fgac-sa-2@... resource.type == "spanner…"
```

Note that the first result is missing a condition and therefore the principals in this binding have access to all database roles.

## Drop a database role

Dropping a database role automatically revokes the membership of other roles from the role, and revokes the role's membership in other roles.

To drop a database role, you must first do the following:

1.  Revoke all fine-grained access control privileges from the role.
2.  Remove any IAM policy bindings that refer to that role, so that a database role created later with the same name does not inherit those bindings.

### Console

To drop a database role, follow these steps:

1.  On the database **Overview** page, click **Spanner Studio** .

2.  To revoke privileges from the role, enter a `  REVOKE  ` statement.
    
    ### GoogleSQL
    
    For syntax details for the `  REVOKE  ` statement, see [GoogleSQL data definition language](/spanner/docs/reference/standard-sql/data-definition-language#revoke_statement) . For information about privileges, see [Fine-grained access control privileges](/spanner/docs/fgac-privileges) .
    
    For example, to revoke `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` permissions on tables `  employees  ` and `  contractors  ` from the database role `  hr_manager  ` , enter the following statement:
    
    ``` text
    REVOKE SELECT, INSERT, UPDATE ON TABLE employees, contractors FROM ROLE hr_manager;
    ```
    
    ### PostgreSQL
    
    For syntax details for the `  REVOKE  ` statement, see [PostgreSQL data definition language](/spanner/docs/reference/postgresql/data-definition-language#revoke_statement) . For information about privileges, see [Fine-grained access control privileges](/spanner/docs/fgac-privileges) .
    
    For example, to revoke the `  SELECT  ` , `  INSERT  ` , and `  UPDATE  ` privileges on the `  employees  ` and `  contractors  ` tables from the database role `  hr_manager  ` , enter the following statement:
    
    ``` text
    REVOKE SELECT, INSERT, UPDATE ON TABLE employees, contractors FROM hr_manager;
    ```
    
    You can use a DDL template for the `  REVOKE  ` statement. In the **Explorer** pane, navigate to the role for which you want to revoke privilege. Click more\_vert **View actions** and select the type of privilege you want to revoke access for this role. The `  REVOKE  ` template statement populates in a new editor tab.

3.  Delete any IAM conditions associated with the role.
    
    1.  In the list of roles on the **Info panel** , locate the **Cloud Spanner Database Role User** role that has the condition title of interest adjacent to it, and then expand the role to view the principals that have access to the role.
    
    2.  For one of the principals, click the **Edit principal** (pencil) icon.
    
    3.  On the **Edit access** page, click the **Delete role** (trash can) icon adjacent to the Cloud Spanner Database Role User role.
    
    4.  Click **Save** .
    
    5.  Repeat the previous three steps for other principals who are listed under the condition.

4.  To drop the role, go to the **Spanner Studio** page and enter the following statement:
    
    ``` text
    DROP ROLE ROLE_NAME;
    ```

5.  Click **Submit** .

### gcloud

1.  To revoke all privileges for a role and then drop the role, use the [`  gcloud spanner databases ddl update  `](/sdk/gcloud/reference/spanner/databases/ddl/update) command as follows:
    
    ### GoogleSQL
    
    ``` text
    gcloud spanner databases ddl update DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --ddl='REVOKE PERMISSIONS ON TABLE TABLE_NAME FROM ROLE ROLE_NAME; DROP ROLE ROLE_NAME;'
    ```
    
    ### PostgreSQL
    
    ``` text
    gcloud spanner databases ddl update DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --ddl='REVOKE PERMISSIONS ON TABLE TABLE_NAME FROM ROLE_NAME; DROP ROLE ROLE_NAME;'
    ```
    
    Valid values for `  PERMISSIONS  ` are `  SELECT  ` , `  INSERT  ` , `  UPDATE  ` , and `  DELETE  ` .

2.  To delete any related IAM conditions, use the [`  gcloud spanner databases remove-iam-policy-binding  `](/sdk/gcloud/reference/spanner/databases/remove-iam-policy-binding) command as follows:
    
    ``` text
    gcloud spanner databases remove-iam-policy-binding DATABASE_NAME \
    --instance=INSTANCE_NAME \
    --role=ROLE_NAME \
    --member=MEMBER_NAME \
    --condition=CONDITION
    ```
    
      - `  MEMBER_NAME  ` is the identifier for the principal. It must be of the form `  user|group|serviceAccount: email  ` or `  domain: domain  ` .
    
      - `  CONDITION  ` is an IAM condition expression that specifies the roles to grant to the principal.
        
        `  CONDITION  ` has the following form:
        
        ``` text
        --condition='expression=(resource.type == "spanner.googleapis.com/DatabaseRole" && (resource.name.endsWith("/ROLE1") || resource.name.endsWith("/ROLE2"))),title=TITLE,description=DESCRIPTION'
        ```
        
        The entire condition specification must exactly match the condition specification that was used in the command that granted permission, including title and description.

### Client libraries

These code samples both create and drop a database role.

### C++

``` cpp
void AddAndDropDatabaseRole(
    google::cloud::spanner_admin::DatabaseAdminClient client,
    std::string const& project_id, std::string const& instance_id,
    std::string const& database_id, std::string const& role_parent,
    std::string const& role_child) {
  google::cloud::spanner::Database database(project_id, instance_id,
                                            database_id);
  std::vector<std::string> grant_statements = {
      "CREATE ROLE " + role_parent,
      "GRANT SELECT ON TABLE Singers TO ROLE " + role_parent,
      "CREATE ROLE " + role_child,
      "GRANT ROLE " + role_parent + " TO ROLE " + role_child,
  };
  auto metadata =
      client.UpdateDatabaseDdl(database.FullName(), grant_statements).get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Created roles " << role_parent << " and " << role_child
            << " and granted privileges\n";

  std::vector<std::string> revoke_statements = {
      "REVOKE ROLE " + role_parent + " FROM ROLE " + role_child,
      "DROP ROLE " + role_child,
  };
  metadata =
      client.UpdateDatabaseDdl(database.FullName(), revoke_statements).get();
  google::cloud::spanner_testing::LogUpdateDatabaseDdl(  //! TODO(#4758)
      client, database, metadata.status());              //! TODO(#4758)
  if (!metadata) throw std::move(metadata).status();
  std::cout << "Revoked privileges and dropped role " << role_child << "\n";
}
```

### C\#

``` csharp
using Google.Cloud.Spanner.Data;
using System.Threading.Tasks;

public class AddAndDropDatabaseRoleAsyncSample
{
    public async Task AddDatabaseRoleAsync(string projectId, string instanceId, string databaseId, string databaseRole)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string createRoleStatement = $"CREATE ROLE {databaseRole}";

        // Creates the given database role.
        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(createRoleStatement);
        await updateCmd.ExecuteNonQueryAsync();
    }

    public async Task DropDatabaseRoleAsync(string projectId, string instanceId, string databaseId, string databaseRole)
    {
        string connectionString = $"Data Source=projects/{projectId}/instances/{instanceId}/databases/{databaseId}";
        string deleteRoleStatement = $"DROP ROLE {databaseRole}";

        // Drops the given database role.
        using var connection = new SpannerConnection(connectionString);
        using var updateCmd = connection.CreateDdlCommand(deleteRoleStatement);
        await updateCmd.ExecuteNonQueryAsync();
    }
}
```

### Go

``` go
import (
 "context"
 "io"

 database "cloud.google.com/go/spanner/admin/database/apiv1"
 adminpb "cloud.google.com/go/spanner/admin/database/apiv1/databasepb"
)

func addAndDropDatabaseRole(w io.Writer, db string) error {
 ctx := context.Background()
 adminClient, err := database.NewDatabaseAdminClient(ctx)
 if err != nil {
     return err
 }
 defer adminClient.Close()

 // Set up database roles and membership. After database roles are created,
 // users can be granted roles by setting IAM policies.
 op, err := adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "CREATE ROLE parent",
         "GRANT SELECT ON TABLE Albums TO ROLE parent",
         "CREATE ROLE child",
         "GRANT ROLE parent TO ROLE child",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }

 // Delete role and membership.
 op, err = adminClient.UpdateDatabaseDdl(ctx, &adminpb.UpdateDatabaseDdlRequest{
     Database: db,
     Statements: []string{
         "REVOKE ROLE parent FROM ROLE child",
         "DROP ROLE child",
     },
 })
 if err != nil {
     return err
 }
 if err := op.Wait(ctx); err != nil {
     return err
 }
 return nil
}
```

### Java

``` java
import com.google.cloud.spanner.Spanner;
import com.google.cloud.spanner.SpannerOptions;
import com.google.cloud.spanner.admin.database.v1.DatabaseAdminClient;
import com.google.common.collect.ImmutableList;
import com.google.spanner.admin.database.v1.DatabaseName;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;

public class AddAndDropDatabaseRole {

  static void addAndDropDatabaseRole() {
    // TODO(developer): Replace these variables before running the sample.
    String projectId = "my-project";
    String instanceId = "my-instance";
    String databaseId = "my-database";
    String parentRole = "parent_role";
    String childRole = "child_role";
    addAndDropDatabaseRole(projectId, instanceId, databaseId, parentRole, childRole, "Albums");
  }

  static void addAndDropDatabaseRole(
      String projectId, String instanceId, String databaseId,
      String parentRole, String childRole, String... tables) {
    try (Spanner spanner =
        SpannerOptions.newBuilder()
            .setProjectId(projectId)
            .build()
            .getService();
        DatabaseAdminClient databaseAdminClient = spanner.createDatabaseAdminClient()) {
      System.out.println("Waiting for role create operation to complete...");
      List<String> roleStatements = new ArrayList<>(ImmutableList.of(
          String.format("CREATE ROLE %s", parentRole),
          String.format("CREATE ROLE %s", childRole),
          String.format("GRANT ROLE %s TO ROLE %s", parentRole, childRole)));
      for (String table : tables) {
        roleStatements.add(String.format("GRANT SELECT ON TABLE %s TO ROLE %s", table, parentRole));
      }
      databaseAdminClient.updateDatabaseDdlAsync(
              DatabaseName.of(projectId, instanceId, databaseId), roleStatements)
          .get(5, TimeUnit.MINUTES);
      System.out.printf(
          "Created roles %s and %s and granted privileges%n", parentRole, childRole);
      // Delete role and membership.
      System.out.println("Waiting for role revoke & drop operation to complete...");
      databaseAdminClient.updateDatabaseDdlAsync(
          DatabaseName.of(projectId, instanceId, databaseId),
          ImmutableList.of(
              String.format("REVOKE ROLE %s FROM ROLE %s", parentRole, childRole),
              String.format("DROP ROLE %s", childRole))).get(5, TimeUnit.MINUTES);
      System.out.printf("Revoked privileges and dropped role %s%n", childRole);
    } catch (ExecutionException | TimeoutException e) {
      System.out.printf(
          "Error: AddAndDropDatabaseRole failed with error message %s\n", e.getMessage());
      e.printStackTrace();
    } catch (InterruptedException e) {
      System.out.println(
          "Error: Waiting for AddAndDropDatabaseRole operation to finish was interrupted");
    }
  }
}
```

**Note:** The old client library interface code samples for Java are archived in [GitHub](https://github.com/googleapis/java-spanner/tree/main/samples/snippets/src/main/java/com/example/spanner/admin/archived) .

### Node.js

``` javascript
/**
 * TODO(developer): Uncomment these variables before running the sample.
 */
// const instanceId = 'my-instance';
// const databaseId = 'my-database';
// const projectId = 'my-project-id';

// Imports the Google Cloud client library
const {Spanner} = require('@google-cloud/spanner');

// creates a client
const spanner = new Spanner({
  projectId: projectId,
});

const databaseAdminClient = spanner.getDatabaseAdminClient();

async function addAndDropNewDatabaseRole() {
  // Creates a new user defined role and grant permissions
  try {
    const request = [
      'CREATE ROLE parent',
      'GRANT SELECT ON TABLE Singers TO ROLE parent',
      'CREATE ROLE child',
      'GRANT ROLE parent TO ROLE child',
    ];
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log('Created roles child and parent and granted privileges');
  } catch (err) {
    console.error('ERROR:', err);
  }

  // Revoke permissions and drop child role.
  // A role can't be dropped until all its permissions are revoked.
  try {
    const request = ['REVOKE ROLE parent FROM ROLE child', 'DROP ROLE child'];
    const [operation] = await databaseAdminClient.updateDatabaseDdl({
      database: databaseAdminClient.databasePath(
        projectId,
        instanceId,
        databaseId,
      ),
      statements: request,
    });

    console.log('Waiting for operation to complete...');
    await operation.promise();

    console.log('Revoked privileges and dropped role child');
  } catch (err) {
    console.error('ERROR:', err);
  } finally {
    // Close the spanner client when finished.
    // The databaseAdminClient does not require explicit closure. The closure of the Spanner client will automatically close the databaseAdminClient.
    spanner.close();
  }
}
addAndDropNewDatabaseRole();
```

**Note:** The old client library interface code samples for Node.js are archived in [GitHub](https://github.com/googleapis/nodejs-spanner/tree/main/samples/archived) .

### PHP

```` php
use Google\Cloud\Spanner\Admin\Database\V1\Client\DatabaseAdminClient;
use Google\Cloud\Spanner\Admin\Database\V1\UpdateDatabaseDdlRequest;

/**
 * Adds and drops roles to the Singers table in the example database.
 * Example:
 * ```
 * add_drop_database_role($projectId, $instanceId, $databaseId);
 * ```
 *
 * @param string $projectId The Google Cloud project ID.
 * @param string $instanceId The Spanner instance ID.
 * @param string $databaseId The Spanner database ID.
 */
function add_drop_database_role(string $projectId, string $instanceId, string $databaseId): void
{
    $databaseAdminClient = new DatabaseAdminClient();
    $databaseName = DatabaseAdminClient::databaseName($projectId, $instanceId, $databaseId);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            'CREATE ROLE new_parent',
            'GRANT SELECT ON TABLE Singers TO ROLE new_parent',
            'CREATE ROLE new_child',
            'GRANT ROLE new_parent TO ROLE new_child'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    printf('Waiting for create role and grant operation to complete...%s', PHP_EOL);
    $operation->pollUntilComplete();

    printf('Created roles %s and %s and granted privileges%s', 'new_parent', 'new_child', PHP_EOL);

    $request = new UpdateDatabaseDdlRequest([
        'database' => $databaseName,
        'statements' => [
            'REVOKE ROLE new_parent FROM ROLE new_child',
            'DROP ROLE new_child'
        ]
    ]);

    $operation = $databaseAdminClient->updateDatabaseDdl($request);

    printf('Waiting for revoke role and drop role operation to complete...%s', PHP_EOL);
    $operation->pollUntilComplete();

    printf('Revoked privileges and dropped role %s%s', 'new_child', PHP_EOL);
}
````

**Note:** The old client library interface code samples for PHP are archived in [GitHub](https://github.com/GoogleCloudPlatform/php-docs-samples/tree/main/spanner/src/admin/archived) .

### Python

``` python
# instance_id = "your-spanner-instance"
# database_id = "your-spanner-db-id"

from google.cloud.spanner_admin_database_v1.types import spanner_database_admin

spanner_client = spanner.Client()
database_admin_api = spanner_client.database_admin_api

role_parent = "new_parent"
role_child = "new_child"

request = spanner_database_admin.UpdateDatabaseDdlRequest(
    database=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    statements=[
        "CREATE ROLE {}".format(role_parent),
        "GRANT SELECT ON TABLE Singers TO ROLE {}".format(role_parent),
        "CREATE ROLE {}".format(role_child),
        "GRANT ROLE {} TO ROLE {}".format(role_parent, role_child),
    ],
)
operation = database_admin_api.update_database_ddl(request)

operation.result(OPERATION_TIMEOUT_SECONDS)
print(
    "Created roles {} and {} and granted privileges".format(role_parent, role_child)
)

request = spanner_database_admin.UpdateDatabaseDdlRequest(
    database=database_admin_api.database_path(
        spanner_client.project, instance_id, database_id
    ),
    statements=[
        "REVOKE ROLE {} FROM ROLE {}".format(role_parent, role_child),
        "DROP ROLE {}".format(role_child),
    ],
)
operation = database_admin_api.update_database_ddl(request)

operation.result(OPERATION_TIMEOUT_SECONDS)
print("Revoked privileges and dropped role {}".format(role_child))
```

**Note:** The old client library interface code samples for Python are archived in [GitHub](https://github.com/googleapis/python-spanner/tree/main/samples/samples/archived) .

### Ruby

``` ruby
require "google/cloud/spanner"

def spanner_add_and_drop_database_role project_id:, instance_id:, database_id:
  # project_id  = "Your Google Cloud project ID"
  # instance_id = "Your Spanner instance ID"
  # database_id = "Your Spanner database ID"

  admin_client = Google::Cloud::Spanner::Admin::Database::V1::DatabaseAdmin::Client.new
  role_parent = "new_parent"
  role_child = "new_child"

  db_path = admin_client.database_path project: project_id, instance: instance_id, database: database_id

  job = admin_client.update_database_ddl database: db_path, statements: [
    "CREATE ROLE #{role_parent}",
    "GRANT SELECT ON TABLE Singers TO ROLE #{role_parent}",
    "CREATE ROLE #{role_child}",
    "GRANT ROLE #{role_parent} TO ROLE #{role_child}"
  ]

  job.wait_until_done!
  puts "Created roles #{role_parent} and #{role_child} and granted privileges"


  job = admin_client.update_database_ddl database: db_path, statements: [
    "REVOKE ROLE #{role_parent} FROM ROLE #{role_child}",
    "DROP ROLE #{role_child}"
  ]

  job.wait_until_done!
  puts "Revoked privileges and dropped role #{role_child}"
end
```

## More information

  - [About fine-grained access control](/spanner/docs/fgac-about)
  - [Fine-grained access control for change streams](/spanner/docs/fgac-change-streams)
  - [Fine-grained access control privileges](/spanner/docs/fgac-privileges)
  - [GoogleSQL DDL reference](/spanner/docs/reference/standard-sql/data-definition-language)
  - [Make schema updates](/spanner/docs/schema-updates)
