---
name: documents/docs.cloud.google.com/spanner/docs/concepts/function-volatility
uri: https://docs.cloud.google.com/spanner/docs/concepts/function-volatility
title: Function volatility
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

In Spanner, functions, including built-in SQL and user-defined functions, are classified by their volatility. The volatility category indicates whether a function returns the same result when you call it with the same arguments. This classification affects how the query optimizer handles the function and where you can use the function, such as in generated column expressions or index definitions. Understanding function volatility helps you write more predictable and performant queries and schemas in Spanner.

There are three volatility categories, which are similar to concepts in systems such as [PostgreSQL](https://www.postgresql.org/docs/13/xfunc-volatility.html) :

  - **Immutable** : A function is *immutable* if it returns the same results when you call it with the same argument values. This behavior means the function's result depends only on its input arguments and not on database state, session settings, or external factors. Most mathematical functions are immutable. The following functions are examples:
    
      - [`ABS`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/mathematical_functions#abs)
    
      - [`LENGTH`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#length)
    
      - String manipulation functions, such as [`CONCAT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#concat) and [`LOWER`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/string_functions#lower)
    
      - Date and time formatting functions with explicit inputs
    
    Functions that depend on runtime-configurable settings, such as [time zone](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#timezone_definitions) , aren't immutable because their results can vary across different database or session configurations.

  - **Stable** : A function is *stable* if it returns the same results for the same argument values within a single statement execution. However, its result can change across different SQL statements. Functions such as [`CURRENT_TIMESTAMP`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/timestamp_functions#current_timestamp) , [`CURRENT_DATE`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/date_functions#current_date) , and similar time-based functions are stable. For example, multiple calls to `CURRENT_TIMESTAMP` within the same `SELECT` statement yield the same timestamp, but subsequent statements might produce a different timestamp.

  - **Volatile** : A function is *volatile* if its value can change even within a single statement execution with the same arguments. For example, the [`GENERATE_UUID`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/utility-functions#generate_uuid) function, which generates a unique ID, is volatile.

## Volatility implications

A function's volatility category dictates where it can be used in your database schema and how queries containing the function are optimized:

  - **Generated columns:** Expressions that define [generated columns](https://docs.cloud.google.com/spanner/docs/generated-column/how-to) must be immutable. This ensures that the stored value is deterministic based on the other columns in the row. Spanner produces an error if you try to define a generated column with an expression that's not immutable.

  - **Indexing:** You can't create indexes on expressions that involve volatile functions. Spanner produces an error if you try to create an index on an expression that involves volatile functions.

  - **Query optimization:** The query optimizer uses volatility information to perform optimizations. For example, the [query optimizer](https://docs.cloud.google.com/spanner/docs/query-optimizer/overview) precalculates calls to immutable functions with constant arguments. The optimizer often caches results of stable functions within the scope of a single statement's execution.
