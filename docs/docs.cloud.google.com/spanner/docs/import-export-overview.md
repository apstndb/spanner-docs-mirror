You can import and export a large amount of data into or out of Spanner using any of the following methods:

  - Import or export any Spanner database using [Dataflow](/dataflow) .
  - Export any Spanner database into a Cloud Storage bucket using either Avro or CSV file formats.
  - Import data from Avro or CSV files into a new Spanner database.

**Note:** If you export a Spanner database to Cloud Storage, then import it back to Spanner, make sure you import the database into a Spanner instance with the same (or higher-tier) Spanner edition as the source instance. Otherwise, the import might fail. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

## Use cases

You can use Spanner import and export for the following use cases:

  - **Bulk loading** : You can import data in bulk into Spanner.

  - **Long-term backup and archiving** : You can export your database at any time and store it in a Cloud Storage bucket location of your choice for long-term backup or archiving. In addition, you can use [point-in-time recovery](/spanner/docs/pitr) to export a database from a specific past timestamp. If you are looking for disaster recovery techniques that offer quicker restoration but have a shorter retention periods, consider using [backups](/spanner/docs/backup) or [point-in-time recovery (PITR)](/spanner/docs/pitr) .

  - **Copying databases to development or test projects** : You can export a database from a production project and then import it into your development or test project to use for integration tests or other experiments.

  - **Ingesting for analytics** : You can export a database to ingest your operational data in bulk to analytics services such as BigQuery. BigQuery can automatically ingest data in Avro format from a Cloud Storage bucket, making it easier for you to run analytics on your operational data. If you want to use BigQuery for real-time analysis of Spanner data without copying or moving the data, you can use Spanner federated queries instead.

## Compare import and export to back up and restore

Spanner import and export is similar to back up and restore in many ways. The following table describes similarities and differences between them to help you decide which one to use.

Back up and restore

Import and export

Data consistency

Both backups and exported databases are transactionally and externally consistent.

Performance impact

Backups have no impact on an instance's performance. Spanner performs backups using dedicated jobs that don't draw upon an instance's server resources.

Export runs as a medium-priority task to minimize impact on database performance. For more information, see [task priority](/spanner/docs/cpu-utilization#task-priority) .

Storage format

Uses a proprietary, encrypted format designed for fast restore.

Supports both CSV and [Avro](https://en.wikipedia.org/wiki/Apache_Avro) file formats.

Portability

You [create](/spanner/docs/backup/create-backup) backups in the same instance as their source database.  
  
After a backup is created, you can [copy](/spanner/docs/backup/copy-backup) the backup to an instance in a different region or project if you need a cross-region or cross-project backup. You can then [restore](/spanner/docs/backup/restore-backup-overview) from a backup as a new database to any instance in the same project. The instance that you are restoring to should have the same instance configuration as the instance where the backup is stored.

Exported databases reside in [Cloud Storage](https://cloud.google.com/storage) and the data can be migrated to any system that supports CSV or Avro.

Retention

Backups can be retained for up to one year.

Exported databases are stored in Cloud Storage where, by default, they are retained until they are deleted. You can customize [lifecycle](/storage/docs/lifecycle) and [retention](/storage/docs/bucket-lock) policies.

Pricing

Backups are billed to your Spanner project based on the storage used per unit time. For more details, see the [Pricing](/spanner/docs/backup#pricing) section.

Billing for import and export is more complicated due to its use of [Cloud Storage](https://cloud.google.com/storage) and [Dataflow](https://cloud.google.com/dataflow) . For more information, see [Database export and import pricing](https://cloud.google.com/spanner/pricing#export-import-pricing) .

Restore time

Restore happens in two operations: restore and optimize. The restore operation offers fast time-to-first-byte because the database directly mounts the backup without copying the data. After the restore operation completes, the database is ready for use, though read latency might be slightly higher while it is optimizing. For more information, see [How restore works](/spanner/docs/backup/restore-backup-overview#how-restore-works) .

Import is slower. You need to wait for all the data to be written into the database.

## Compare file formats

The following table compares the capability differences between Avro and CSV file formats when importing and exporting Spanner data.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Capability</th>
<th style="text-align: center;">Avro format</th>
<th style="text-align: center;">CSV format</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Import or export an entire database</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ability to export only selected tables in a database</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Ability to import previously exported tables</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="even">
<td style="text-align: left;">Export at a past timestamp</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Import or export using Google Cloud CLI</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="even">
<td style="text-align: left;">Import or export using Dataflow</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">Yes</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Import or Export using Spanner</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">No</td>
</tr>
</tbody>
</table>

### Avro files

When exporting to Avro format, you can specify a [list of tables to export](/spanner/docs/export#export-subset-tables) . Any [child tables](/spanner/docs/schema-and-data-model#parent-child_table_relationships) exported this way need to be accompanied by their parent tables. Spanner maintains the entire database schema in the exported files.

When importing from Avro format, Spanner recreates the exported database's whole schema, including all tables. Tables included in the original export receive all their exported data; all other tables remain empty.

The Spanner page of the Google Cloud console offers limited Avro-format import and export options. For example, you can't set network and subnetwork options. For a wider set of options, use [Dataflow](/dataflow/docs/overview) instead.

#### Limitations

You can't export and import [locality groups](/spanner/docs/create-manage-locality-groups) to the Avro format.

### CSV files

You can export only a single Spanner table in the CSV format at a time. When you export, the schema is not exported, only the data is exported.

Before importing from CSV files, you need to [create a JSON manifest](/spanner/docs/import-export-csv#create-json-manifest) file.

## Pricing

There are no additional charges from Spanner for using the export or import tools; you pay the standard rates for data storage when you import a database to Spanner. However, there are other potential charges associated with importing and exporting databases. For more information, see [Database export and import pricing](https://cloud.google.com/spanner/pricing#database-export-and-import-pricing) .

## What's next

  - [Export databases from Spanner to Avro](/spanner/docs/export)
  - [Import Spanner Avro files](/spanner/docs/import)
  - [Import and export data in CSV format](/spanner/docs/import-export-csv)
  - [Import data from non-Spanner databases](/spanner/docs/import-non-spanner)
