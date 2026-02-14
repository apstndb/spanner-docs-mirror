Google storage technologies power some of the world's largest applications. However, scale is not always an automatic result of using these systems. Designers must think carefully about how to model their data to ensure that their application can scale and perform as it grows in various dimensions.

Spanner is a *distributed database* , and using it effectively requires thinking differently about schema design and access patterns than you might with other databases. Distributed systems, by their nature, force designers to think about data and processing locality.

Spanner supports SQL queries and transactions with the ability to scale out horizontally. Careful design is often necessary to realize Spanner's full benefit. This paper discusses some of the key ideas that will help you to ensure that your application can scale to arbitrary levels, and to maximize its performance. Two tools in particular have a great impact on scalability: key definition and interleaving.

## Table layout

Rows in a Spanner table are organized lexicographically by `  PRIMARY KEY  ` . Conceptually, keys are ordered by the concatenation of the columns in the order that they are declared in the `  PRIMARY KEY  ` clause. This exhibits all the standard properties of locality:

  - Scanning the table in lexicographic order is efficient.
  - Sufficiently close rows will be stored in the same disk blocks, and will be read and cached together.

Spanner replicates your data across multiple [zones](/docs/geography-and-regions) for availability and scale. Each zone holds a complete replica of your data. When you provision a Spanner instance node, you specify its [compute capacity](/spanner/docs/compute-capacity) . The compute capacity is the amount of compute resource allocated to your instance in each of these zones. While each replica is a complete set of your data, data within a replica is partitioned across the compute resources in that zone.

Data within each Spanner replica is organized into two levels of physical hierarchy: *database splits* , then *blocks* . Splits hold contiguous ranges of rows, and are the unit by which Spanner distributes your database across compute resources. Over time, splits may be broken into smaller parts, merged, or moved to other nodes in your instance to increase parallelism and allow your application to scale. Operations that span splits are more expensive than equivalent operations that don't, due to increased communication. This is true even if those splits happen to be served by the same node.

There are two types of tables in Spanner: *root tables* (sometimes called top-level tables), and *interleaved tables* . Interleaved tables are defined by specifying another table as its *parent* , causing rows in the interleaved table to be clustered with the parent row. Root tables have no parent, and each row in a root table defines a new top-level row, or *root row* . Rows interleaved with this root row are called *child rows* , and the collection of a root row plus all its descendants is called a *row tree* . The parent row must exist before you can insert child rows. The parent row can either already exist in the database or can be inserted before the insertion of the child rows in the same transaction.

Spanner automatically partitions splits when it deems necessary due to size or load. To preserve data locality, Spanner prefers adding split boundaries as close as to the *root tables* , so that any given row tree can be kept in a single split. This means that operations within a row tree tend to be more efficient because they are unlikely to require communication with other splits.

However, if there is hotspot in a child row, Spanner will attempt to add split boundaries to *interleaved tables* in order to isolate that hotspot row, along with all child rows below it.

Choosing which tables should be roots is an important decision in designing your application to scale. Roots are typically things like Users, Accounts, Projects and the like, and their child tables hold most of the other data about the entity in question.

### Recommendations:

  - Use a common key prefix for related rows in the same table to improve locality.
  - Interleave related data into another table whenever it makes sense.

## Tradeoffs of locality

If data is frequently written or read together, it can benefit both latency and throughput to cluster them by carefully selecting primary keys and using interleaving. This is because there is a fixed cost to communicating to any server or disk block, so why not get as much as possible while there? Furthermore, the more servers that you communicate with, the higher the chance that you are going to encounter a temporarily busy server, increasing tail latencies. Finally, transactions that span splits, while automatic and transparent in Spanner, have a slightly higher CPU cost and latency due to the distributed nature of two-phase commit.

On the flip side, if data is related but not frequently accessed together, consider going out of your way to separate them. This has the most benefit when the infrequently accessed data is large. For example, many databases store large binary data out-of-band from the primary row data, with only references to the large data interleaved.

Note that some level of two-phase commit and non-local data operations are unavoidable in a distributed database. Don't become overly concerned with getting a perfect locality story for every operation. Focus on getting the desired locality for the most important root entities and most common access patterns, and let less frequent or less performance sensitive distributed operations happen when they need to. Two-phase commit and distributed reads are there to help simplify schemas and ease programmer toil: in all but the most performance-critical use cases, it is better to let them.

### Recommendations:

  - Organize your data into hierarchies such that data read or written together tends to be nearby.
  - Consider storing large columns in non-interleaved tables if less frequently accessed.

## Index options

[Secondary indexes](/spanner/docs/secondary-indexes) allow you to quickly find rows by values other than the primary key. Spanner supports both non-interleaved and interleaved indexes. Non-interleaved indexes are the default and the type most analogous to what is supported in other RDBMS . They don't place any restrictions over the columns being indexed and, while powerful, they are not always the best choice. Interleaved indexes must be defined over columns that share a prefix with the parent table, and allow greater control of locality.

Spanner stores index data in the same way as tables, with one row per index entry. Many of the design considerations for tables also apply to indexes. Non-interleaved indexes store data in root tables. Because root tables can be split between any root row, this ensures that non-interleaved indexes can scale to arbitrary size and, ignoring hot spots, to almost any workload. Unfortunately it also means that the index entries are usually not in the same splits as the primary data. This creates extra work and latency for any writing process, and adds additional splits to consult at read time.

Interleaved indexes, by contrast, store data in interleaved tables. They are suitable when you are searching within the domain of a single entity. Interleaved indexes force data and index entries to remain in the same row tree, making joins between them far more efficient. Examples of uses for an interleaved index:

  - Accessing your photos by various sort orders like taken date, last modified date, title, album, etc.
  - Finding all your posts that have a particular set of tags.
  - Finding my previous shopping orders that contained a specific item.

### Recommendations:

  - Use non-interleaved indexes when you need to find rows from anywhere in your database.
  - Prefer interleaved indexes whenever your searches are scoped to a single entity.

## STORING index clause

Secondary indexes allow you to find rows by attributes other than the primary key. If all the data requested is in the index itself, it can be consulted on its own without reading the primary record. This can save significant resources as no join is required.

Unfortunately, index keys are limited to 16 in number and 8 KiB in aggregate size, restricting what can be put in them. To compensate for these limitations, Spanner has the ability to store extra data in any index, using the `  STORING  ` clause. `  STORING  ` a column in an index results in its values being duplicated, with a copy stored in the index. You can think of an index with `  STORING  ` as a simple single table materialized view (views are not natively supported in Spanner at this time).

Another useful application of `  STORING  ` is as part of a `  NULL_FILTERED  ` index. This lets you define what is effectively a materialized view of a sparse subset of a table that you can scan efficiently. For example, you might create such an index on the `  is_unread  ` column of a mailbox to be able to serve the unread messages view in a single table scan, but without paying for a complete copy of every mailbox.

### Recommendations:

  - Make prudent use of `  STORING  ` to tradeoff read time performance against storage size and write time performance.
  - Use `  NULL_FILTERED  ` to control storage costs of sparse indexes.

## Anti-patterns

### Anti-pattern: timestamp ordering

Many schema designers are inclined to define a root table that is timestamp ordered, and updated on every write. Unfortunately, this is one of the least scalable things that you can do. The reason is that this design results in a huge **hot spot** at the end of the table that can't easily be mitigated. As write rates increase, so do RPCs to a single split, as do lock contention events and other problems. Often these sorts of problems don't appear in small load tests, and instead appear after the application has been in production for some time. By then, it's too late\!

If your application absolutely must include a log that is timestamp ordered, consider if you can make the log local by interleaving it in one of your other root tables. This has the benefit of distributing the hot spot over many roots. But you still need to be careful that each distinct root has sufficiently low write rate.

If you need a global (cross root) timestamp ordered table, and you need to support higher write rates to that table than a single node is capable of, use application-level *sharding* . Sharding a table means partitioning it into some number N of roughly equal divisions called shards. This is typically done by prefixing the original primary key with an additional `  ShardId  ` column holding integer values between `  [0, N)  ` . The `  ShardId  ` for a given write is typically selected either at random, or by hashing a part of the base key. Hashing is often preferred because it can be used to ensure all records of a given type go into the same shard, improving performance of retrieval. Either way, the goal is to ensure that, over time, writes are distributed across all shards equally. This approach sometimes means that reads need to scan all shards to reconstruct the original total ordering of writes.

#### Recommendations:

  - Avoid high write-rate timestamp ordered tables and indexes **at all cost** .
  - Use some technique to spread hot spots, be it interleaving in another table or sharding.

### Anti-pattern: sequences

Application developers love using database sequences (or auto-increment) to generate primary keys. Unfortunately, this habit from the RDBMS days (called surrogate keys) is almost as harmful as the timestamp ordering anti-pattern described above. The reason is that database sequences tend to emit values in a quasi-monotonic way, over time, to producing values that are clustered near each other. This typically produces hot spots when used as primary keys, especially for root rows.

Contrary to RDBMS conventional wisdom, we recommend that you use real-world attributes for primary keys whenever it makes sense. This is particularly the case if the attribute is never going to change.

If you want to generate numerical unique primary keys, aim to get the high order bits of subsequent numbers to be distributed roughly equally over the entire number space. One trick is to generate sequential numbers by conventional means, and then bit-reversing to obtain a final value. Alternatively you could look into a [UUID](/spanner/docs/reference/standard-sql/data-types#uuid_type) generator, but be careful: not all UUID functions are created equally, and some store the timestamp in the high order bits, effectively defeating the benefit. Make sure your UUID generator pseudo-randomly chooses high order bits.

#### Recommendations:

  - Avoid using incrementing sequence values as primary keys. Instead, bit-reverse a sequence value, or use a carefully chosen UUID.
  - Use real-world values for primary keys rather than surrogate keys.
