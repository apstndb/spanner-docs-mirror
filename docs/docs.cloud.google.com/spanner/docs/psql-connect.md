---
name: documents/docs.cloud.google.com/spanner/docs/psql-connect
uri: https://docs.cloud.google.com/spanner/docs/psql-connect
title: Connect psql to a PostgreSQL-dialect database
description: Connect to Spanner PostgreSQL-dialect databases using the `psql` command-line tool through Spanner PG Connector (`spgc`) or PGAdapter.
data_source: docs.cloud.google.com
---

This page explains how to connect `psql` to a PostgreSQL-dialect database in Spanner. `psql` is the command-line frontend to PostgreSQL.

You can connect `psql` to Spanner in two ways:

- **Using Spanner PG Connector ( `spgc` )** : `spgc` is a standalone CLI wrapper that bundles PGAdapter and a minimal Java runtime. It starts and stops PGAdapter in the background when you run `psql` .
- **Using a running PGAdapter instance** : start and manage the PGAdapter proxy process separately. Then connect `psql` to `localhost` .

## Connect using Spanner PG Connector ( `spgc` )

`spanner-pg-connector` (aliased as `spgc` ) lets you run `psql` without installing Java or Docker or manually managing a separate PGAdapter process.

1.  Install `psql` on your `PATH` , and install `spgc` by following the instructions in [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start#spgc) .

2.  Configure [Application Default Credentials (ADC)](https://docs.cloud.google.com/docs/authentication/provide-credentials-adc) , for example, by running `gcloud auth application-default login` or setting `GOOGLE_APPLICATION_CREDENTIALS` .

3.  Run `psql` through `spgc` by specifying the full database resource name:

    ```
    spgc psql -d "projects/PROJECT_ID/instances/INSTANCE_ID/databases/DATABASE_ID"
    ```

    Alternatively, set `GOOGLE_CLOUD_PROJECT` and `SPANNER_INSTANCE` (and optionally `SPANNER_DATABASE` ):

    ```
    export GOOGLE_CLOUD_PROJECT=PROJECT_ID
    export SPANNER_INSTANCE=INSTANCE_ID

    spgc psql -d DATABASE_ID
    ```

    > **Note:** Don't pass `-h` ( `--host` ) or `-p` ( `--port` ) when using `spgc` . `spgc` starts PGAdapter on a dynamically assigned `localhost` port and sets `PGHOST` and `PGPORT` for the `psql` command.

4.  Optional: Verify that `psql` connected to a PostgreSQL-dialect database by running the following query:

    ```
    SELECT 1::bigint;
    ```

    This query is incompatible with GoogleSQL-dialect databases.

## Connect to a running PGAdapter instance

If you're running PGAdapter as a separate process or container:

1.  Start PGAdapter on the same machine as the `psql` command.

    For more information, see [Start PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter-start) .

2.  In a terminal window, start `psql` and specify `localhost` as the database server host.

    - Optionally specify a port number if PGAdapter is configured to listen on a port other than the default PostgreSQL port (5432).
    - Optionally specify one or more commands to send to the database or driver (for example, the JDBC driver). For each occurrence of the `-c` option, you can specify a single command or a batch of commands separated by semicolons (;). For more information, see [psql command-line tool](https://docs.cloud.google.com/spanner/docs/psql-commands) .

    `psql -h localhost [-p `` PORT `` ] [-c " `` COMMAND `` ; ..."] ...`

3.  Optional: Verify that `psql` successfully connected to a PostgreSQL-dialect database by submitting the following query:

    `SELECT 1::bigint;`

    This query is incompatible with GoogleSQL-dialect databases.

## What's next

- Learn about [PGAdapter](https://docs.cloud.google.com/spanner/docs/pgadapter) .
- See the supported `psql` commands in [psql command-line tool](https://docs.cloud.google.com/spanner/docs/psql-commands) .
