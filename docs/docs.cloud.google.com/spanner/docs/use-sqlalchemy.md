[SQLAlchemy](https://www.sqlalchemy.org/) is a Python SQL toolkit and Object Relational Mapper.

The [Spanner dialect for SQLAlchemy](https://github.com/googleapis/python-spanner-sqlalchemy) lets SQLAlchemy users to use Spanner databases. The dialect is built on top of the [Spanner API](https://github.com/googleapis/python-spanner/tree/main/google/cloud/spanner_dbapi) , which is designed in accordance with [PEP-249](https://www.python.org/dev/peps/pep-0249/) , and is compatible with SQLAlchemy versions between [1.1.13](https://pypi.org/project/SQLAlchemy/1.1.13/) and [1.3.23](https://pypi.org/project/SQLAlchemy/1.3.23/) , and [2.0](https://pypi.org/project/SQLAlchemy/2.0.0/) .

**PostgreSQL interface note:** To use [PostgreSQL interface for Spanner](/spanner/docs/postgresql-interface) with SQLAlchemy, see [Integrate Spanner with SQLAlchemy 2 ORM (PostgreSQL-dialect)](/spanner/docs/use-sqlalchemy-pg) .

## Set up the Spanner dialect for SQLAlchemy

To set up the Spanner dialect for SQLAlchemy in your application, install the [`  sqlalchemy-spanner package  `](https://pypi.org/project/sqlalchemy-spanner/) .

``` text
pip3 install sqlalchemy-spanner
```

Alternatively, you can install from source.

``` text
git clone https://github.com/googleapis/python-spanner-sqlalchemy.git
cd python-spanner-sqlalchemy
python setup.py install
```

As authentication for the Spanner dialect for SQLAlchemy, provide the [service account JSON credentials](/docs/authentication/getting-started) file location in the `  GOOGLE_APPLICATION_CREDENTIALS  ` environment variable. Otherwise, the dialect can also use the default credentials set in the gcloud CLI application.

## Use the Spanner dialect for SQLAlchemy

For more information about the available features, limitations of the dialect, recommendations on how to use the dialect, and for code samples, please consult the [reference documentation](https://github.com/googleapis/python-spanner-sqlalchemy#readme) on GitHub.

## What's next

  - Check out the [code examples](https://github.com/googleapis/python-spanner-sqlalchemy/blob/main/samples/snippets.py) on how to use the Spanner dialect for SQLAlchemy.
  - View the repository for the Spanner dialect for SQLAlchemy on [GitHub](https://github.com/googleapis/python-spanner-sqlalchemy) .
  - File a [GitHub issue](https://github.com/googleapis/python-spanner-sqlalchemy/issues) to report a bug or ask a question about the Spanner dialect for SQLAlchemy.
  - Learn more about [SQLAlchemy](https://www.sqlalchemy.org/) .
  - Learn more about [PyPI](https://pypi.org/) .
  - Learn about authorization and authentication credentials in [Getting started with authentication](/docs/authentication/getting-started) .
