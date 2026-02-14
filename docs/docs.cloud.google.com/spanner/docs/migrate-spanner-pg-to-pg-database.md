This page describes key considerations and steps to follow when migrating from Spanner to another PostgreSQL-dialect database if you want to move an application out of Spanner or Google Cloud. You can also use information on this page if you need to understand or demonstrate thefeasibility of moving your database, for example, for *stressed exit* disaster planning.

Spanner's PostgreSQL interface is the best choice for applications that need the option to deploy to another PostgreSQL-compatible environment, either within Google Cloud or elsewhere. Using familiar syntax and standard clients from the PostgreSQL ecosystem, the PostgreSQL interface lets developers and operators use their existing PostgreSQL knowledge and skills.

It uses the same query processing, transaction coordination, distributed storage, and network infrastructure as the GoogleSQL dialect. If you need a database that supports portability, you're not compromising on Spanner's core scalability, consistency, or price-performance benefits when you select the PostgreSQL interface.

Learn more about the [differences between the PostgreSQL and GoogleSQL dialects](/spanner/docs/choose-googlesql-or-postgres) in Spanner.

At a high level, the steps are the following:

1.  Remove Spanner-specific extensions from queries and DDL statements
2.  Migrate the schema
3.  Migrate the data
4.  Migrate the application

## Spanner-specific considerations

Spanner's PostgreSQL interface supports PostgreSQL queries out of the box, so most SQL queries that run on a Spanner PostgreSQL-dialect database have the same behavior as other PostgreSQL-compatible databases. Using this approach, the number of SQL and data access changes required to move an application from one platform to another is likely low. This makes the porting process quicker, easier, and less prone to error than a similar GoogleSQL-dialect database.

In addition to broad PostgreSQL compatibility, the PostgreSQL interface offers a number of Spanner-specific extensions. If you use these extensions in your applications, you'll have to remove them or map them to PostgreSQL features manually. Some notable examples are given in [Query syntax extensions](#query-syntax-extensions) and [Schema Management (DDL) extensions](#schema-management-extensions) .

### Query syntax extensions

Spanner's PostgreSQL interface provides a number of Spanner-specific extensions. Most use the prefix `  spanner.  ` for identification. In the following table, we list these extensions and the actions that you might need to take before the same application can run on a PostgreSQL database.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<tbody>
<tr class="odd">
<td><strong>Extension kind</strong></td>
<td><strong>Specific extensions</strong></td>
<td><strong>Actions to take before migration</strong></td>
</tr>
<tr class="even">
<td>Spanner-specific functions</td>
<td><ul>
<li>Vector functions listed in the <a href="/spanner/docs/reference/postgresql/functions-and-operators#mathematical">Mathematical functions list</a> with the <code dir="ltr" translate="no">         spanner.        </code> prefix</li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#ml">Machine learning functions</a></li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#hash-functions">Hash functions</a></li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#spanner-date-time-functions">Date and time functions</a></li>
<li><a href="/spanner/docs/commit-timestamp-postgresql">Commit timestamp functions</a> .</li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#spanner-jsonb-functions">JSONB functions</a></li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#sequence">Sequence functions</a> with the <code dir="ltr" translate="no">         spanner.        </code> prefix</li>
<li><a href="/spanner/docs/reference/postgresql/functions-and-operators#utility">Utility functions</a></li>
</ul></td>
<td>Find functions prefixed with <code dir="ltr" translate="no">       spanner.      </code> and remove these calls.</td>
</tr>
<tr class="odd">
<td>Type extensions</td>
<td><ul>
<li><a href="/spanner/docs/reference/postgresql/data-types#array-extensions">Array data type</a> with the <code dir="ltr" translate="no">         VECTOR LENGTH        </code> parameter</li>
</ul></td>
<td>Remove the <code dir="ltr" translate="no">       VECTOR LENGTH      </code> syntax or consider using <a href="https://github.com/pgvector/pgvector">pgvector</a> .</td>
</tr>
<tr class="even">
<td>Query syntax</td>
<td><ul>
<li><a href="/spanner/docs/reference/postgresql/query-syntax#pg_extensions">Spanner query hints</a></li>
</ul></td>
<td>No action required as hints are represented inside comments.<br />
For details on performance considerations, see <a href="#query-migration">Query migration</a> .</td>
</tr>
<tr class="odd">
<td>Stored system procedures</td>
<td><ul>
<li><a href="/spanner/docs/reference/postgresql/stored-procedures-pg#query-cancellation">Query cancellation</a></li>
</ul></td>
<td>Remove calls to <code dir="ltr" translate="no">       spanner.cancel_query()      </code> .<br />
Optionally, you can replace the calls with a <a href="https://www.postgresql.org/docs/current/libpq-cancel.html">PostgreSQL equivalent</a> .</td>
</tr>
<tr class="even">
<td>SET/SHOW operations</td>
<td></td>
<td>Can be ignored as PostgreSQL doesn't have any built-in parameters that begin with <code dir="ltr" translate="no">       spanner.      </code> , so setting any variables with that prefix doesn't have any impact on expected behavior.</td>
</tr>
</tbody>
</table>

### Schema Management (DDL) extensions

Spanner offers a range of extensions related to data management, as described in the [data definition language (DDL)](/spanner/docs/reference/postgresql/data-definition-language) page.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><strong>Extension</strong></td>
<td><strong>Actions to take before migration</strong></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/schema-and-data-model#create-interleaved-tables">Interleaved tables</a><br />
Co-locates many-to-one related data in physical storage, making joins across them significantly more efficient.</td>
<td>Remove the <code dir="ltr" translate="no">       INTERLEAVE IN      </code> clause.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/commit-timestamp-postgresql">Commit timestamps</a><br />
Enables atomically storing the commit timestamp of a transaction into a column.</td>
<td>Either replace <code dir="ltr" translate="no">       SPANNER.COMMIT_TIMESTAMP      </code> with a PostgreSQL timestamp type and manage setting the timestamp in your application or remove that column.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/postgresql/data-definition-language#alter-database">Point-in-time recovery</a><br />
Provides protection against accidental deletion or writes.</td>
<td>Remove any DDL statements that set <code dir="ltr" translate="no">       spanner.version_retention_period      </code> .</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/ttl">Time to live</a> (TTL)<br />
Prompts the automatic deletion of records based on age.</td>
<td>Remove the <code dir="ltr" translate="no">       TTL INTERVAL      </code> clause. Consider leveraging a <code dir="ltr" translate="no">       cron      </code> or scheduled task to periodically delete outdated. rows.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/query-optimizer/manage-query-optimizer">Optimizer options</a><br />
Sets options to minimize any potential for performance regression when the query optimizer or statistics change.</td>
<td>Remove DDL statements that set optimizer options.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/change-streams/manage">Change streams</a><br />
Watches and streams out a Spanner database's data changes—inserts, updates, and deletes—in near real-time.</td>
<td>Remove any DDL statements related to change streams.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/postgresql/data-definition-language#alter-database">Default leader</a><br />
Lets you specify the leader for your database in dual- and multi-region configurations.</td>
<td>Remove any DDL statements that set <code dir="ltr" translate="no">       spanner.default_leader      </code> .</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/create-manage-data-placements">Geo-partitioning</a><br />
Lets you further segment and store rows in your database table across different instance configurations.</td>
<td>Remove any DDL statements related to geo-partitioning.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/postgresql/data-definition-language#sequence_statements">Sequences</a><br />
Spanner only supports the <code dir="ltr" translate="no">       bit_reversed_positive      </code> sequence.</td>
<td>Replace <code dir="ltr" translate="no">       bit_reversed_positive      </code> with a sequence available in PostgreSQL. Remove any DDL statements that set <code dir="ltr" translate="no">       spanner.default_sequence_kind      </code> .</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/postgresql/data-definition-language#locality-group-statements">Locality groups</a><br />
Lets you define column group strategy to store columns separately or to use <a href="/spanner/docs/tiered-storage">tiered storage</a> .</td>
<td>Remove any DDL statements related to locality groups.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/postgresql/data-definition-language#search-index-statements">Search indexes</a><br />
Lets you define indexes to perform <a href="/spanner/docs/full-text-search">full text search</a> .</td>
<td>Remove any DDL statements related to search indexes.</td>
</tr>
</tbody>
</table>

## Schema migration

You can export a PostgreSQL-dialect database schema in PostgreSQL syntax. For databases configured to use the PostgreSQL interface, you can achieve this with `  psql  ` using [PGAdapter](/spanner/docs/pgadapter) , the sidecar proxy that lets you use standard PostgreSQL drivers or client libraries to connect to Spanner:

``` text
psql -v ON_ERROR_STOP=1 \
  --host "$PGADAPTER_HOST" \
  --port "$PGADAPTER_PORT" \
  --dbname "$SPANNER_DATABASE" \
  -qAtX \
  -c "show database ddl"
```

You can also use the following `  gcloud  ` command to output the schema as a PostgreSQL-compatible SQL script:

``` text
gcloud spanner databases ddl describe databasename
```

If the database uses Spanner-specific schema extensions, like those discussed in [Schema management extensions](#schema-management-extensions) , they are listed when you run this command. You need to remove them before migrating the schema to PostgreSQL.

## Data migration

Spanner's PostgreSQL interface supports PostgreSQL's `  COPY TO STDIN  ` and `  STDOUT  ` extensions using [PGAdapter](/spanner/docs/pgadapter) . This is one way to load data into and out of Spanner. Read more about the `  COPY  ` command in the [psql command-line tool for Spanner documentation](/spanner/docs/psql-commands#copy-command) .

This script exports smaller quantities of data (recommended for less than 100GB of data) from Spanner's PostgreSQL interface into the new PostgreSQL database:

``` text
psql -h pgadapter-host -c "COPY $TABLE TO STDOUT BINARY" | \
psql -h postgresql-host -c "COPY $TABLE FROM STDIN BINARY"
```

For larger tables (greater than or equal to100GB of data), you can launch a [Dataflow export to a CSV template](/spanner/docs/import-export-csv#export-to-csv) .

You can perform live data migrations using the Debezium Kafka connector to [stream Spanner updates](https://cloud.google.com/blog/products/databases/stream-data-changes-from-spanner-at-scale-with-apache-kafka) into PostgreSQL. You can customize it further if you use the [Spanner Change Streams API](/spanner/docs/change-streams/details#query) to access the Change Data Capture (CDC) streams directly.

## Query migration

The PostgreSQL interface for Spanner implements much of the most common PostgreSQL query syntax, functions, and operators.

If you are using hints in your queries, you don't need to re-write your queries because query hints on Spanner are defined in PostgreSQL-compatible comments:

``` text
SELECT s.FirstName, s.LastName,
 s.SingerInfo, a.AlbumTitle, a.Charts
FROM Singers AS s
LEFT OUTER JOIN/*@JOIN_METHOD=APPLY_JOIN*/ Albums AS a
 ON s.SingerId = a.SingerId;
```

These comments are processed by Spanner's query planner, but a PostgreSQL database ignores these, so you can either include them or remove them.

To achieve optimal performance in the new environment, queries and database schema (such as indexes) might need optimization for the new environment. We recommend that you run benchmark checks to confirm this empirically.

## Application migration

When it comes to connectivity from your applications, your migration strategy depends on the initial choices made when configuring your application to use Spanner, such as whether you use PostgreSQL drivers, Spanner drivers, or Spanner client libraries. This section describes considerations for each option.

### PostgreSQL drivers

Spanner supports common PostgreSQL clients using PGAdaper, a lightweight proxy that translates the PostgreSQL wire protocol into Spanner's low-level gRPC query APIs. If you are using one of these, the change to a different PostgreSQL target involves updating your connection string to point directly to the new PostgreSQL database instead of the PGAdapter proxy. This approach provides good performance and strong compatibility, so it's a good fit for when portability is a top concern. Most queries that run on Spanner's PostgreSQL interface works the same in other PostgreSQL environments. However, the opposite isn't necessarily true; PostgreSQL supports syntax and features that Spanner doesn't support.

### Spanner drivers

These drivers are Spanner-specific implementations for [common languages and application frameworks](/spanner/docs/drivers-overview) . For example, the Spanner JDBC (Java) driver implements the same API that the PostgreSQL JDBC driver implements, so applications that use the Spanner JDBC driver can update their build process to link in the equivalent built-in PostgreSQL driver when they want to run the application with PostgreSQL. This option is best if you are already using Spanner or if you are seeking a performant solution that leverages Spanner features like the [Mutations API](/spanner/docs/modify-mutation-api#java) , which wouldn't be exposed using a built-in PostgreSQL driver. If you need full compatibility with built-in drivers and value portability, you should instead consider using PostgreSQL's built-in drivers with PGAdapter to ensure some level of application portability.

For more information, see [PostgreSQL drivers and ORMs](/spanner/docs/drivers-overview#postgresql_drivers_and_orms) .

### Spanner client libraries

Spanner also offers various [idiomatic client libraries](/spanner/docs/reference/libraries) that provide direct access to Spanner without implementing or going through a PostgreSQL-standardized interface. These clients provide maximum access to Spanner-specific features, but are not API-compatible with PostgreSQL drivers. These options offer the highest level of feature performance, but are less portable than the aforementioned options.

## What's next

  - Learn how to [choose between PostgreSQL and GoogleSQL](/spanner/docs/choose-googlesql-or-postgres) .
  - Follow the [quickstart](/spanner/docs/create-query-database-console) to create and interact with a PostgreSQL database.
  - Learn more about [Spanner's PostgreSQL language support](/spanner/docs/reference/postgresql/overview) .
  - Learn about [PGAdapter](/spanner/docs/pgadapter) .
  - Learn about the [PGAdapter GitHub repository](https://github.com/GoogleCloudPlatform/pgadapter) .
  - Review [known issues](/spanner/docs/known-issues-postgresql-interface) in the PostgreSQL interface.
