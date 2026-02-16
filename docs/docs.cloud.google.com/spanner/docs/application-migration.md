A typical application uses a client, such as a low-level driver or an object-relational mapper (ORM) to connect to the database. Spanner supports clients in multiple languages that let you use common libraries, such as JDBC, across developer environments. For more information, see [Overview of drivers](/spanner/docs/drivers-overview) .

In addition to managing how your application connects to Spanner, you also have to manage which queries are sent and which syntax the queries are using. Spanner supports two SQL dialects, both based on ANSI SQL standard. You can choose to use GoogleSQL or PostgreSQL interface for Spanner based on the ecosystem you want to operate in. For more information, see [Choosing the right dialect for your Spanner database](/spanner/docs/choose-googlesql-or-postgres) . Because of the architectural differences between the Spanner database and your source database, the syntax used in Spanner might not align with the syntax of your source database.

Complete the following steps manually to migrate your application to Spanner:

  - Spanner doesn't support running user code in the database, so you need to move any procedures and triggers stored at the database level into the application.

  - Use Spanner client libraries and ORMs. For more information, see [Overview of APIs, client libraries, and ORM drivers](/spanner/docs/api-libraries-overview) .

  - Take note of [Spanner partitioned DML](/spanner/docs/dml-partitioned) , [read-only transactions](/spanner/docs/transactions#read-only_transactions) , [commit timestamps](/spanner/docs/commit-timestamp) , and read timestamps and how they can optimize application performance.

  - You also might need to make changes to transaction handling. Consider the following:
    
      - The mutations per commit limit is 80,000. Each secondary index on a table is an additional mutation per row. To modify data using mutations, see [Insert, update, and delete data using mutations](/spanner/docs/modify-mutation-api) . To modify a large amount of data, use [partitioned DML](/spanner/docs/dml-partitioned) .
