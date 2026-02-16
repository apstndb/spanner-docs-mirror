This page summarizes PostgreSQL language support in Spanner.

Spanner provides PostgreSQL language support by expressing Spanner database features through a subset of open source PostgreSQL language constructs, with extensions added to support Spanner features like interleaved tables and hints.

For detailed information about this support, refer to these pages:

  - [Lexical structure and syntax](/spanner/docs/reference/postgresql/lexical) describes how to form valid PostgreSQL statements.
  - [Data definition (DDL)](/spanner/docs/reference/postgresql/data-definition-language) defines the syntax of DDL statements like `  CREATE DATABASE  ` and `  CREATE TABLE  ` .
  - [Data manipulation (DML)](/spanner/docs/reference/postgresql/dml-syntax) defines the syntax of DML statements like `  INSERT  ` and `  UPDATE  ` .
  - [Queries](/spanner/docs/reference/postgresql/query-syntax) defines the syntax of the `  SELECT  ` statement.
  - [Subqueries](/spanner/docs/reference/postgresql/subqueries) defines the syntax of subqueries.
  - [Data types](/spanner/docs/reference/postgresql/data-types) describes the data types that Spanner supports.
  - [Functions](/spanner/docs/reference/postgresql/functions-and-operators) describes the functions that Spanner supports.
  - [Stored procedures](/spanner/docs/reference/postgresql/stored-procedures-pg) describes stored procedures that come with Spanner.
  - [Known issues in the PostgreSQL interface for Spanner](/spanner/docs/reference/postgresql/known-issues-postgresql-interface) describes PostgreSQL language features that don't work as expected.

Spanner doesn't support several open source PostgreSQL features, including the following:

  - Triggers
  - SERIAL
  - Fine-grained concurrency control
  - SAVEPOINT
  - Transactional DDL
  - Partial indexes
  - Extensions
  - Foreign data wrappers
  - User-defined data types, functions and operators
