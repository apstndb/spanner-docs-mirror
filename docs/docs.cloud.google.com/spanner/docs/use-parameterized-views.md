---
name: documents/docs.cloud.google.com/spanner/docs/use-parameterized-views
uri: https://docs.cloud.google.com/spanner/docs/use-parameterized-views
title: Use parameterized views
description: Tutorial on how to use Spanner parameterized views to restrict application data access using secure context parameters.
data_source: docs.cloud.google.com
---

This tutorial shows how to use parameterized views in Spanner to restrict access to data based on parameters that an application passes securely.

For more information about parameterized views, see [Parameterized views overview](https://docs.cloud.google.com/spanner/docs/parameterized-views) .

## Objectives

- Create a table with sample data.
- Create a parameterized view that filters data based on a secure context parameter.
- Create a database role for fine-grained access control that the application uses to connect to the database and access the parameterized view.
- Grant the new role permissions to query the parameterized view without granting it access to the base table.
- Query the parameterized view from an application by passing a parameter in the secure context.

## Before you begin

You need a Spanner instance and database to complete this tutorial. If you don't have one, see [Create and query a database using the Google Cloud console](https://docs.cloud.google.com/spanner/docs/create-query-database-console) to create an instance and a Spanner database.

## Prepare sample data

1.  In the Google Cloud console, go to the **Spanner** page.

2.  Click the name of the instance that you want to use, then select the database to use for this tutorial.

3.  On the **Database overview** page, in the navigation menu, click **Spanner Studio** .

4.  Enter the following DDL statement to create the `checked_items` table:

    ```
    CREATE TABLE checked_items(
      bag_id INT64,
      item_timestamp TIMESTAMP,
      loc_code STRING(3),
      scan_type STRING(1),
      location STRING(MAX),
      customer_id INT64
    ) PRIMARY KEY(bag_id);
    ```

5.  Click **Run** .

6.  After Spanner creates the table, enter the following statement to insert sample data into the table:

    ```
    INSERT INTO checked_items (bag_id, item_timestamp, loc_code, scan_type, location, customer_id) VALUES
    (101, '2023-10-26 10:00:00', 'ABC', 'I', 'Warehouse A', 123),
    (102, '2023-10-26 10:15:30', 'DEF', 'O', 'Loading Dock B', 456),
    (103, '2023-10-26 10:30:45', 'GHI', 'I', 'Conveyor Belt 1', 789),
    (104, '2023-10-26 11:00:00', 'JKL', 'O', 'Shipping Area C', 101),
    (105, '2023-10-26 11:45:15', 'MNO', 'I', 'Sorting Station D', 202),
    (106, '2023-10-26 12:00:00', 'PQR', 'O', 'Truck Bay E', 303);
    ```

7.  Click **Run** .

## Create a parameterized view

Create a view with `SQL SECURITY DEFINER` that filters rows from `checked_items` based on the value of `app_end_userid` provided by `SECURE_CONTEXT()` . This function retrieves parameter values that are securely passed from the application.

1.  In the query editor, enter the following statement to create the `secure_checked_items` view:

    ```
    CREATE VIEW secure_checked_items
    SQL SECURITY DEFINER AS
    (SELECT bag_id, item_timestamp, location
    FROM checked_items t
    WHERE customer_id = CAST(SECURE_CONTEXT('app_end_userid'), INT64));
    ```

2.  Click **Run** .

## Set up access privileges

To ensure that users can only access data through the parameterized view, create a [fine-grained access control role](https://docs.cloud.google.com/spanner/docs/fgac-about) and grant that role `SELECT` permission on the view. Ensure that neither the role nor the [`public`](https://docs.cloud.google.com/spanner/docs/fgac-system-roles#public_system_role) system role has privileges on the underlying base table.

1.  In the query editor, enter the following statement to create the `pv_user` role:

    ```
    CREATE ROLE pv_user;
    ```

2.  Grant access to the view:

    ```
    GRANT SELECT ON VIEW secure_checked_items TO ROLE pv_user;
    ```

3.  Click **Run** .

> **Note:** Application users can query definer's rights views that call the `SECURE_CONTEXT()` function without needing explicit permissions on the function itself.

However, if you need to call `SECURE_CONTEXT()` directly to debug your queries, grant yourself the `spanner_secure_context_reader` role. You must not grant this role to untrusted agents.

## Query data using the parameterized view

Your application queries a parameterized view by first setting parameters in the secure context of the query request, and then executing a standard `SELECT` query against the view. The view uses the `SECURE_CONTEXT()` function to filter results based on parameter values.

The following code samples demonstrate how to set the `app_end_userid` secure parameter to `303` and query the `secure_checked_items` view:

### C#

```csharp
static async Task QueryWithUserId(SpannerConnection connection, string userId)
{
    string query = "SELECT * FROM myuserdata";
    using (var command = connection.CreateSelectCommand(query))
    {
        command.ClientContext = new Google.Cloud.Spanner.Data.ClientContext {
            ClientContextModifier = (v1Context) => {
                v1Context.SecureContext["user_id"] = Value.ForString(userId);
            }
        };
        using (var reader = await command.ExecuteReaderAsync())
        {
            int rowCount = 0;
            while (await reader.ReadAsync())
            {
                Console.WriteLine($"UserID: {reader["userid"]}, UserName: {reader["username"]}, SecretData: {reader["secretdata"]}");
                rowCount++;
            }
            if (rowCount != 1) throw new Exception($"Expected 1 row, got {rowCount}");
        }
    }
}
```

### Go

```go
func queryWithUserID(ctx context.Context, client *spanner.Client, userID string) error {
    stmt := spanner.Statement{SQL: "SELECT * FROM myuserdata"}
    iter := client.Single().QueryWithOptions(ctx, stmt, spanner.QueryOptions{
        ClientContext: &spannerpb.RequestOptions_ClientContext{
            SecureContext: map[string]*structpb.Value{"user_id": structpb.NewStringValue(userID)},
        },
    })

    rowCount := 0
    err := iter.Do(func(r *spanner.Row) error {
        var id int64
        var name, secret string
        if err := r.Columns(&id, &name, &secret); err != nil {
            return err
        }
        if fmt.Sprintf("%d", id) != userID {
            return fmt.Errorf("unexpected UserID %d for user %s", id, userID)
        }
        fmt.Printf("UserID: %d, UserName: %s, SecretData: %s\n", id, name, secret)
        rowCount++
        return nil
    })
    if err != nil {
        return err
    }
    if rowCount != 1 {
        return fmt.Errorf("expected 1 row, got %d", rowCount)
    }
    return nil
}
```

### Java

```java
static void queryWithUserId(DatabaseClient dbClient, String userId) {
  RequestOptions.ClientContext clientContext =
      RequestOptions.ClientContext.newBuilder()
          .putSecureContext("user_id", Value.newBuilder().setStringValue(userId).build())
          .build();

  try (ResultSet resultSet =
      dbClient
          .singleUse()
          .executeQuery(
              Statement.of("SELECT * FROM MyUserData"), Options.clientContext(clientContext))) {
    int rowCount = 0;
    while (resultSet.next()) {
      long id = resultSet.getLong("UserID");
      if (!String.valueOf(id).equals(userId)) {
        throw new RuntimeException("Unexpected UserID " + id + " for user " + userId);
      }
      System.out.printf(
          "UserID: %d, UserName: %s, SecretData: %s%n",
          id, resultSet.getString("UserName"), resultSet.getString("SecretData"));
      rowCount++;
    }
    if (rowCount != 1) {
      throw new RuntimeException("Expected 1 row, got " + rowCount);
    }
  }
}
```

### Python

```python
def query_with_user_id(database, user_id: str):
    """Queries a parameterized view using secure context parameters."""
    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            "SELECT * FROM MyUserData",
            request_options={
                "client_context": {
                    "secure_context": {"user_id": user_id}
                }
            },
        )
        for row in results:
            print(f"UserID: {row[0]}, UserName: {row[1]}, SecretData: {row[2]}")
```

This query returns only rows from `checked_items` where `customer_id` is `303` :

```
+--------+---------------------------+-------------+
| bag_id | item_timestamp            | location    |
+--------+---------------------------+-------------+
|    106 | 2023-10-26 12:00:00+00:00 | Truck Bay E |
+--------+---------------------------+-------------+
1 row.
```

If your application connects to Spanner using a principal that has been granted the `pv_user` role, any attempt to query the `checked_items` table directly results in a permission denied error.

## What's next

- Learn more about [Parameterized views overview](https://docs.cloud.google.com/spanner/docs/parameterized-views) .
- Learn how to [Create and manage views](https://docs.cloud.google.com/spanner/docs/create-manage-views) .
- Learn more about [Fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) .
