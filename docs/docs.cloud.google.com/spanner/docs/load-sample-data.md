This document provides instructions on how to load a small amount of data in the CSV file format into Spanner. You can load sample data before performing a production data migration to test schemas, queries, and applications.

## Before you begin

1.  [Install the Google Cloud CLI](/sdk/docs/install) or use [Cloud Shell](/shell/docs/launching-cloud-shell) , which has all the necessary tools pre-installed.

2.  To get the permissions that you need to export BigQuery data to Spanner, ask your administrator to grant you the following IAM roles on your project:
    
      - Export data from a BigQuery table: BigQuery Data Viewer ( `  roles/bigquery.dataViewer  ` )
      - Run an export job: BigQuery User ( `  roles/bigquery.user  ` )
      - Write data to a Spanner table: Spanner Database User ( `  roles/spanner.databaseUser  ` )
    
    For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .
    
    You might also be able to get the required permissions through [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Load sample data to Spanner

The following instructions are performed using the [BigQuery reverse ETL](/bigquery/docs/export-to-spanner) workflow and the [Google Cloud CLI](/sdk) .

1.  Set a default project on the gcloud CLI using the following command:
    
    ``` text
     gcloud config set project PROJECT_ID
    ```

2.  Export the source data in the CSV file format. Consider using [`  pg_dump  `](https://www.postgresql.org/docs/current/app-pgdump.html) for PostgreSQL databases or [`  mysqldump  `](https://dev.mysql.com/doc/refman/8.4/en/mysqldump.html) for MySQL databases tools to convert your sample data into the CSV file format.
    
    **Note:** If you are working with sample data that's not available in the CSV file format, then consider [batch loading](/bigquery/docs/batch-loading-data) the sample data to BigQuery.

3.  Load the data into BigQuery by using the following `  bq  ` commands:
    
    1.  Create a BigQuery dataset.
        
        ``` text
        bq mk BQ_DATASET
        ```
    
    2.  Batch load the data into a new BigQuery table.
        
        ``` text
        bq load \
        --source_format=CSV \
        --autodetect \
        --allow_quoted_newlines \
        BQ_DATASET.BQ_TABLE /path/to/file
        ```
        
        Alternatively, you can batch load the data from a Cloud Storage file.
        
        ``` text
        bq load \
        --source_format=CSV \
        --autodetect \
        --allow_quoted_newlines \
        BQ_DATASET.BQ_TABLE gs://BUCKET/FILE
        ```

4.  Create a Spanner schema that matches the imported BQ\_TABLE by using the following command:
    
    ``` text
     gcloud spanner databases ddl update SPANNER_DATABASE \
     --instance=SPANNER_INSTANCE \
     --ddl="CREATE TABLE SPANNER_TABLE ..."
    ```
    
    For more information, see [Update Spanner schema](/spanner/docs/getting-started/gcloud#create-database) .

5.  Export data from BigQuery to Spanner by using the following command:
    
    ``` text
       bq --use_legacy_sql=false 'EXPORT DATA OPTIONS(
         uri="https://spanner.googleapis.com/projects/PROJECT_ID/instances/SPANNER_INSTANCE/databases/SPANNER_DATABASE"
         format='CLOUD_SPANNER'
         spanner_options="""{ "table": "SPANNER_TABLE" }"""
         ) AS
         SELECT *
         FROM BQ_DATASET.BQ_TABLE;'
     
    ```

## What's next

  - [Migration overview](/spanner/docs/migration-overview)
