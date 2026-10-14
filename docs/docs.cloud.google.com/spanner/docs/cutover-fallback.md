---
name: documents/docs.cloud.google.com/spanner/docs/cutover-fallback
uri: https://docs.cloud.google.com/spanner/docs/cutover-fallback
title: Configure fallback and cutover
description: Use fallback and cutover when migrating your database to Spanner to minimize downtime and risk.
data_source: docs.cloud.google.com
---

Migrations are often time-consuming and complex. It is possible that after data migration and cutover, you might encounter inconsistent performance or issues with the Spanner database.

We recommend configuring fallback from your Spanner database before performing cutover to avoid significant impact if an error occurs during migration. Fallback lets you revert to the source database with minimal downtime.

Reverse replication supports fallback by replicating data written to Spanner back to your source database. If you need to fall back, you can then point your application to the source database and continue serving requests with minimal downtime.

Spanner provides prebuilt reverse replication pipelines that handle this process out of the box, so you don't need to build custom reverse replication tooling. The pipeline automatically does the following:

- Handles changes in data types and schema mappings between Spanner and your source database.
- Reverses any transformations performed during the forward migration.
- Routes data to the appropriate destination, taking into account sharding schemes on the source database.
- Captures changes from Spanner change streams before writing to your source database, ensuring consistency eventually.
- Manages error handling and retries through dead-letter queues.

## Source-specific guides

Use the following guides for each phase of fallback and cutover:

- MySQL:
  - [Configure fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-configure-fallback) .
  - [Perform cutover from MySQL](https://docs.cloud.google.com/spanner/docs/mysql-perform-cutover) .
  - [Perform fallback to MySQL](https://docs.cloud.google.com/spanner/docs/mysql-fallback) .
