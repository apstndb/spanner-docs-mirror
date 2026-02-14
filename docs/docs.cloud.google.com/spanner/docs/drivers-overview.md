This page describes the drivers that are supported for Spanner for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

Google supports open-source Spanner drivers for several popular object-relational mapping libraries (ORMs) and frameworks. These drivers allow the use of Spanner databases through APIs defined by those frameworks.

The following tables list all the open-source ORMs and drivers that are supported for Spanner and the features that each one supports. There is one table for each supported SQL dialect.

You can also use the Spanner client libraries to access the Spanner API. For more information, see [Spanner client libraries](/spanner/docs/reference/libraries) .

### GoogleSQL drivers and ORMs

<table>
<thead>
<tr class="header">
<th>Feature</th>
<th><a href="/spanner/docs/use-golang-database-sql">database/sql</a></th>
<th><a href="/spanner/docs/use-gorm">GORM</a></th>
<th><a href="/spanner/docs/use-oss-jdbc">Spanner JDBC</a></th>
<th><a href="/spanner/docs/use-hibernate">Hibernate</a></th>
<th><a href="/spanner/docs/adding-spring">Spring Data</a></th>
<th><a href="/spanner/docs/use-oss-r2dbc">R2DBC</a> *</th>
<th><a href="/spanner/docs/use-entity-framework">Entity Framework</a></th>
<th><a href="/spanner/docs/use-active-record">Active Record</a></th>
<th><a href="/spanner/docs/use-sqlalchemy">SQLAlchemy</a></th>
<th><a href="https://cloud.google.com/blog/topics/developers-practitioners/django-orm-support-cloud-spanner-now-generally-available">Django</a></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Language</td>
<td>Go</td>
<td>Go</td>
<td>Java</td>
<td>Java</td>
<td>Java</td>
<td>Java</td>
<td>C#</td>
<td>Ruby</td>
<td>Python</td>
<td>Python</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/standard-sql/query-syntax#statement-hints">Statement hints</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td>†</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/modify-mutation-api">Mutations</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/samples/spanner-dml-batch-update">Batch DML</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/schema-updates#order_of_execution_of_statements_in_batches">Batch DDL</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reads">Stale reads</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/schema-and-data-model#parent-child">Interleaved tables</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/dml-partitioned">Partitioned DML</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/sessions">Session labeling</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/rest/v1/RequestOptions#Priority">Request priority</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/working-with-json">JSON type</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/introspection/troubleshooting-with-tags">Request tagging</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/standard-sql/overview">Google SQL dialect</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/postgresql/overview">PostgreSQL dialect</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td>Limitations</td>
<td></td>
<td><a href="https://github.com/googleapis/go-gorm-spanner/blob/main/docs/limitations.md">View</a></td>
<td></td>
<td><a href="https://github.com/GoogleCloudPlatform/google-cloud-spanner-hibernate#cloud-spanner-hibernate-orm-limitations">View</a></td>
<td></td>
<td></td>
<td><a href="https://github.com/googleapis/dotnet-spanner-entity-framework#limitations">View</a></td>
<td><a href="https://github.com/googleapis/ruby-spanner-activerecord#limitations">View</a></td>
<td><a href="https://github.com/googleapis/python-spanner-sqlalchemy#features-and-limitations">View</a></td>
<td><a href="https://github.com/googleapis/python-spanner-django/blob/main/docs/limitations.rst">View</a></td>
</tr>
</tbody>
</table>

\* This column describes R2DBC when used specifically with Spring Data.  
† Available only when sending literal SQL queries, by using the Spring Data `  @Query  ` annotation.

### PostgreSQL drivers and ORMs

All PostgreSQL drivers require the PGAdapter proxy to be running. For more information, see [About PGAdapter](/spanner/docs/pgadapter) .

<table>
<thead>
<tr class="header">
<th>Feature</th>
<th><a href="/spanner/docs/use-oss-jdbc">Spanner JDBC</a></th>
<th><a href="/spanner/docs/pg-jdbc-connect">PostgreSQL JDBC</a></th>
<th><a href="/spanner/docs/use-hibernate-postgresql">Hibernate</a></th>
<th><a href="/spanner/docs/pg-pgx-connect">pgx</a></th>
<th><a href="/spanner/docs/use-gorm-postgresql">GORM</a></th>
<th><a href="/spanner/docs/pg-psycopg2-connect">psycopg2</a></th>
<th><a href="/spanner/docs/pg-psycopg3-connect">psycopg3</a></th>
<th><a href="/spanner/docs/use-sqlalchemy-pg">SQLAlchemy 2</a></th>
<th><a href="/spanner/docs/pg-node-postgres-connect">node-postgres</a></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Language</td>
<td>Java</td>
<td>Java</td>
<td>Java</td>
<td>Go</td>
<td>Go</td>
<td>Python</td>
<td>Python</td>
<td>Python</td>
<td>Node.js</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/standard-sql/query-syntax#statement-hints">Statement hints</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/modify-mutation-api">Mutations</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/samples/spanner-dml-batch-update">Batch DML</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/schema-updates#order_of_execution_of_statements_in_batches">Batch DDL</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reads">Stale reads</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/schema-and-data-model#parent-child">Interleaved tables</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/dml-partitioned">Partitioned DML</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/sessions">Session labeling</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/rest/v1/RequestOptions#Priority">Request priority</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/introspection/troubleshooting-with-tags">Request tagging</a></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
<tr class="even">
<td>Limitations</td>
<td></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/jdbc.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/java/hibernate/README.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/pgx.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/golang/gorm/README.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/psycopg2.md#limitations-and-known-bugs">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/psycopg3.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/samples/python/sqlalchemy2-sample/README.md#limitations">View</a></td>
<td><a href="https://github.com/GoogleCloudPlatform/pgadapter/blob/-/docs/node-postgres.md#limitations">View</a></td>
</tr>
</tbody>
</table>
