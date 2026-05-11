---
name: documents/docs.cloud.google.com/spanner-omni/import-export-data
uri: https://docs.cloud.google.com/spanner-omni/import-export-data
title: Import and export data
description: Learn how to import and export data in Spanner Omni using Avro and CSV formats with Amazon S3, Google Cloud Storage, and local file systems.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to migrate, back up, and transfer data in Spanner Omni using Avro and CSV formats. Use the Spanner Omni CLI to move database content between Spanner Omni and storage solutions such as Cloud Storage, Amazon Simple Storage Service (Amazon S3), S3-compatible local storage, or local file systems (NFS). The import and export data flows in Spanner Omni don't support models, locality groups, or placements.

Import and export operations run on Spanner Omni servers and share available system resources. Importing is resource-intensive and can cause high RAM, CPU, and disk usage, which might affect active workloads. While these tasks usually run at a lower priority than regular traffic, you should monitor your deployment for potential performance impacts.

### Comparison of file formats

The following table compares the capabilities of Avro and CSV file formats for importing and exporting Spanner data.

| Capability                          | Avro | CSV |
| ----------------------------------- | ---- | --- |
| Import or export an entire database | Yes  | No  |
| Import previously exported tables   | Yes  | Yes |
| Export at a past timestamp          | Yes  | Yes |
| Import or export using Spanner      | Yes  | Yes |
| Import data from other databases    | No   | Yes |

Both Avro and CSV formats export all tables in the database. The Avro format also exports the schema so you can import it again. The CSV format doesn't export the schema.

## Before you begin

Before you start an import or export operation, verify your permissions and configure access to your data storage location.

### Permissions

Ensure you have the following permissions before you begin:

  - `spanner.databases.import`
  - `spanner.databases.export`

For more information about Identity and Access Management (IAM) in Spanner Omni, see [IAM overview](https://docs.cloud.google.com/spanner-omni/iam) . To learn how to update a user's roles, see [Update users](https://docs.cloud.google.com/spanner-omni/authentication#updating-users) .

### Data source and destination

You can store data in an Amazon Simple Storage Service (Amazon S3) bucket, Cloud Storage bucket, Amazon S3-compatible local storage (such as MinIO), or a local file system (NFS). If you use a local file system, ensure the data is available at the same path on all servers in the deployment.

You can provide access to the datastore in two ways:

  - Add external storage to the deployment: This is the preferred method if you plan to reuse a bucket.

  - Create one-time credentials: Ensure these credentials last longer than the duration of the import or export operation (for example, 48 hours).

The credentials must provide permissions to list and read objects in the bucket for imports. For exports to Amazon S3, you need the following additional Amazon S3 permissions:

  - `s3:PutObject`
  - `s3:AbortMultipartUpload`
  - `s3:ListBucketMultipartUploads`

For more information, see [IAM permissions](https://docs.cloud.google.com/spanner-omni/iam#permissions) .

## Import Spanner Avro files

To import data you've previously exported from another Spanner database (Spanner or Spanner Omni) in Avro format, follow these steps.

### Avro import prerequisites

Before you begin the Avro import, ensure your environment meets the following requirements:

  - You've created the destination database.

  - The schema objects you're importing don't already exist in the database. The Avro import process creates these tables before importing data.

### Avro import instructions

Identify the path to the folder containing the exported data. The folder contains the following:

  - A `spanner-export.json` file.

  - An `  ENTITY_NAME -manifest.json ` file for each exported entity (such as a table, sequence, or schema).

  - All Avro files listed in the manifest files.

If you've already added the datastore as external storage, you don't need to include credentials in the path. You can provide the path directly. If you're using one-time credentials, use the following URL formats:

  - Cloud Storage: `gs:// BUCKET_NAME / BASE_FOLDER [?accesskey= ACCESS_KEY &secret= SECRET_KEY ]` . Use HMAC credentials. For more information, see [HMAC keys](https://docs.cloud.google.com/storage/docs/authentication/hmackeys) .

  - Amazon S3: `s3:// S3_BUCKET / BASE_FOLDER [?accesskey= ACCESS_KEY &secret= SECRET_KEY [&sessiontoken= SESSION_TOKEN ]]`

  - Local file folde\*: ` file:/// PATH_TO_DIR  `

To start the import, run the following command:

    spanner databases import DATABASE_ID --url="URL" --format=avro [--avro-skip-wait-for-index-creation]

#### Additional notes

Consider the following information when you import Avro files:

  - [A note on importing generated columns and change streams](https://docs.cloud.google.com/spanner/docs/import#a_note_on_importing_generated_columns_and_change_streams) in Spanner documentation.

  - [A note on importing sequences](https://docs.cloud.google.com/spanner/docs/import#a_note_on_importing_sequences) in Spanner documentation.

  - [A note on importing interleaved tables and foreign keys](https://docs.cloud.google.com/spanner/docs/import#a_note_on_importing_interleaved_tables_and_foreign_keys) in Spanner documentation.

  - To skip importing specific entities, remove them from the `spanner-export.json` file.

  - Index creation can take a significant amount of time for large datasets. To skip waiting for index creation, use the optional `--avro-skip-wait-for-index-creation` flag.

When the import operation starts successfully, it returns a long-running operation ID. Use this ID to track the status of the operation.

## Import CSV files

To import text data you've exported from another database, follow these steps.

### CSV import prerequisites

Before you begin the CSV import, make sure you do the following:

  - Ensure your tables are in one of the following supported data types: `BOOL` , `INT64` , `FLOAT64` , `NUMERIC` , `STRING` , `DATE` , `TIMESTAMP` , `BYTES` , and `JSON` .

  - Create destination database.

  - Create all the tables into which you want to import data. The CSV import process doesn't create tables.

  - Make sure the CSV file doesn't contain a header row.

### CSV import instructions

To import CSV files, create a manifest file that describes the data to import. The manifest file uses the following structure, defined here in protobuf format:

    message ImportManifest {
      // The per-table import manifest.
      message TableManifest {
        // Required. The name of the destination table.
        string table_name = 1;
        // Required. The CSV files to import. This value can be either a path or a glob pattern.
        repeated string file_patterns = 2;
        // The schema for a table column.
        message Column {
          // Required for each column that you specify. The name of the column in the
          // destination table.
          string column_name = 1;
          // Required for each column that you specify. The type of the column.
          string type_name = 2;
        }
        // Optional. The schema for the table columns.
        repeated Column columns = 3;
      }
      // Required. The TableManifest of the tables to be imported.
      repeated TableManifest tables = 1;
    
      enum ProtoDialect {
        GOOGLE_STANDARD_SQL = 0;
        POSTGRESQL = 1;
      }
      // Optional. The dialect of the receiving database. Defaults to GOOGLE_STANDARD_SQL.
      ProtoDialect dialect = 2;
    }

The following is an example manifest:

    {
      "tables": [
        {
          "table_name": "Albums",
          "file_patterns": [
            "gs://bucket1/Albums_1.csv",
            "gs://bucket1/Albums_2.csv"
          ]
        },
        {
          "table_name": "Singers",
          "file_patterns": [
            "gs://bucket1/Singers*.csv"
          ],
          "columns": [
            {"column_name": "SingerId", "type_name": "INT64"},
            {"column_name": "FirstName", "type_name": "STRING"},
            {"column_name": "LastName", "type_name": "STRING"}
          ]
        }
      ]
    }

The URL in the following The CSV file does not contain a header rowimport command must point to the folder containing a manifest file in JSON format, as described in the [example manifest](https://docs.cloud.google.com/spanner-omni/import-export-data#example-manifest) . This file can be located in Cloud Storage, Amazon S3, or a local file folder, using the same URL format for credentials as described in [Avro import instructions](https://docs.cloud.google.com/spanner-omni/import-export-data#avro-instructions) . To start the import, run the following command:

    spanner databases import DATABASE_ID --url="URL" --format=csv

#### CSV import options

Use the following flags to customize how Spanner Omni handles text files:

  - `--csv-date-format` : Overrides the format for date columns. The default is `%Y-%m-%d` . Example: `%d/%m/%Y` .

  - `--csv-timestamp-format` : Overrides the format for timestamp columns. Use this only if Spanner Omni doesn't support the format in the CSV. Example: `%d/%m/%Y %H:%M:%S%Ez` .

  - `--csv-delimiter` : Overrides the delimiter character. The default is a comma.

  - `--csv-quote-char` : Overrides the quote character. The default is a double quote.

  - `--csv-escape-char` : Overrides the escape character. The default is a double quote.

  - `--csv-null-string` : Overrides the string that represents `NULL` values. The default is `\N` .

  - `--csv-has-trailing-delimiters` : Specifies whether the CSV files have trailing delimiters. The default is `false` .

## Export to Avro files

To export data to Avro files, follow the URL format instructions in the [Avro import instructions](https://docs.cloud.google.com/spanner-omni/import-export-data#avro-instructions) .

Any server in the deployment can write data to the provided datastore. If you use a local file folder as the destination, ensure all servers have access to the same path and can write to it in parallel.

The system exports all tables and entities in the database. Ensure you provide a new, empty folder path for the exported data.

To start the export, run the following command:

    spanner databases export DATABASE_ID --url="URL" --format=avro

CSV exports support tables only and don't export the database schema.

## Export to CSV files

CSV exports don't export the database schema and support tables only. To export data to CSV files, run the following command:

    spanner databases export DATABASE_ID --url="URL" --format=csv

## Troubleshooting

If an import fails, schema updates and imported data don't automatically revert. Manually clean up the database before you retry the operation.

The speed of an import operation depends on several factors, including the number of files in the folder, the available compute resources in the deployment, and disk speed. If sufficient resources are available, the system imports files in parallel.
