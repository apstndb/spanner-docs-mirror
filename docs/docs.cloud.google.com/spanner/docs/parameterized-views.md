---
name: documents/docs.cloud.google.com/spanner/docs/parameterized-views
uri: https://docs.cloud.google.com/spanner/docs/parameterized-views
title: Parameterized views overview
description: Overview of parameterized views in Spanner, which dynamically filter data based on session context parameters, especially for LLM and agentic systems.
data_source: docs.cloud.google.com
---

This document describes parameterized views in Spanner. Parameterized views let you dynamically filter data based on session parameters passed with queries, especially for agentic applications and systems that use large language models (LLMs).

Parameterized views extend the capabilities of database [views](https://docs.cloud.google.com/spanner/docs/views) by injecting context parameters directly into a view definition. Parameterized views help you do the following:

- Pass query-time context parameters, such as user or tenant identity, into a view definition.
- Restrict query results to only the rows matching the provided parameters.
- Prevent users from bypassing application-level data filters in generated or ad hoc SQL queries.

For more information, see [Create a parameterized view](https://docs.cloud.google.com/spanner/docs/create-manage-views#create-parameterized-view) .

## How parameterized views work

Parameterized views use *parameters* to safely pass application-level context, such as a user ID, to the database. When your application executes a query, it sends the context with the query statement. Spanner accesses this context during execution, but neither the query nor the application user can read or modify it.

The `SECURE_CONTEXT()` function provides the SQL interface for accessing these secure parameters within a view definition. Parameterized views require the [definer's rights view](https://docs.cloud.google.com/spanner/docs/views#definer) security model, where the view uses `SECURE_CONTEXT()` to access values and filter results.

For example, to filter data based on the ID of the user who makes the query, you can include the following in your view's `WHERE` clause:

```
CREATE VIEW MyUserData
SQL SECURITY DEFINER AS
(SELECT *
FROM AllUserData
WHERE UserID = CAST(SECURE_CONTEXT('user_id'), INT));
```

This lets the application see their data, but no one else's data. You can also create a view that lets moderators view less sensitive data from any user:

```
CREATE VIEW SafeUserData
SQL SECURITY DEFINER AS
(SELECT UserID, AccountCreationTime, PublicProfileText
FROM AllUserData
WHERE SECURE_CONTEXT('role') = 'Moderator');
```

Although you can use `SECURE_CONTEXT()` anywhere in a view query definition, the best practice for performance is to use it in the `WHERE` clause to filter on the leading key column(s) of a table or index. This ensures that Spanner efficiently pushes the filter logic to the storage layer.

By default, querying the `SECURE_CONTEXT()` function directly in an ad hoc query fails unless the database role has the `spanner_secure_context_reader` role. This prevents unprivileged users from directly inspecting parameter values.

A recommended pattern is to combine parameterized views with [definer's rights views](https://docs.cloud.google.com/spanner/docs/views#definer) and [fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) (FGAC). To ensure users can only query data through the view:

1.  Enable FGAC and create a database role for your application.
2.  `GRANT` the `SELECT` privilege on the view to the role.
3.  Ensure that the [`public`](https://docs.cloud.google.com/spanner/docs/fgac-system-roles#public_system_role) system role has no privileges granted on the base tables or views.

For example:

```
-- Create a role for the application user
CREATE ROLE pv_user;

-- Grant SELECT permission on the view to the role
GRANT SELECT ON VIEW MyUserData TO ROLE pv_user;
```

After you create the parameterized view, the process for a typical request is as follows:

1.  Your application sets the secure context on a query request. The context contains information that identifies the application user, such as a user ID or role.
2.  The application executes a query against a definer's rights view.
3.  The view uses the `SECURE_CONTEXT()` function in a `WHERE` clause to filter rows based on the values provided in the secure context.
4.  Spanner returns only the rows that match the filter, effectively filtering query results to the data associated with the user's session context.

For example, an application can pass the current user's ID in the secure context using one of the Spanner client libraries. The following example uses the C++ client library to set the `user_id` secure parameter:

```
(*request.mutable_secure_context())["user_id"] = "ca6095c4-1a68-44a8-9562-dd9e897326f0";
```

The application then executes a standard SQL query against the view. The view uses this `user_id` in its `WHERE` clause to return only the records that belong to that user:

```
--Query executed by the application
SELECT * FROM MyUserData;
```

## Benefits of parameterized views

Parameters help you achieve the following goals:

- **Isolate tenant data in multi-tenant applications** : Use parameters to ensure queries return data only for a specific tenant.

- **Constrain agentic AI queries** : Restrict natural language queries to a user's or tenant's data.

- **Filter data with ephemeral credentials** : Restrict data access based on a user or tenant ID that doesn't exist in Identity and Access Management (IAM).

- **Use private credentials for filtering** : Filter data using values required for the query that shouldn't be returned to the end user.

### Security risk mitigation

Parameterized views help mitigate security risks when end users run untrusted queries — like natural language queries — on your database. These risks include the following:

- **Malicious prompts** : Users might try to manipulate the underlying model to access all application data.
- **Broadly-scoped SQL queries** : Large language models (LLMs) might generate SQL queries that expose sensitive data, even from well-intentioned user queries.

By using parameterized views, you can restrict the range of rows returned to individual application users, regardless of how users phrase their queries.

## Best practices

Follow these practices to effectively implement parameterized views.

- **Combine with definer's rights views and FGAC** : Use parameters as part of a comprehensive security model that includes definer's rights views and fine-grained access control roles. Grant the role `SELECT` privileges only on the views, and ensure the role and the `public` system role have no privileges on the underlying base tables.

- **Use JSON for complex context** : To pass structured context, such as a user with multiple roles, serialize the data as a JSON string. Pass the JSON string as a single parameter. Use Spanner's built-in [JSON](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions) or [JSONB](https://docs.cloud.google.com/spanner/docs/reference/postgresql/functions#jsonb) to extract values inside the view.

- **Simplify access with SQL UDFs** : If you frequently access fields within a structured JSON context, create user-defined SQL functions (UDFs) to encapsulate the extraction logic. Using UDFs in your views improves readability and maintainability.

- **Protect against prompt injection with Model Armor** : When generating queries using LLMs with untrusted user input, use [Model Armor](https://cloud.google.com/security/products/model-armor) to proactively identify and block prompt injection and jailbreaking attempts.

## Limitations

Parameterized views have the following limitations:

- **Argument types** : The `SECURE_CONTEXT()` function requires a string literal argument.
- **Result types** : The `SECURE_CONTEXT()` function returns a string.
- **Spanner feature interactions** : You cannot use `SECURE_CONTEXT()` in generated columns, default column value expressions, or inside user-defined functions (UDFs).
- **Security barriers** : Parameterized views in Spanner don't include a security barrier. While all filters are applied before returning query results, they might not restrict what data is accessed during internal query processing.

## What's next

- Learn how to [Create a parameterized view](https://docs.cloud.google.com/spanner/docs/create-manage-views#create-parameterized-view) .
- Learn how to [Use parameterized views](https://docs.cloud.google.com/spanner/docs/use-parameterized-views) .
- Learn more about [definer's rights views](https://docs.cloud.google.com/spanner/docs/views#definer) .
- Learn more about [fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) .
