This page explains how to connect `  psql  ` to a PostgreSQL-dialect database in Spanner. `  psql  ` is the command-line front end to PostgreSQL.

1.  Ensure that PGAdapter is running on the same machine as the `  psql  ` command.
    
    For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

2.  In a terminal window, start `  psql  ` and specify `  localhost  ` as the database server host.
    
      - Optionally specify a port number if PGAdapter is configured to listen on a port other than the default PostgreSQL port (5432).
      - Optionally specify one or more commands to send to the database or driver (for example, the JDBC driver). For each occurrence of the `  -c  ` option, you can specify a single command or a batch of commands separated by semicolons (;). For more information, see [psql command-line tool](/spanner/docs/psql-commands) .
    
    `  psql -h localhost [-p PORT ] [-c " COMMAND ; ..."] ...  `

3.  Optional: Verify that `  psql  ` successfully connected to a PostgreSQL-dialect database by submitting the following query:
    
    `  SELECT 1::bigint;  `
    
    This query is incompatible with GoogleSQL-dialect databases.

## What's next

  - Learn about [PGAdapter](/spanner/docs/pgadapter) .
  - See the supported `  psql  ` commands in [psql command-line tool](/spanner/docs/psql-commands) .
