**PostgreSQL interface note:** Views are supported in PostgreSQL-dialect databases with some difference from open source PostgreSQL. In Spanner, views are read-only. Views can't be the target of an `  INSERT  ` , `  UPDATE  ` , or `  DELETE  ` statement.

This document introduces and describes Spanner views.

## Overview

A *view* is a virtual table defined by a SQL query. When you create a view, you specify the SQL query it represents. Once you have created a view, you can execute queries that refer to the view as though it were a table.

When a query that refers to a view is executed, Spanner creates the virtual table by executing the query defined in the view, and that virtual table's content is used by the referring query.

Because the query defining a view is executed every time a query referring to the view is executed, views are sometimes called logical views or dynamic views to distinguish them from SQL materialized views, which store the results of the query defining the view as an actual table in data storage.

In Spanner, you can create a view as either an *invoker's rights view* or a *definer's rights view* . They are the two types of security models controlling access to a view for users.

Invoker's rights views

Definer's rights views

Description

If you create a view with invoker's rights, a database role needs privileges on the view and all the schema objects that the view references to query the view. For more information, see [Invoker's rights views](#invoker) .

If you create a view with definer's rights, a database role needs privileges on the view (and only the view) to query the view. Use fine-grained access control alongside definer's rights view, otherwise the definer's rights view doesn't add any additional access control. For more information, see [Definer's rights views](#definer) .

Permissions required to create the view

To create, grant, and revoke access to either view types, you must have database-level `  spanner.database.updateDdl  ` permission.

Privileges required to query the view

A database role needs privileges to the view and all its underlying schema objects to query the view.

A database role needs privileges to the view (and only the view) to query the view.

## Benefits of views

Views offer several benefits over including the queries they define in the application logic.

  - **Views can provide logical data-modeling to applications.**
    
    Sometimes the choices that make sense for physical data-modeling on Spanner are not the best abstraction for applications reading that data. A view can present an alternate table schema that is a more appropriate abstraction for applications.

  - **Views centralize query definitions and so simplify maintenance.**
    
    By creating views for widely used or complex queries, you can factor query text out of applications and centralize it. Doing so makes keeping query text up-to-date across applications much simpler and permits revision and tuning of queries without requiring application code to change.

  - **Views provide stability across schema changes.**
    
    Because the query that defines a view is stored in the database schema instead of in application logic, Spanner can and does ensure that schema changes to the objects (tables, columns and so on) the query refers to do not invalidate the query.

## Common use cases

Use views when your Spanner database includes highly privileged data that shouldn't be exposed to all database users or if you want to encapsulate your data.

If your view doesn't need additional security functionality and all invokers of the view have access to all schema objects that the view references, create an invoker's rights view.

If you want to create a view where not all invokers have access to all schema objects that the view references, create a definer's rights view. Definer's rights views are better protected and have more restrictions because the database admin can provide fewer users with privileges on the tables and columns referenced in the view. Definer's rights views are useful when a user needs a way to securely access a relevant subset of a Spanner database. For example, you might want to create a definer's rights view for the following data:

  - Personal account data (e.g., application customer).
  - Role specific data (e.g., HR personnel, sales associate).
  - Location specific data.

## Invoker's rights views

If a view has invoker's rights, it means that when a user, the invoker, executes a query against the view, Spanner checks the user's privileges on the view and on all the schema objects that the view references. The user must have privileges on all schema objects to query the view.

## Definer's rights views

A definer's rights view adds additional security functionality to the view. It provides different privileges on the view and the underlying schema objects. Like for invoker's rights views, users must have database-level permissions to create definer's rights views. The main difference is that when a database role queries a definer's rights view, Spanner verifies that the role has access to the view itself (and only the view). Therefore, even if the user who queries the view doesn't have access to all the underlying schema objects, they can access the view and see its contents. Definer's rights views give users access to fresh data, limited to the rows defined in the view.

Spanner Identity and Access Management (IAM) permissions are granted at the database level. Use [fine-grained access control](/spanner/docs/fgac-about) alongside definer's rights view, otherwise the definer's rights view doesn't add any additional access control. This is because if the user has read permissions on the database, they have read permissions on all schema objects in the database. After you configure fine-grained access control on your database, fine-grained access control users with the `  SELECT  ` privilege on the view and users with database-level permissions on the database can query the view. The difference is that the fine-grained access control user doesn't need privileges on the underlying objects.

## Limitations of views

Views have limitations compared to actual tables that make them inappropriate for certain use cases.

  - **Views are read-only. They cannot be used to add, update or delete data.**
    
    You cannot use views in DML statements ( `  INSERT  ` , `  UPDATE  ` , `  DELETE  ` ).

  - **The query that defines a view cannot use query parameters.**

  - **Views cannot be indexed.**

  - **References to views cannot use [table hints](/spanner/docs/reference/standard-sql/query-syntax#table-hints) .**
    
    However, the query that defines a view can include table hints on the tables it refers to.

  - **Views are not supported by the [Read](/spanner/docs/reference/rpc/google.spanner.v1#google.spanner.v1.Spanner.Read) API.**

  - **Definer's rights views are not supported with Spanner [Data Boost](/spanner/docs/databoost/databoost-overview) .**
    
    Running a query that contains a definer's rights view in Data Boost results in an error.

  - **The recommended [query mode](/spanner/docs/reference/rest/v1/QueryMode) for accessing a definer's rights view is `  NORMAL  ` mode.**
    
    Users who don't have access to the underlying schema objects of a definer's rights view receive an error when querying in a query mode other than normal.

  - **It's possible for a user to create a carefully crafted query that causes Spanner to throw an error that shows or reveals the existence of data that is not available in the definer's rights view.**
    
    For example, assume there is the following view QualifiedStudentScores which returns scores of students who qualify for a course. The criteria for qualifying is based on the level and exam score of the student. If the student's level is equal or lower than six, the score matters, and the student has to get at least 50 points on the exam to qualify. Otherwise, for levels equal or greater than six, the student qualifies by default.
    
    ``` text
      CREATE VIEW QualifiedStudentScores
      SQL SECURITY DEFINER AS
      SELECT
        s.Name,
        s.Level,
        sc.Score
      FROM Students AS s
      JOIN Scores AS sc ON sc.StudentId = s.StudentId
      WHERE
      (CASE
        WHEN (s.Level < 6) OR (s.Level >= 6 AND sc.Score >= 50)
          THEN 'QUALIFIED';
        ELSE 'FAILED';
      END) = 'QUALIFIED';
    ```
    
    A user can run a query in the form of `  SELECT * FROM QualifiedStudentScores s WHERE s.Level = 7 AND 1/(s.Score - 20) = 1;  ` . This query might fail with a division by zero error if there is a student in level 7 who got a score of 20 points, even though the view limits data to 50 points and above for that level.

## Query performance when using views

A query that refers to a view performs comparably to that same query with its view reference replaced by the view's definition.

## Quotas and limits that apply to views

  - The [Quotas & limits](/spanner/quotas#views) page lists quota and limit information specifically for views.

  - Using a view in a query can affect that query's conformance to [query limits](/spanner/quotas#tables) because the view's definition becomes part of the query.

## Cost impact

Using views has a very small impact on the cost of an instance:

  - Using views has no impact on the compute capacity needs of an instance, as compared to embedding their defined query text in queries that refer to them.

  - Using views has very small impact on the database storage of an instance because the table generated by executing a view's query definition is not saved to persistent database storage.

## What's next

  - Learn how to [Create and manage views](/spanner/docs/create-manage-views) .
