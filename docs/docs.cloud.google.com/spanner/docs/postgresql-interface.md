The PostgreSQL interface for Spanner lets you take advantage of Spanner's fully managed, scalable, and highly available infrastructure using familiar PostgreSQL tools and syntax. This page helps you understand the capabilities and limitations of the PostgreSQL interface.

## Benefits of the PostgreSQL interface

  - **Portability** : the PostgreSQL interface provides access to the breadth of Spanner features, using schemas, queries, and clients that are compatible with open source PostgreSQL. This simplifies moving an application built on Spanner to another PostgreSQL environment. This portability provides deployment flexibility and supports disaster recovery scenarios, such as a stressed exit.
  - **Familiarity** : if you already use PostgreSQL, you can quickly get started with Spanner using many of the same PostgreSQL statements and tools. Using PostgreSQL across your database portfolio means fewer variations between specific products and a common set of best practices.
  - **Uncompromisingly Spanner** : because it's built on Spanner's existing foundation, the PostgreSQL interface provides all of Spanner's existing availability, consistency, and price-performance benefits without having to compromise on any of the capabilities available in the complementary GoogleSQL ecosystem.

## Unsupported PostgreSQL features on Spanner

It's important to understand that the PostgreSQL interface provides the capabilities of Spanner through schemas, types, queries, and clients that are compatible with PostgreSQL. It doesn't support all of the features of PostgreSQL. Migrating an existing PostgreSQL application to Spanner, even using the PostgreSQL interface for Spanner, likely requires some rework to accommodate unsupported PostgreSQL capabilities or differences in behavior, like query optimization or primary key design. However, once it's migrated, your workloads can take advantage of Spanner's reliability and unique multi-model capabilities.

The following list provides more information on supported and unsupported PostgreSQL features:

  - **Supported PostgreSQL functionality** : the PostgreSQL interface supports many of the most commonly used features of PostgreSQL. This includes core parts of the schema and type system, many common query shapes, a variety of functions and operators, and the key aspects of PostgreSQL's system catalog. Applications can use many PostgreSQL clients by connecting over Spanner's implementation of the PostgreSQL wire protocol.
  - **Some PostgreSQL language features aren't supported** : extensions, user-defined data types, user-defined stored procedures, and other features aren't supported. For a complete list, see [The PostgreSQL language in Spanner](/spanner/docs/reference/postgresql/overview) . There are also some features in PostgreSQL that behave differently from open source PostgreSQL. For more information, see [Known issues in the PostgreSQL interface for Spanner](/spanner/docs/reference/postgresql/known-issues-postgresql-interface) .
  - **Spanner control plane** : databases with PostgreSQL interfaces use Spanner and Google Cloud tools to provision, secure, monitor, and optimize instances. Spanner doesn't support tools, such as pgAdmin for administrative activities.
  - **Client and wire protocol support** : Spanner supports the core query capabilities of the PostgreSQL wire protocol using PGAdapter, a lightweight proxy that runs alongside your application. This lets many Spanner clients work as-is with a Spanner PostgreSQL interface database, while leveraging Spanner's global endpoint and connection management and IAM authentication. Google's internal benchmarking shows that PGAdapter doesn't add any noticeable additional latency compared to direct connection to Spanner's built-in endpoints.

## Administration and Management

The PostgreSQL interface supports the administration and management of your Spanner databases with the following features:

  - **Unified experience** : provision, manage, and monitor PostgreSQL interface-enabled databases using Spanner's existing console, APIs, and tools like Google Cloud CLI.
  - **Flexible configuration** : configure the PostgreSQL interface per database at creation time. A single Spanner instance can accommodate both GoogleSQL and PostgreSQL interface databases.
  - **Shared benefits** : both database dialects share the same underlying distributed database engine, ensuring consistent scalability, consistency, performance, and security.

## Features

Spanner's PostgreSQL interface offers two primary features that enable integration with the PostgreSQL ecosystem:

  - **PostgreSQL dialect support**
    
    Spanner provides a subset of the PostgreSQL SQL dialect, including Data Query Language (DQL), Data Manipulation Language (DML), and Data Definition Language (DDL). Additionally, it includes extensions to support Spanner-specific features like [interleaved tables](/spanner/docs/schema-and-data-model#parent-child) , [time to live (TTL)](/spanner/docs/ttl) , and [query hints](/spanner/docs/reference/postgresql/query-syntax#pg_extensions) .
    
    For detailed information on the supported PostgreSQL language elements, see [The PostgreSQL language in Spanner](/spanner/docs/reference/postgresql/overview) . To understand how to use Spanner features with the PostgreSQL dialect, consult the documentation for the specific feature.

  - 
    
    <div id="connect-pg">
    
    **PostgreSQL client support**
    
    Spanner lets you connect to databases from a variety of clients:
    
      - **PostgreSQL ecosystem tools:** you can use familiar tools like the [PostgreSQL JDBC driver](/spanner/docs/pg-jdbc-connect) and [PostgreSQL pgx driver](https://github.com/jackc/pgx) to connect your applications to a PostgreSQL interface database. For a list of supported drivers, ORMs, and tools see [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql-drivers-and-orms) .
    
      - **psql command-line tool** : the popular [`  psql  ` interactive environment](/spanner/docs/psql-commands) is supported, letting you run queries, explore metadata, and load data directly from your terminal.
    
      - **PGAdapter:** this lightweight proxy simplifies connection management and authentication. For more details, refer to the [PGAdapter overview](/spanner/docs/pgadapter) .
    
      - **Spanner clients:** Spanner provides open source Spanner clients for various programming languages (Java, Go, Python, Node.js, Ruby, PHP, C\#, C++), along with a [Spanner JDBC driver](https://github.com/googleapis/java-spanner-jdbc) and a [driver for Go's SQL package](https://github.com/googleapis/go-sql-spanner) . Spanner clients connect directly to Spanner's global endpoint without a proxy. However, Spanner clients don't offer compatibility with existing PostgreSQL clients, ORMs, or tools.
    
    </div>

## Best practices for using the PostgreSQL interface

Use the following best practices when using the PostgreSQL interface:

1.  **Connect your applications** : use the [set of supported PostgreSQL tools](#connect-pg) for efficient connectivity.

2.  **Interact with your database** : for interactive work, choose between the following:
    
      - The familiar [psql command-line tool](/spanner/docs/psql-connect) (using the [PGAdapter proxy](/spanner/docs/pgadapter-get) )
      - The intuitive [Spanner Studio](/spanner/docs/manage-data-using-console) page within the Google Cloud console
      - IDEs, such [DBeaver](https://cloud.google.com/blog/topics/developers-practitioners/exploring-cloud-spanner-data-dbeaver/) and [Visual Studio Code](https://cloud.google.com/blog/topics/developers-practitioners/browse-and-query-cloud-spanner-databases-visual-studio-code) , [JetBrains](https://cloud.google.com/blog/topics/developers-practitioners/cloud-spanner-connectivity-using-jetbrains-ides) , and [IntelliJ](/spanner/docs/use-intellij)
      - The Spanner emulator which lets you emulate Spanner on your local machine. This is useful during the development and test process.

### What's next

  - Learn how to [choose between PostgreSQL and GoogleSQL](/spanner/docs/choose-googlesql-or-postgres) .
  - Follow the [quickstart](/spanner/docs/create-query-database-console) to create and interact with a PostgreSQL database.
  - Learn more about [Spanner's PostgreSQL language support](/spanner/docs/reference/postgresql/overview) .
  - Learn about [PGAdapter](/spanner/docs/pgadapter) .
  - Learn about the [PGAdapter GitHub repository](https://github.com/GoogleCloudPlatform/pgadapter) .
  - Review [known issues](/spanner/docs/known-issues-postgresql-interface) in the PostgreSQL interface.
