---
name: documents/docs.cloud.google.com/spanner/docs/pgadapter
uri: https://docs.cloud.google.com/spanner/docs/pgadapter
title: PGAdapter overview
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

PGAdapter translates the PostgreSQL wire protocol into the Spanner gRPC protocol. It lets you connect PostgreSQL applications and drivers to the Spanner PostgreSQL interface for Spanner databases with minimal latency overhead.

To learn how to start PGAdapter, see [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start) .

You can run PGAdapter as a sidecar proxy alongside your main application or, for Java applications, in-process directly within the application JVM. By exposing an endpoint on localhost that supports the PostgreSQL wire protocol, PGAdapter enables tools like `psql` to connect to Spanner.

PGAdapter adds at most 0.2 ms of latency overhead. The PostgreSQL interface maintains the same latency levels as GoogleSQL.

The following diagram shows how `psql` connects to Spanner through PGAdapter. ![psql connecting to Spanner through PGAdapter.](https://docs.cloud.google.com/static/spanner/docs/images/pgadapter-diagram.png)

PGAdapter supports basic and extended query modes, and any data type that the PostgreSQL interface for Spanner supports.

## Supported drivers and ORMs

PGAdapter supports standard PostgreSQL drivers, ORMs, and client tools, including:

  - **Command-line tools** : [`psql`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/psql.md) (versions 11 through 14) and [JetBrains IDEs](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/intellij.md) (such as DataGrip).
  - **Java/JVM** : [JDBC](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/jdbc.md) (version 42.x and higher), [R2DBC](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/java/r2dbc) , [Hibernate](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/java/hibernate) , and [Spring Data JPA](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/java/spring-data-jpa) .
  - **Go** : [`pgx`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/pgx.md) (version 4.15 and higher) and [`gorm`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/gorm.md) .
  - **Python** : [`psycopg2`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/psycopg2.md) , [`psycopg3`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/psycopg3.md) , [`SQLAlchemy`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/sqlalchemy.md) , and [Python ADBC](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/adbc.md) .
  - **Node.js** : [`node-postgres`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/node-postgres.md) , [`Knex.js`](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/nodejs/knex) , [`Sequelize.js`](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/nodejs/sequelize) , [`Prisma`](https://github.com/GoogleCloudPlatform/pgadapter/tree/master/samples/nodejs/prisma-sample-app) , and [`Drizzle`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/drizzle.md) .
  - **Other languages** : [`npgsql`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/npgsql.md) (.NET) and [`PDO_PGSQL`](https://github.com/GoogleCloudPlatform/pgadapter/blob/master/docs/pdo.md) (PHP).

For a complete compatibility matrix, see [PostgreSQL drivers and ORMs](https://docs.cloud.google.com/spanner/docs/drivers-overview#postgresql_drivers_and_orms) .

## PGAdapter execution environments

You can run PGAdapter by using one of the following methods:

  - **Standalone** : PGAdapter is supplied as a JAR file and runs standalone in the JVM.
  - **Docker** . PGAdapter is also packaged as a Docker image.
  - **Cloud Run** : PGAdapter can be deployed as a sidecar proxy on Cloud Run.
  - **Sidecar proxy** : a typical use as a sidecar proxy is in a Kubernetes cluster.
  - **In-process** : your Java application code can use the supplied JAR file to create and start a PGAdapter instance.

For details about these methods, see [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start) .

## Authorization with PGAdapter

PGAdapter determines the service account or other Identity and Access Management (IAM) principal for the connection by examining the credentials that you specify when you start the proxy. The IAM permissions granted to that principal determine the allowed database operations.

When using fine-grained access control, specify a database role when starting PGAdapter. PGAdapter uses fine-grained access control when sending queries and DML statements, which requires the IAM `spanner.databases.useRoleBasedAccess` permission. For database roles other than `public` , the principal also needs the `spanner.databaseRoles.use` permission. The privileges granted to the database role determine the operations that the connecting application can perform. If you don't specify a database role, the permissions granted to the IAM principal apply. To perform DDL statements, the principal must have the `spanner.databases.updateDdl` permission.

For more information, see [About fine-grained access control](https://docs.cloud.google.com/spanner/docs/fgac-about) and [Access control with IAM](https://docs.cloud.google.com/spanner/docs/iam) .

## What's next

  - [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start)
  - Learn more about the [PGAdapter GitHub repository](https://github.com/GoogleCloudPlatform/pgadapter) .
  - Learn more about [PostgreSQL drivers and ORMs](https://docs.cloud.google.com/spanner/docs/drivers-overview#postgresql_drivers_and_orms) for a table of PostgreSQL drivers and ORMs that PGAdapter supports.
