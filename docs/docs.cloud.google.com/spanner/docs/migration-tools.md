We recommend using the following tools to assist you in various stages of your Spanner migration, depending on your source database and other factors. Some tools only support certain source databases. For some steps of the migration process, no tool is available, so you must complete those steps manually.

  - [Spanner migration tool (SMT)](https://github.com/GoogleCloudPlatform/spanner-migration-tool) is an open-source tool that performs assessments, schema conversion, and data migrations. For information on setting up the Spanner migration tool, see [Set up Spanner migration tool](/spanner/docs/set-up-spanner-migration-tool) .

  - [Datastream](/datastream/docs/overview) is a Google Cloud service that lets you read change data capture (CDC) events and bulk data from a source database and write to a specified destination.

  - [Bulk data migration](/dataflow/docs/guides/templates/provided/sourcedb-to-spanner) is a Dataflow template that lets you migrate large MySQL data sets directly to Spanner.

  - [Live data migration](https://googlecloudplatform.github.io/spanner-migration-tool/minimal) uses Datastream and Dataflow to migrate:
    
      - Existing data in your source database.
      - Stream of changes that are made to your source database during the migration.

  - [Data Validation Tool (DVT)](https://github.com/GoogleCloudPlatform/professional-services-data-validator) is a standardized data validation method built by Google and supported by the open source community. You can integrate DVT into existing Google Cloud products.

  - [Database Migration Assessment (DMA)](https://googlecloudplatform.github.io/database-assessment/) offers a basic assessment to migrate MySQL and PostgreSQL to Spanner.

### Migration tools for MySQL source databases

If your source database is MySQL, then you can perform some of the initial migration stages using MySQL dump files. You must directly connect to your running source MySQL database to complete a production migration.

**Note:** If your source database is not a MySQL or PostgreSQL and you have a sample dump file in a CSV, Avro, or Parquet file format, you can [load the file to BigQuery](/bigquery/docs/loading-data-cloud-storage-csv) and [copy to Spanner using reverse ETL](/bigquery/docs/export-to-spanner) . For all other migration stages, you need to evaluate custom solutions.

The following table recommends migration tools based on the migration stage and whether you're using a dump file or directly connecting your source database:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Migration stage</strong></th>
<th><strong>Dump file</strong></th>
<th><strong>Direct connection to source database</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Assessment</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc/mysql.html#using-spanner-migration-tool-with-mysqldump">SMT</a> with <code dir="ltr" translate="no">       mysqldump      </code> .</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc/mysql.html#using-spanner-migration-tool-with-mysqldump">SMT</a> with <code dir="ltr" translate="no">       mysqldump      </code> .</td>
</tr>
<tr class="even">
<td>Schema conversion</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc/mysql.html#using-spanner-migration-tool-with-mysqldump">SMT</a> with <code dir="ltr" translate="no">       mysqldump      </code> .</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/ui/schema-conv">SMT</a> to configure and convert schema.</td>
</tr>
<tr class="odd">
<td>Sample data load</td>
<td><ul>
<li>If your sample dump file is less than 100 GB, use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc">SMT</a> in POC mode.</li>
<li>If your sample dump file is more than 100 GB, export the file to <a href="/sql/docs/mysql/import-export/import-export-sql">Cloud SQL</a> and perform a <a href="/dataflow/docs/guides/templates/provided/sourcedb-to-spanner">bulk migration</a> .</li>
<li>If your sample dump file is in a CSV, Avro, or Parquet file format, <a href="/spanner/docs/load-sample-data">load the file to BigQuery and copy to Spanner</a> using reverse ETL.</li>
</ul></td>
<td>Perform a <a href="/dataflow/docs/guides/templates/provided/sourcedb-to-spanner">bulk migration</a> .</td>
</tr>
<tr class="even">
<td>Data migration</td>
<td>Not applicable</td>
<td>Perform a <a href="/dataflow/docs/guides/templates/provided/sourcedb-to-spanner">bulk migration</a> , then perform a <a href="https://googlecloudplatform.github.io/spanner-migration-tool/minimal">minimal downtime migration</a> .</td>
</tr>
<tr class="odd">
<td>Data validation</td>
<td>Not applicable</td>
<td>Use <a href="https://github.com/GoogleCloudPlatform/professional-services-data-validator">DVT</a> .</td>
</tr>
<tr class="even">
<td>Cutover and fallback configuration</td>
<td>Not applicable</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/reverse-replication">SMT</a> for reverse replication.</td>
</tr>
</tbody>
</table>

### Migration tools for PostgreSQL source databases

If your source database uses PostgreSQL, then you can perform some of the migration stages using a PostgreSQL dump file. You must directly connect to your running source PostgreSQL database to complete the migration.

The following table recommends migration tools based on the migration stage and whether you're working with a dump file or directly connecting from your source database:

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th><strong>Migration stage</strong></th>
<th><strong>Dump file</strong></th>
<th><strong>Direct connection to source database</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Assessment</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc/postgres.html#using-spanner-migration-tool-with-pg_dump">SMT</a> with <code dir="ltr" translate="no">       pg_dump      </code> .</td>
<td>Use <a href="https://googlecloudplatform.github.io/database-assessment/">DMA</a> .</td>
</tr>
<tr class="even">
<td>Schema conversion</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc/postgres.html#using-spanner-migration-tool-with-pg_dump">SMT</a> with <code dir="ltr" translate="no">       pg_dump      </code> .</td>
<td>Use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/ui/schema-conv">SMT</a> to configure and convert schema.</td>
</tr>
<tr class="odd">
<td>Sample data load</td>
<td><ul>
<li>If your sample dump file is less than 100GB, use <a href="https://googlecloudplatform.github.io/spanner-migration-tool/poc">SMT</a> in POC mode.</li>
<li>If your sample dump file is more than 100GB, export the file to <a href="/sql/docs/mysql/import-export/import-export-sql">Cloud SQL</a> and perform a <a href="https://googlecloudplatform.github.io/spanner-migration-tool/minimal">minimal downtime migration</a> .</li>
<li>If your sample dump file is in a CSV, Avro, or Parquet file format, <a href="/spanner/docs/load-sample-data">load the file to BigQuery and copy to Spanner</a> using reverse ETL.</li>
</ul></td>
<td>Perform a <a href="https://googlecloudplatform.github.io/spanner-migration-tool/minimal">minimal downtime migration</a> .</td>
</tr>
<tr class="even">
<td>Data migration</td>
<td>Not applicable</td>
<td>Perform a <a href="https://googlecloudplatform.github.io/spanner-migration-tool/minimal">minimal downtime migration</a> .</td>
</tr>
<tr class="odd">
<td>Data validation</td>
<td>Not applicable</td>
<td>Use <a href="https://github.com/GoogleCloudPlatform/professional-services-data-validator">DVT</a> .</td>
</tr>
<tr class="even">
<td>Cutover and fallback configuration</td>
<td>Not applicable</td>
<td>Not applicable</td>
</tr>
</tbody>
</table>

### Migration tools for a Cassandra-source databases

**Preview — Cassandra**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

Spanner offers a Cassandra-compatible interface that supports near-zero application code changes when migrating from Cassandra to Spanner. For more information about compatibility details, see the [Cassandra overview](/spanner/docs/non-relational/spanner-for-cassandra-users) .

The following table recommends migration tools based on the migration stage:

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Migration stage</th>
<th>Recommended tool or process</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Assessment</td>
<td>Not applicable</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#convert-schema-data-model">Schema conversion</a></td>
<td><a href="https://github.com/cloudspannerecosystem/spanner-cassandra-schema-tool">Schema conversion tool</a></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#bulk-export-data-from-cassandra">Data migration</a></td>
<td><ul>
<li><a href="/dataflow/docs/guides/templates/provided/sourcedb-to-spanner">Bulk migrate: Use the Sourcedb to Spanner template</a></li>
<li>Live data: Use the <a href="https://github.com/GoogleCloudPlatform/spanner-migration-tool/tree/master/sources/cassandra">DataStax ZDM Proxy</a> .</li>
</ul></td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#data-validation">Data validation</a></td>
<td><ul>
<li><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#compare-rows">Validate data row count</a></li>
<li><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#compare-data">Validate data row content</a></li>
</ul>
<p>For large scale databases (&gt;10 million rows): build your own tooling. For more information, see <a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#tips">Tips to validate Cassandra using row matching</a> .</p>
<p>For small scale databases (&lt;10 million rows), see the <a href="https://github.com/GoogleCloudPlatform/spanner-migration-tool/tree/master/sources/cassandra/validations">sample validation scripts</a> in GitHub.</p></td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/non-relational/migrate-from-cassandra-to-spanner#reverse-replication">Failover configuration</a></td>
<td>Use the <a href="https://googlecloudplatform.github.io/spanner-migration-tool/reverse-replication">Spanner migration tool CLI (SMT)</a> in GitHub for reverse replication.</td>
</tr>
</tbody>
</table>
