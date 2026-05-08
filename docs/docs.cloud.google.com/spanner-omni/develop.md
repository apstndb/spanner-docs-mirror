> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

Develop applications with Spanner Omni using supported Spanner features. While many capabilities are shared with Spanner, Spanner Omni has differences in client libraries, the command-line interface (CLI), and console functionality.

Supported Spanner development capabilities include:

  - [Best practices for application design and optimization](https://docs.cloud.google.com/spanner-omni/develop#best-practices) .

  - [Integration with language frameworks](https://docs.cloud.google.com/spanner-omni/develop#language-frameworks) .

  - [Transaction management](https://docs.cloud.google.com/spanner-omni/develop#transactions-concurrency) .

  - [Data access and modification methods](https://docs.cloud.google.com/spanner-omni/develop#data-access-modify) .

  - Features such as [change streams](https://docs.cloud.google.com/spanner-omni/develop#stream-changes) and [full-text search](https://docs.cloud.google.com/spanner-omni/develop#fulltext-search) .

## Key differences

While much of the Spanner development experience applies, the following areas differ in Spanner Omni:

  - **Supported client libraries:** Spanner Omni supports the Java, Go, and Python client libraries.

  - **Command-line interface (CLI):** The Spanner Omni CLI is a dedicated tool that is distinct from the Google Cloud CLI. While it shares a similar syntax, you don't need to provide the `--instance` flag when running commands.

  - **Console limitations:** The Spanner Omni console is read-only, so you can't use it to modify deployments or databases.

The following Spanner features and best practices apply to Spanner Omni.

## Best practices and use cases

Follow these best practices and use cases to design and optimize your applications for Spanner Omni.

  - [Spanner as a gaming database](https://docs.cloud.google.com/spanner/docs/best-practices-gaming-database) : Use Spanner's scalability and consistency for gaming backends.

  - [SQL best practices](https://docs.cloud.google.com/spanner/docs/sql-best-practices) : Optimize your queries for performance and efficiency. Most concepts apply to Spanner Omni. Because Spanner Omni doesn't support tiered storage, concepts such as timestamp predicate pushdown might not fully apply. Also, Spanner Omni doesn't have query plan visualization capabilities.

## Language frameworks

Integrate Spanner Omni with popular language frameworks:

### Hibernate ORM

Use Hibernate Object-Relational Mapping (ORM) to map your Java objects to Spanner Omni tables. For more information, see the following:

  - [Use Hibernate to connect to Spanner Omni](https://docs.cloud.google.com/spanner-omni/hibernate) .

  - [Integrate with Hibernate ORM (PostgreSQL)](https://docs.cloud.google.com/spanner/docs/use-hibernate-postgresql) in the Spanner documentation.

  - [Write a Hibernate app that connects to Spanner](https://docs.cloud.google.com/spanner/docs/write-hibernate-app) in the Spanner documentation.

### GORM

Integrate Spanner Omni with [GORM](https://gorm.io/) , an object-relational mapping (ORM) tool for the Go programming language. To use the object-relational mapping capabilities of GORM in your Go applications. For more information, see the following:

  - [Use GORM to connect to Spanner Omni](https://docs.cloud.google.com/spanner-omni/gorm) .

  - [Integrate with GORM (GoogleSQL)](https://docs.cloud.google.com/spanner/docs/use-gorm) in the Spanner documentation.

  - [Integrate with GORM (PostgreSQL)](https://docs.cloud.google.com/spanner/docs/use-gorm-postgresql) in the Spanner documentation.

## Transactions and concurrency

Spanner Omni supports the following transaction management features:

  - [Transactions overview](https://docs.cloud.google.com/spanner/docs/transactions) : Learn about read-write and read-only transactions.

  - [Timestamp bounds](https://docs.cloud.google.com/spanner/docs/timestamp-bounds) : Control the staleness of your reads. For Spanner Omni, you can configure [version retention period](https://docs.cloud.google.com/spanner/docs/timestamp-bounds#maximum_timestamp_staleness) ( `version_retention_period` ) up to 30 days. In Spanner, you can configure this to up to one week.

  - Understand commit timestamps [in GoogleSQL databases](https://docs.cloud.google.com/spanner/docs/commit-timestamp) and [in PostgreSQL databases](https://docs.cloud.google.com/spanner/docs/commit-timestamp-postgresql) .

  - [TrueTime and external consistency](https://docs.cloud.google.com/spanner/docs/true-time-external-consistency) : Understand how Spanner maintains consistency across the deployment.

### Isolation levels

Understand the different isolation levels that Spanner Omni supports to ensure data consistency.

  - [Isolation levels overview](https://docs.cloud.google.com/spanner/docs/isolation-levels) .
  - [Use repeatable read isolation](https://docs.cloud.google.com/spanner/docs/use-repeatable-read-isolation) .

### Optimization

Use these techniques to optimize your transaction performance and throughput.

  - [Throughput optimized writes](https://docs.cloud.google.com/spanner/docs/throughput-optimized-writes) .

  - [Retrieve commit statistics for a transaction](https://docs.cloud.google.com/spanner/docs/commit-statistics) .

### Locking

Learn how to use explicit locking to manage concurrent access to your data.

  - [Use `SELECT FOR UPDATE` in serializable isolation](https://docs.cloud.google.com/spanner/docs/use-select-for-update-serializable) .

  - [Use `SELECT FOR UPDATE` in repeatable read isolation](https://docs.cloud.google.com/spanner/docs/use-select-for-update-repeatable-read) .

## Data access and modification

Access and modify your data using standard Spanner methods, including the client libraries, CLI, and DML.

To learn how client libraries manage sessions, see [Sessions](https://docs.cloud.google.com/spanner/docs/sessions) . While the underlying session concepts apply to Spanner Omni, it supports [multiplexed sessions](https://docs.cloud.google.com/spanner/docs/sessions#multiplexed_sessions) only.

### Reading data

Read data from Spanner Omni using various methods, including stale reads and directed reads.

  - [Reads outside of transactions](https://docs.cloud.google.com/spanner/docs/reads) .

  - [Directed reads](https://docs.cloud.google.com/spanner/docs/directed-reads) .

  - [Read lease](https://docs.cloud.google.com/spanner/docs/read-lease) .

### Modifying data

Modify your data using DML, mutations, or the CLI.

  - [Modify data using the SQL shell](https://docs.cloud.google.com/spanner-omni/sql-shell-commands) .

  - [Insert, update, and delete data using DML](https://docs.cloud.google.com/spanner/docs/dml-tasks) .

  - [Partitioned DML](https://docs.cloud.google.com/spanner/docs/dml-partitioned) .

  - [DML best practices](https://docs.cloud.google.com/spanner/docs/dml-best-practices) .

  - [Modify data using mutations](https://docs.cloud.google.com/spanner/docs/modify-mutation-api) .

  - [Compare DML with mutations](https://docs.cloud.google.com/spanner/docs/dml-versus-mutations) .

  - [Modify data using batch write](https://docs.cloud.google.com/spanner/docs/batch-write) .

## Data types

Spanner Omni supports the following data types to represent your application's data:

  - Work with arrays [in GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/arrays) and [in PostgreSQL](https://docs.cloud.google.com/spanner/docs/reference/postgresql/arrays) .

  - [Work with `STRUCT` objects](https://docs.cloud.google.com/spanner/docs/structs) .

  - [Work with `NUMERIC` data](https://docs.cloud.google.com/spanner/docs/working-with-numerics) .

  - [Work with `JSON` data](https://docs.cloud.google.com/spanner/docs/working-with-json) .

  - [Work with `JSONB` data](https://docs.cloud.google.com/spanner/docs/working-with-jsonb) .

  - [Work with protocol buffers in GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/protocol-buffers) .

## Development and testing

Configure your development environment and application behavior to ensure reliable performance and error handling.

  - [Configure custom timeouts and retries](https://docs.cloud.google.com/spanner/docs/custom-timeout-and-retry) .

  - [Configure statement timeout](https://docs.cloud.google.com/spanner/docs/statement-timeout) .

  - [Configure transaction timeout](https://docs.cloud.google.com/spanner/docs/transaction-timeout) .

## Stream out data changes

Spanner [change streams](https://docs.cloud.google.com/spanner/docs/change-streams) track database changes like inserts, updates, and deletes in near real-time. Managed using DDL, they capture details like primary keys and commit timestamps for entire databases or specific tables. While functionality is nearly identical between Spanner and Spanner Omni, Dataflow is not supported in Spanner Omni.

To learn more, see:

  - [Change streams overview](https://docs.cloud.google.com/spanner/docs/change-streams) : Learn what change streams do and how they work.

  - [Create and manage change streams](https://docs.cloud.google.com/spanner/docs/change-streams/manage) : Learn how to use DDL to create, modify, and delete change streams.

## Full-text search

Use Spanner full-text search (FTS) to search tables for words, phrases, or numbers. FTS reads the latest committed data.

Key features of FTS include:

  - [Automatic spelling correction](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview#enhanced_query_mode) .
  - [Language detection](https://docs.cloud.google.com/spanner/docs/full-text-search/tokenization#language_detection_refinement_with_the_language_tag_argument) , including for Chinese, Japanese, and Korean.

To use FTS, create search indexes on the columns you want to search. Spanner breaks down data in these columns into individual words. It updates the index instantly when new data is added.

FTS searches offer advanced capabilities beyond standard text matching, such as:

  - [Searching for exact numbers and number ranges](https://docs.cloud.google.com/spanner/docs/full-text-search/numeric-indexes) .
  - [Finding parts of words (n-grams) for names and misspellings](https://docs.cloud.google.com/spanner/docs/full-text-search/tokenization#tokenizers) .
  - [Searching for similar-sounding words (Soundex)](https://docs.cloud.google.com/spanner/docs/full-text-search#types_of_full-text_search) .
  - [Ignoring common words](https://docs.cloud.google.com/spanner/docs/full-text-search/tokenization#tokenize_plain_text_or_html_content) .

FTS functionality is consistent between Spanner and Spanner Omni.

### Full-text differences in Spanner

Spanner supports the core FTS functionality that's in Spanner with the following differences:

  - [Enhanced query mode](https://docs.cloud.google.com/spanner/docs/full-text-search/query-overview#enhanced_query_mode) isn't supported in Spanner Omni.

  - An [instance](https://docs.cloud.google.com/spanner/docs/instances) in Spanner is equivalent to a deployment in Spanner Omni.

## What's next

  - [Use the Go client library to connect to Spanner](https://docs.cloud.google.com/spanner-omni/go) .

  - [Use the Java client library to connect to Spanner](https://docs.cloud.google.com/spanner-omni/java) .

  - [Use the Python client library to connect to Spanner](https://docs.cloud.google.com/spanner-omni/python) .
