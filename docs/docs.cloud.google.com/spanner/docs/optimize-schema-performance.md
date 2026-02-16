Performance tuning is an iterative process in which you evaluate metrics like CPU utilization and latency, adjust your schema and application to improve performance, and test again.

For example, in your schema, you might add or change an index, or change a primary key. In your application, you might batch writes, or you might merge or modify your queries.

For production traffic in particular, performance tuning is important to help avoid surprises. Performance tuning is more effective the closer the setup is to live production traffic throughput and data sizes.

To test and tune your schema and application performance, follow these steps:

1.  Upload a subset of your data into a Spanner database. You can use the [BigQuery reverse ETL workflow](/bigquery/docs/export-to-spanner) to load the sample data. For more information, see [Load sample data](/spanner/docs/load-sample-data) .
2.  Point the application to Spanner.
3.  Verify the database consistency by checking for basic flows.
4.  Verify that performance meets your expectations by performing load tests on your application. For help identifying and optimizing your most expensive queries, see [Detect query performance issues with query insights](/spanner/docs/using-query-insights) . In particular, the following factors can contribute to suboptimal query performance:
    1.  **Inefficient queries** : For information about writing efficient SQL queries, see [SQL best practices](/spanner/docs/sql-best-practices) .
    2.  **High CPU utilization** : For more information, see [Investigate high CPU utilization](/spanner/docs/introspection/investigate-cpu-utilization) .
    3.  **Locking** : To reduce bottlenecks caused by transaction locking, see [Identify transactions that might cause high latencies](/spanner/docs/use-lock-and-transaction-insights) .
    4.  **Inefficient schema design** : If the schema isn't designed well, query optimization isn't very useful. For more information about designing good schemas, see [Schema design best practices](/spanner/docs/schema-design) .
    5.  **Hotspots** : Hotspots in Spanner limit write throughput, especially for high-QPS applications. To identify hotspots or schema design issues, check the [Key Visualizer](/spanner/docs/key-visualizer) statistics from the Google Cloud console. For more information about avoiding hotspots, see [Choose a primary key to prevent hotspots](/spanner/docs/schema-design#primary-key-prevent-hotspots) .
5.  If you modify schema or indexes, repeat database consistency and performance testing until you achieve satisfactory results.

For more information about fine-tuning your database performance, contact [Spanner support](/spanner/docs/getting-support) .
