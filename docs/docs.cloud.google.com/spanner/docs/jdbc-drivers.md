This page describes the JDBC drivers that Spanner supports for GoogleSQL-dialect databases and PostgreSQL-dialect databases. You can use these drivers to connect to Spanner from Java applications. Spanner supports the following JDBC drivers:

  - The Spanner JDBC driver, which is an open-source JDBC driver that is written, provided, and supported by Google, similar to the [Cloud Client Libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) . This is the recommended JDBC driver for Spanner, especially for the GoogleSQL dialect. You can also use it to connect to PostgreSQL-dialect databases.
  - The PostgreSQL JDBC driver in combination with PGAdapter. This driver only supports PostgreSQL-dialect databases.

|             | Spanner JDBC driver                                                                           | PostgreSQL JDBC driver                                                       |
| ----------- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| Download    | [Maven Central](https://search.maven.org/artifact/com.google.cloud/google-cloud-spanner-jdbc) | [Maven Central](https://search.maven.org/artifact/org.postgresql/postgresql) |
| Written by  | Google                                                                                        | PostgreSQL                                                                   |
| Support     | Google                                                                                        | Google                                                                       |
| Open source | Yes; Apache license                                                                           | Yes; BSD 2-Clause                                                            |

> **Note:** Spanner previously supported the Simba JDBC driver. If you already use the Simba driver with Spanner, you can continue to do so.

For information about the Spanner JDBC driver, see [Spanner JDBC driver FAQ](https://docs.cloud.google.com/spanner/docs/open-source-jdbc) and [How to use the Spanner JDBC driver](https://docs.cloud.google.com/spanner/docs/use-oss-jdbc) .

For information about the PostgreSQL JDBC driver, see [Connect JDBC to a PostgreSQL-dialect database](https://docs.cloud.google.com/spanner/docs/pg-jdbc-connect) .
