This page describes production quotas and limits for Spanner. Quota and limit might be used interchangeably in the Google Cloud console.

The quota and limit values are subject to change.

## Permissions to check and edit quotas

To view your quotas, you must have the [`  serviceusage.quotas.get  `](/iam/docs/roles-permissions/serviceusage#serviceusage.quotas.get) Identity and Access Management (IAM) permission.

To change your quotas, you must have the [`  serviceusage.quotas.update  `](/iam/docs/roles-permissions/serviceusage#serviceusage.quotas.update) IAM permission. This permission is included by default for the following [predefined roles](/iam/docs/understanding-roles) : Owner, Editor, and Quota Administrator.

These permissions are included by default in the [basic IAM roles](/iam/docs/roles-overview#basic) Owner and Editor, and in the predefined [Quota Administrator role](/iam/docs/roles-permissions/servicemanagement) .

## Check your quotas

To check the current quotas for resources in your project, use the Google Cloud console:

## Increase your quotas

**Note:** You can apply to increase your node limits. All other limits are hard limits and cannot be adjusted.

As your use of Spanner expands over time, your quotas can increase accordingly. If you expect a notable upcoming increase in usage, you should make your request a few days in advance to verify that your quotas are adequately sized.

You might also need to increase your consumer quota override. For more information, see [Creating a consumer quota override](/service-usage/docs/manage-quota#create_consumer_quota_override) .

You can increase your current Spanner instance configuration node limit by using the Google Cloud console.

1.  Go to the **Quotas** page.

2.  Select **Spanner API** in the **Service** drop-down list.
    
    If you don't see **Spanner API** , the Spanner API has not been enabled. For more information, see [Enabling APIs](/apis/docs/getting-started#enabling_apis) .

3.  Select the quotas that you want to change.

4.  Click **Edit Quotas** .

5.  In the **Quota changes** panel that appears, enter your new quota limit.

6.  Click **Done** , then **Submit request** .
    
    If you're unable to increase your node limit to your target limit manually, click **apply for higher quota** . Fill out the form to submit a request to the Spanner team. You will receive a response within 48 hours of your request.

### Increase your quota for a custom instance configuration

You can increase the node quota for your [custom instance configuration](/spanner/docs/instance-configurations#configuration) .

1.  Check the node limit of a custom instance configuration by checking the node limit of the base instance configuration.
    
    Use the [show instance configurations detail](/spanner/docs/create-manage-configurations#show-details) command if you don't know or remember the base configuration of your custom instance configuration.

2.  If the node limit required for your custom instance configuration is less than 85, follow the instructions in the previous [Increase your quotas](#increase-quotas) section. Use the Google Cloud console to increase the node limit of the *base instance configuration* associated with your custom instance configuration.
    
    If the node limit required for your custom instance configuration is more than 85, fill out the [Request a Quota Increase for your Spanner Nodes](https://docs.google.com/forms/d/e/1FAIpQLSczQOE6S_1MUTf4KBpF_i-cJVMQloUEZQ71KcNQzbAkWDDuVw/viewform) form. Specify the ID of your *custom instance configuration* in the form.

## Node limits

<table style="width:60%;">
<colgroup>
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Nodes per instance configuration</td>
<td><p>Default limits vary by project and instance configuration. To change project quota limits or request a limit increase, see <a href="/spanner/quotas#increase-quotas">Increase your quotas</a> .</p></td>
</tr>
</tbody>
</table>

## Instance limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Instance ID length</td>
<td>2 to 64 characters</td>
</tr>
</tbody>
</table>

### Free trial instance limits

A [Spanner free trial instance](/spanner/docs/free-trial-instance) has the following additional limits. To raise or remove these limits, upgrade your free trial instance to a paid instance.

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Storage capacity</td>
<td>10 GiB</td>
</tr>
<tr class="even">
<td>Database limit</td>
<td>Create up to five databases</td>
</tr>
<tr class="odd">
<td>Unsupported features</td>
<td><a href="/spanner/docs/backup">Backup and restore</a></td>
</tr>
<tr class="even">
<td>SLA</td>
<td>No SLA</td>
</tr>
<tr class="odd">
<td>Trial duration</td>
<td>90-day free trial period</td>
</tr>
</tbody>
</table>

### Geo-partitioning limits

**Preview — [Geo-partitioning](/spanner/docs/geo-partitioning)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of partitions per instance</td>
<td>20</td>
</tr>
<tr class="even">
<td>Maximum number of placements per database</td>
<td>50</td>
</tr>
<tr class="odd">
<td>Maximum number of placement rows per node in your partition</td>
<td>100 million</td>
</tr>
</tbody>
</table>

### Saved queries limits

**Preview — [Saved queries](/spanner/docs/saved-queries)**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of saved queries per project (including saved queries for other Google Cloud products)</td>
<td>10,000</td>
</tr>
<tr class="even">
<td>Maximum size for each query</td>
<td>1 MiB</td>
</tr>
</tbody>
</table>

## Instance configuration limits

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum custom instance configurations per project</td>
<td>100</td>
</tr>
<tr class="even">
<td>Custom instance configuration ID length</td>
<td><p>8 to 64 characters</p>
<p>A custom instance configuration ID must start with <code dir="ltr" translate="no">        custom-       </code></p></td>
</tr>
</tbody>
</table>

## Database limits

<table style="width:60%;">
<colgroup>
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Databases per instance</td>
<td><ul>
<li>For instances of 1 node (1000 processing units) and larger: 100 databases</li>
<li>For instances smaller than 1 node: 10 databases per 100 processing units</li>
</ul></td>
</tr>
<tr class="even">
<td>Roles per database</td>
<td>100</td>
</tr>
<tr class="odd">
<td>Database ID length</td>
<td>2 to 30 characters</td>
</tr>
<tr class="even">
<td>Storage size <sup><a href="#note1">1</a></sup></td>
<td><ul>
<li>For instances of 1 node (1000 processing units) and larger: 10 TiB per node</li>
<li>For instances smaller than 1 node: 1024.0 GiB per 100 processing units</li>
</ul>
<p>Increased storage capacity of 10 TiB per node is available in most regional, dual-region, and multi-region Spanner instance configurations. For more information, see <a href="/spanner/docs/performance#improved-performance">Performance and storage improvements</a> .</p>
<p>If you use <a href="/spanner/docs/tiered-storage">tiered storage</a> , then you can use a combined storage (SSD and HDD) up to 10 TiB per node.</p>
<p><a href="/spanner/docs/backup">Backups</a> are stored separately and don't count towards this limit. For more information, see <a href="/spanner/docs/storage-utilization">Storage utilization metrics</a> .</p>
<p>Note that Spanner bills for actual storage utilized within an instance, and not its total available storage.</p></td>
</tr>
</tbody>
</table>

## Backup and restore limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Number of ongoing create backup operations per database</td>
<td>1 (applies to on-demand backups only). <a href="/spanner/docs/backup#backup-schedules">Backup schedules</a> have their own frequency constraints and aren't subject to this limit.</td>
</tr>
<tr class="even">
<td>Number of ongoing restore database operations per instance for the restored database, not the backup.</td>
<td>10</td>
</tr>
<tr class="odd">
<td>Maximum retention time of backup</td>
<td>1 year, including the extra day in a leap year.</td>
</tr>
</tbody>
</table>

## Schema limits

### Schema objects

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>The total number of schema objects within all databases in the same instance</td>
<td>Default limits vary by the instance configuration <sup><a href="#note2">2</a></sup></td>
</tr>
</tbody>
</table>

### DDL statements

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/data-definition-language">DDL</a> statement size for a single schema change</td>
<td>10 MiB</td>
</tr>
<tr class="even">
<td>DDL statement size for a database's entire schema, as returned by <a href="/spanner/docs/reference/rpc/google.spanner.admin.database.v1#google.spanner.admin.database.v1.DatabaseAdmin.GetDatabaseDdl"><code dir="ltr" translate="no">        GetDatabaseDdl       </code></a></td>
<td>10 MiB</td>
</tr>
</tbody>
</table>

### Graphs

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Property graphs per database</td>
<td>16</td>
</tr>
<tr class="even">
<td>Property graph name length</td>
<td>1 to 128 characters</td>
</tr>
</tbody>
</table>

### Tables

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Tables per database</td>
<td>5,000</td>
</tr>
<tr class="even">
<td>Table name length</td>
<td>1 to 128 characters</td>
</tr>
<tr class="odd">
<td>Columns per table</td>
<td>1,024</td>
</tr>
<tr class="even">
<td>Column name length</td>
<td>1 to 128 characters</td>
</tr>
<tr class="odd">
<td>Maximum size of data per cell</td>
<td>10 MiB</td>
</tr>
<tr class="even">
<td>Size of a <a href="/spanner/docs/reference/standard-sql/data-types#string_type"><code dir="ltr" translate="no">        STRING       </code></a> cell</td>
<td>2,621,440 Unicode characters</td>
</tr>
<tr class="odd">
<td>Number of columns in a table key</td>
<td><p>16</p>
<p>Includes key columns shared with any parent table</p></td>
</tr>
<tr class="even">
<td>Table interleaving depth</td>
<td><p>7</p>
<p>A top-level table with child table(s) has depth 1.</p>
<p>A top-level table with grandchild tables has depth 2, and subsequent nested tables increase the depth accordingly.</p></td>
</tr>
<tr class="odd">
<td>Maximum size of a primary key or index key per row</td>
<td><p>8 KiB</p>
<p>Includes the size of all columns that make up the key</p></td>
</tr>
<tr class="even">
<td>Total size of non-key columns per row</td>
<td><p>1600 MiB</p>
<p>Includes the size of all non-key columns per row for a table</p></td>
</tr>
</tbody>
</table>

### Indexes

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Indexes per database</td>
<td>10,000</td>
</tr>
<tr class="even">
<td>Indexes per table</td>
<td>128</td>
</tr>
<tr class="odd">
<td>Index name length</td>
<td>1 to 128 characters</td>
</tr>
<tr class="even">
<td>Number of columns in an index key</td>
<td><p>16</p>
<p>The number of indexed columns (except for STORING columns) plus the number of primary key columns in the base table</p></td>
</tr>
</tbody>
</table>

### Views

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Views per database</td>
<td>5,000</td>
</tr>
<tr class="even">
<td>View name length</td>
<td>1 to 128 characters</td>
</tr>
<tr class="odd">
<td>Nesting depth</td>
<td><p>10</p>
<p>A view that refers to another view has nesting depth 1. A view that refers to another view that refers to yet another view has nesting depth 2, and so on.</p></td>
</tr>
</tbody>
</table>

### Locality groups

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Maximum number of <a href="/spanner/docs/create-manage-locality-groups">locality groups</a> per database</td>
<td>16 (1 default locality group and 15 optional additional locality groups)</td>
</tr>
<tr class="even">
<td>Minimum amount of time required in the <code dir="ltr" translate="no">       ssd_to_hdd_spill_timespan      </code> option</td>
<td>1 hour</td>
</tr>
<tr class="odd">
<td>Maximum amount of time allowed in the <code dir="ltr" translate="no">       ssd_to_hdd_spill_timespan      </code> option</td>
<td>365 days</td>
</tr>
</tbody>
</table>

## Query limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Columns in a <code dir="ltr" translate="no">       GROUP BY      </code> clause</td>
<td>1,000</td>
</tr>
<tr class="even">
<td>Values in an <code dir="ltr" translate="no">       IN      </code> operator</td>
<td>10,000</td>
</tr>
<tr class="odd">
<td>Function calls</td>
<td>1,000</td>
</tr>
<tr class="even">
<td>Joins</td>
<td>20</td>
</tr>
<tr class="odd">
<td>Nested function calls</td>
<td>75</td>
</tr>
<tr class="even">
<td>Nested <code dir="ltr" translate="no">       GROUP BY      </code> clauses</td>
<td>35</td>
</tr>
<tr class="odd">
<td>Nested subquery expressions</td>
<td>25</td>
</tr>
<tr class="even">
<td>Nested subselect statements</td>
<td>60</td>
</tr>
<tr class="odd">
<td>Joins produced by a graph query</td>
<td>100</td>
</tr>
<tr class="even">
<td>Parameters</td>
<td>950</td>
</tr>
<tr class="odd">
<td>Query statement length</td>
<td>1 million characters</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       STRUCT      </code> fields</td>
<td>1,000</td>
</tr>
<tr class="odd">
<td>Subquery expression children</td>
<td>50</td>
</tr>
<tr class="even">
<td>Unions in a query</td>
<td>200</td>
</tr>
<tr class="odd">
<td>Depth of graph <a href="/spanner/docs/graph/queries-overview#quantified-path-patterns">quantified path</a> traversal</td>
<td>100</td>
</tr>
</tbody>
</table>

## Limits for creating, reading, updating, and deleting data

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Commit size (including indexes and change streams)</td>
<td>100 MiB</td>
</tr>
<tr class="even">
<td>Concurrent reads per session</td>
<td>100</td>
</tr>
<tr class="odd">
<td>Mutations per commit (including indexes) <sup><a href="#note3">3</a> ,</sup> <sup><a href="#note10">10</a></sup></td>
<td>80,000</td>
</tr>
<tr class="even">
<td>Mutations per mutation group in a <a href="/spanner/docs/batch-write">batch write</a> request</td>
<td>80,000</td>
</tr>
<tr class="odd">
<td>Concurrent <a href="/spanner/docs/dml-partitioned">partitioned DML</a> statements per database</td>
<td>20,000</td>
</tr>
</tbody>
</table>

## Administrative limits

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Administrative actions request size <sup><a href="#note4">4</a></sup></td>
<td>1 MiB</td>
</tr>
<tr class="even">
<td>Rate limit for administrative actions <sup><a href="#note5">5</a></sup></td>
<td><p>5 per second per project per user</p>
<p>(averaged over 100 seconds)</p></td>
</tr>
</tbody>
</table>

## Request limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Request size other than for commits <sup><a href="#note6">6</a></sup></td>
<td>10 MiB</td>
</tr>
</tbody>
</table>

## Change stream limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/change-streams">Change streams</a> per database</td>
<td>10</td>
</tr>
<tr class="even">
<td>Change streams watching any given non-key column <sup><a href="#note7">7</a></sup></td>
<td>3</td>
</tr>
<tr class="odd">
<td>Concurrent readers per change stream data partition <sup><a href="#note8">8</a></sup></td>
<td>20</td>
</tr>
</tbody>
</table>

## Data Boost limits

<table>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Concurrent Data Boost requests per project in us-central1</td>
<td>1000 <sup><a href="#note9">9</a></sup></td>
</tr>
<tr class="even">
<td>Concurrent Data Boost requests per project per region in other regions</td>
<td>400 <sup><a href="#note9">9</a></sup></td>
</tr>
<tr class="odd">
<td>Concurrent Data Boost requests in milli-operations per project per region</td>
<td>1000000 <sup><a href="#note9">9</a></sup></td>
</tr>
</tbody>
</table>

## Pre-splitting API limits

<table style="width:60%;">
<colgroup>
<col style="width: 20%" />
<col style="width: 40%" />
</colgroup>
<thead>
<tr class="header">
<th>Value</th>
<th>Limit</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Split points added per API request</td>
<td>100</td>
</tr>
<tr class="even">
<td>Split point API request size</td>
<td>1 MiB</td>
</tr>
<tr class="odd">
<td>Split points added per node for all the databases in the instance</td>
<td>50</td>
</tr>
<tr class="even">
<td>Split points added or updated per minute per node</td>
<td>10</td>
</tr>
<tr class="odd">
<td>Split points added or updated per day per node</td>
<td>200</td>
</tr>
</tbody>
</table>

## Notes

1\. To provide high availability and low latency for accessing a database, Spanner defines storage limits based on the compute capacity of the instance:

  - For instances smaller than 1 node (1000 processing units), Spanner allots 1024.0 GiB of data for every 100 processing units in the database.
  - For instances of 1 node and larger, Spanner allots 10 TiB of data for each node.

For example, to create an instance for a 1500 GiB database, you need to set its compute capacity to 200 processing units. This amount of compute capacity will keep the instance below the limit until the database grows to more than 2048.0 GiB. After the database reaches this size, you need to add another 100 processing units to allow the database to grow. Otherwise, writes to the database may be rejected. For more information, see [Recommendations for database storage utilization](/spanner/docs/storage-utilization#recommended-max) .

For a smooth growth experience, add compute capacity before the limit is reached for your database.

2\. The accounted schema objects include all the object types described in [DDL](/spanner/docs/data-definition-language) such as tables, columns, indexes, sequences, etc. The schema object limit is enforced at the instance level and is dependent on the processing units available to your instance.

  - For instances of one node or larger, the default limit is one million objects.
  - For instances smaller than one node (1000 processing units), the limit decreases proportionally to the size of the instance. For example, the limit is 100,000 schema objects for instances with 100 processing units.

To check the schema object count for your databases and the object limit for your instance, look for metrics `  spanner.googleapis.com/instance/schema_objects  ` and `  spanner.googleapis.com/instance/schema_object_count_limit  ` in **Metrics Explorer** . For more information about monitoring, see [Monitor instances with Cloud Monitoring](/spanner/docs/monitoring-cloud) .

If you reach the limit, Spanner prevents you from performing operations that put you over the limit, such as:

  - Modifying the database's schema (for example, adding an index).
  - Creating a new database in the instance.
  - Restoring a database from a backup into the same instance. In this case, you can [restore the backup](/spanner/docs/backup/restore-backups) in a different instance in the same configuration or [create a new instance](/spanner/docs/create-manage-instances) with the same configuration and restore the backup in the new instance.

3\. Insert and update operations count with the multiplicity of the number of columns they affect, and primary key columns are always affected. For example, inserting a new record may count as five mutations, if values are inserted into five columns. Updating three columns in a record may also count as five mutations if the record has two primary key columns. Delete and delete range operations count as one mutation regardless of the number of columns affected. Deleting a row from a parent table that has the [`  ON DELETE CASCADE  `](/spanner/docs/data-definition-language#create_table) annotation is also counted as one mutation regardless of the number of interleaved child rows present. The exception to this is if there are secondary indexes defined on rows being deleted, then the changes to the secondary indexes will be counted individually. For example, if a table has 2 secondary indexes, deleting a range of rows in the table will count as 1 mutation for the table, plus 2 mutations for each row that is deleted because the rows in the secondary index might be scattered over the key-space, making it impossible for Spanner to call a single delete range operation on the secondary indexes. Secondary indexes include the [foreign keys backing indexes](/spanner/docs/foreign-keys/overview#backing-indexes) .

To find the mutation count for a transaction, see [Retrieving commit statistics for a transaction](/spanner/docs/commit-statistics) .

Change streams don't add any mutations that count towards this limit.

4\. The limit for an administrative action request excludes commits, requests listed in [note 9](#note9) , and schema changes.

5\. This rate limit includes all calls to the admin API, which includes calls to poll long-running operations on an instance, database, or backup.

6\. This limit includes requests for creating a database, updating a database, reading, streaming reads, executing SQL queries, and executing streaming SQL queries.

7\. A change stream that watches an entire table or database implicitly watches every column in that table or database, and therefore counts towards this limit.

8\. This limit applies to concurrent readers of the same change streams partition, whether the readers are Dataflow pipelines or direct API queries.

9\. Default limits vary by project and regions. For more information, see [Monitor and manage Data Boost quota usage](/spanner/docs/databoost/databoost-quotas) .

10\. When you write to a table that has [generated columns](/spanner/docs/generated-column/how-to) or [columns with default values](/spanner/docs/primary-key-default-value#methods) , the number of mutations for the write is calculated as follows for each row:

  - Total mutations = (Number of columns explicitly written) + (Number of generated columns or columns with default values) + (Number of primary key columns)

For example, if you have a table with 2 primary key columns and 1 generated column, inserting a single row (by providing values for the 2 primary key columns) counts as 5 mutations:

  - 2 (for the explicitly written columns) + 1 (for the generated column) + 2 (for the primary keys).
