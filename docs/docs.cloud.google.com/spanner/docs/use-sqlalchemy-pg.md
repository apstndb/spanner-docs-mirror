[SQLAlchemy 2](https://docs.sqlalchemy.org/en/20/) is a Python SQL toolkit and Object Relational Mapper (ORM).

You can use the SQLAlchemy 2 ORM in combination with the standard [PostgreSQL psycopg3 driver](https://www.psycopg.org/psycopg3/) and PGAdapter.

## Set up SQLAlchemy 2 with Spanner PostgreSQL-dialect databases

1.  Ensure that PGAdapter is running on the same machine as the application that is connecting using SQLAlchemy 2 with Spanner.
    
    For more information, see [Start PGAdapter](/spanner/docs/pgadapter-start) .

2.  Add SQLAlchemy 2 and psycopg3 to your `  requirements.txt  ` file.
    
    ``` text
    psycopg~=3.1.8
    sqlalchemy~=2.0.1
    ```

3.  Specify `  postgresql+psycopg  ` , `  localhost  ` and `  5432  ` as the database dialect, driver, server host and port in the SQLAlchemy 2 connection string. psycopg3 requires a username and password in the connection string. PGAdapter ignores these.
    
    Optionally, specify a different port number if PGAdapter is configured to listen on a port other than the default PostgreSQL port (5432).
    
    ``` text
    conn_string = "postgresql+psycopg://user:password@localhost:5432/my-database"
    engine = create_engine(conn_string)
    ```

See the [SQLAlchemy 2 with PostgreSQL documentation](https://docs.sqlalchemy.org/en/20/dialects/postgresql.html#dialect-postgresql) for more connection options for PostgreSQL.

## Use SQLAlchemy 2 with PostgreSQL-dialect databases

For more information about the features and recommendations for SQLAlchemy 2 with PostgreSQL-dialect databases, please consult the [reference documentation](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/python/sqlalchemy2-sample) on GitHub.

## What's next

  - Check out the [sample-application](https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/python/sqlalchemy2-sample/sample.py) using SQLAlchemy 2 with PGAdapter and Spanner.
  - Learn more about [SQLAlchemy](https://docs.sqlalchemy.org/en/20/) .
  - Learn more about [PGAdapter](/spanner/docs/pgadapter) .
  - [File a GitHub issue](https://github.com/GoogleCloudPlatform/pgadapter/issues) to report a bug or ask a question about Spanner dialect for SQLAlchemy with PGAdapter.
