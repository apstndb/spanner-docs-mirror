## Client

Spanner supports [SQL queries](/spanner/docs/reference/standard-sql/query-syntax) . Here's a sample query:

``` text
SELECT s.SingerId, s.FirstName, s.LastName, s.SingerInfo
FROM Singers AS s
WHERE s.FirstName = @firstName;
```

The construct `  @firstName  ` is a reference to a query parameter. You can use a query parameter anywhere a literal value can be used. Using parameters in programmatic APIs is strongly recommended. Use of query parameters helps avoid [SQL injection](https://en.wikipedia.org/wiki/SQL_injection) attacks and the resulting queries are more likely to benefit from various server-side caches. For more information, see [Caching](#caching) .

Query parameters must be bound to a value when the query is executed. For example:

``` text
Statement statement =
    Statement.newBuilder("SELECT s.SingerId...").bind("firstName").to("Jimi").build();
try (ResultSet resultSet = dbClient.singleUse().executeQuery(statement)) {
 while (resultSet.next()) {
 ...
 }
}
```

Once Spanner receives an API call, it analyzes the query and bound parameters to determine which Spanner server node should process the query. The server sends back a stream of result rows that are consumed by the calls to `  ResultSet.next()  ` .

## Query execution

Query execution begins with the arrival of an "execute query" request at some Spanner server. The server performs the following steps:

  - Validate the request
  - Parse the query text
  - Generate an initial query algebra
  - Generate an optimized query algebra
  - Generate an executable query plan
  - Execute the plan (check permissions, read data, encode results, etc.)

## Parsing

The SQL parser analyzes the query text and converts it to an [abstract syntax tree](https://en.wikipedia.org/wiki/Abstract_syntax_tree) . It extracts the basic query structure `  (SELECT … FROM … WHERE …)  ` and does syntactic checks.

## Algebra

Spanner's [type system](/spanner/docs/reference/standard-sql/data-types) can represent scalars, arrays, structures, etc. The query algebra defines operators for table scans, filtering, sorting/grouping, all sorts of joins, aggregation, and much more. The initial query algebra is built from the output of the parser. Field name references in the parse tree are resolved using the database schema. This code also checks for semantic errors (e.g., incorrect number of parameters, type mismatches, and so forth).

The next step ("query optimization") takes the initial algebra and generates a more-optimal algebra. This might be simpler, more efficient, or just more-suited to the capabilities of the execution engine. For example, the initial algebra might specify just a "join" while the optimized algebra specifies a "hash join".

## Execution

The final executable query plan is built from the rewritten algebra. Basically, the executable plan is a [directed acyclic graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph) of "iterators". Each iterator exposes a sequence of values. Iterators may consume inputs to produce outputs (e.g., sort iterator). Queries that involve a single [split](/spanner/docs/schema-and-data-model#database-splits) can be executed by a single server (the one that holds the data). The server will scan ranges from various tables, execute joins, perform aggregation, and all other operations defined by the query algebra.

Queries that involve multiple splits will be factored into multiple pieces. Some part of the query will continue to be executed on the main (root) server. Other partial subqueries are handed-off to leaf nodes (those that own the splits being read). This hand-off can be recursively applied for complex queries, resulting in a tree of server executions. All servers agree on a timestamp so that the query results are a consistent snapshot of the data. Each leaf server sends back a stream of partial results. For queries involving aggregation, these could be partially-aggregated results. The query root server processes results from the leaf servers and runs the remainder of the query plan. For more information, see [Query execution plans](/spanner/docs/query-execution-plans) .

When a query involves multiple splits, Spanner can execute the query in parallel across the splits. The degree of parallelism depends on the range of data that the query scans, the query execution plan, and the distribution of data across splits. Spanner automatically sets the maximum degree of parallelism for a query based on its instance size and [instance configuration](/spanner/docs/instance-configurations) (regional or multi-region) in order to achieve optimal query performance and avoid [overloading the CPU](/spanner/docs/introspection/investigate-cpu-utilization) .

## Caching

Many of the artifacts of query processing are automatically cached and re-used for subsequent queries. This includes query algebras, executable query plans, etc. The caching is based on the query text, names and types of bound parameters. This is why using bound parameters (like `  @firstName  ` in the example above) is better than using literal values in the query text. The former can be cached once and reused regardless of the actual bound value. See [Optimizing Spanner Query Performance](/spanner/docs/whitepapers/improving-query-performance) for more details.

## Error handling

The `  executeQuery  ` (or `  executeStreamingSql  ` ) and `  streamingRead  ` methods return a stream of `  PartialResultSet  ` messages. For efficiency, a single row or column value might be split across multiple `  PartialResultSet  ` messages, especially for large data.

This stream can be interrupted by transient network errors, split handoffs, or server restarts. Split handoffs might occur during load balancing and server restarts might occur during upgrades.

To handle these interruptions, Spanner includes opaque `  resume_token  ` strings in some `  PartialResultSet  ` messages.

Key points about `  resume_token  ` :

  - Not every `  PartialResultSet  ` contains a `  resume_token  ` .
  - A `  resume_token  ` is typically included only at the end of a complete row, marking a safe resumption point.
  - `  PartialResultSet  ` with a `  chunked_value  ` (for large values split across messages) won't have a `  resume_token  ` until the entire value and row are sent.
  - To resume an interrupted stream, send a new request with the *last received* non-empty `  resume_token  ` .

The Spanner client libraries automatically manage this buffering and recovery. They assemble complete rows from `  PartialResultSet  ` messages and track the latest `  resume_token  ` . If the connection drops, the library uses the last valid token to restart the stream, discarding any partial data received after that token. This process ensures you see a continuous, duplicate-free stream of complete rows, even if transient failures occur.
