---
name: documents/docs.cloud.google.com/spanner/docs/reference/standard-sql/procedural-language
uri: https://docs.cloud.google.com/spanner/docs/reference/standard-sql/procedural-language
title: Procedural language in GoogleSQL
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

The GoogleSQL procedural language lets you execute multiple statements in one query as a multi-statement query. You can use a multi-statement query to:

  - Run multiple statements in a sequence, with shared state.
  - Automate management tasks such as creating or dropping tables.

## `CALL`

**Syntax**

    CALL procedure_name (procedure_argument[, …])

**Description**

Calls a [procedure](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/stored-procedures) with an argument list. `procedure_argument` may be a variable or an expression. Spanner doesn't support stored procedures or server-side scripts. Use `CALL` with only the example procedures listed on this page.

The maximum depth of procedure calls is 50 frames.

**Examples**

The following example cancels a query with the query ID `12345` .

    CALL cancel_query("12345");
