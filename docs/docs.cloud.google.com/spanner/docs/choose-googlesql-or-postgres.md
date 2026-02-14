When creating a Spanner database, you can choose between two SQL dialects: GoogleSQL and PostgreSQL. Both dialects offer the same core Spanner features, performance, and scalability. Requirements of applications, developers, and the ecosystem in which you work should inform your choice of dialect. This page lists the deciding factors between using GoogleSQL and PostgreSQL interface dialect databases.

**Key Considerations:**

  - **Portability** : if you choose PostgreSQL, you have the option of migrating from Spanner to another PostgreSQL database.
  - **Familiarity:** if your team is already familiar with either PostgreSQL or GoogleSQL syntax and tools, choosing that dialect can streamline development and reduce the learning curve.
  - **Ecosystem:** consider the tools and libraries available for each dialect. GoogleSQL is well-integrated with Google Cloud services, while PostgreSQL has a vast open-source ecosystem.
  - **Application requirements:** assess your application's specific requirements regarding SQL syntax, data types, and potential future needs.
  - **Migration:** if you are migrating from an existing database, choosing the dialect closer to your current environment might simplify the migration process.

If portability is your highest priority, giving you the option to move away from Google Cloud, choose PostgreSQL. If you want the tightest integration with Google Cloud (for example, to use BigQuery), then choose GoogleSQL.

**Spanner implementation of GoogleSQL and the PostgreSQL interface:**

  - **Feature parity:** Both GoogleSQL and PostgreSQL dialects provide equivalent support for Spanner's unique features, such as interleaved tables and query hints.
  - **Underlying engine:** Both dialects share the same underlying distributed storage and query processing engine, ensuring consistent performance, scalability, and reliability.
  - **Management and development:** You can manage and develop applications for both dialects using the same Spanner tools, APIs, and client libraries.

## What's next

  - Learn about [dialect parity between GoogleSQL and PostgreSQL](/spanner/docs/reference/dialect-differences) .
