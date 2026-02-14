This page provides an overview of PGAdapter. To learn how to start PGAdapter, see [Starting PGAdapter](/spanner/docs/pgadapter-start) .

PGAdapter is a sidecar proxy which is a small application that runs alongside your main application to support communications between the [PostgreSQL interface for Spanner](/spanner/docs/postgresql-interface) and Spanner. For Java applications you can even link PGAdapter to the application directly without having to run it in a separate process. PGAdapter is designed to run on the same machine as the application and exposes an endpoint on localhost that supports the PostgreSQL wire protocol. It translates the PostgreSQL wire protocol into the Spanner wire protocol, gRPC. With this proxy running locally, a PostgreSQL client such as `  psql  ` can connect to a PostgreSQL-dialect Spanner database.

PGAdapter adds, at most, 0.2 ms of latency overhead. The PostgreSQL interface has the same latency levels as GoogleSQL.

**Note:** [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql_drivers_and_orms) lists all clients that PGAdapter supports.

The following diagram shows how `  psql  ` connects to Spanner through PGAdapter.

PGAdapter supports basic and extended query modes, and supports any data type that the PostgreSQL interface for Spanner supports.

## PGAdapter execution environments

You can run PGAdapter by using one of the following methods:

  - **Standalone** : PGAdapter is supplied as a JAR file and runs standalone in the JVM.
  - **Docker** . PGAdapter is also packaged as a Docker image.
  - **Cloud Run** : PGAdapter can be deployed as a sidecar proxy on Cloud Run.
  - **Sidecar proxy** : a typical use as a sidecar proxy is in a Kubernetes cluster.
  - **In-process** : your Java application code can use the supplied JAR file to create and start a PGAdapter instance.

For details about these methods, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

## Authorization with PGAdapter

PGAdapter determines the service account or other Identity and Access Management (IAM) principal to use for the connection by examining the credentials that you specify when you start it. The IAM permissions granted to that principal determine the permissions that the connecting application has on the database.

When fine-grained access control is in use, you can optionally specify a database role when you start PGAdapter. If you specify a database role, then PGAdapter uses fine-grained access control when it sends requests for queries and DML statements. This requires the IAM permission `  spanner.databases.useRoleBasedAccess  ` and, for database roles other than `  public  ` , the `  spanner.databaseRoles.use  ` permission. The privileges granted to the database role determine the operations that the connecting application can perform. If you don't specify a database role, then the database-level permissions that are granted to the IAM principal are used. To perform DDL statements, the principal must have the `  spanner.databases.updateDdl  ` permission.

For more information, see [About fine-grained access control](/spanner/docs/fgac-about) and [Access control with IAM](/spanner/docs/iam) .

## What's next

  - [Start PGAdapter](/spanner/docs/pgadapter-start)
  - Learn more about the [PGAdapter GitHub repository](https://github.com/GoogleCloudPlatform/pgadapter) .
  - Learn more about [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql_drivers_and_orms) for a table of PostgreSQL drivers and ORMs that PGAdapter supports.
