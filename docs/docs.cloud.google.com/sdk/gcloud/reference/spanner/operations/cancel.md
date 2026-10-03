---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel
title: gcloud spanner operations cancel
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner operations cancel - cancel a Cloud Spanner operation

SYNOPSIS

`gcloud spanner operations cancel` `OPERATION-ID` ( [`--instance`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#--instance) = `INSTANCE` \| [`--instance-config`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#--instance-config) = `INSTANCE_CONFIG` ) \[ [`--backup`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#--backup) = `BACKUP` \] \[ [`--database`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#--database) = `DATABASE` \] \[ [`--instance-partition`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#--instance-partition) = `INSTANCE_PARTITION` \] \[ [`GCLOUD_WIDE_FLAG`](https://docs.cloud.google.com/sdk/gcloud/reference/spanner/operations/cancel#GCLOUD-WIDE-FLAGS)` …` \]

DESCRIPTION

Cancel a Cloud Spanner operation.

EXAMPLES

To cancel an instance operation with ID `auto_12345, run: `

```
gcloud spanner operations cancel _auto_12345 --instance=my-instance-id
```

`To cancel a database operation with ID` auto_12345, run:

```
gcloud spanner operations cancel _auto_12345 --instance=my-instance-id --database=my-database-id
```

To cancel a backup operation with ID `auto_12345, run:`

```
gcloud spanner operations cancel _auto_12345 --instance=my-instance-id --backup=my-backup-id
```

To cancel an instance partition operation with ID auto_12345, run:

```
gcloud spanner operations cancel auto_12345 --instance=my-instance-id --instance-partition=my-partition-id
```

POSITIONAL ARGUMENTS

`OPERATION-ID`  
ID of the operation

REQUIRED FLAGS

Exactly one of these must be specified:

`--instance` = `INSTANCE`  
The ID of the instance the operation is executing on.

`--instance-config` = `INSTANCE_CONFIG`  
The ID of the instance configuration the operation is executing on.

OPTIONAL FLAGS

`--backup` = `BACKUP`  
For a backup operation, the name of the backup the operation is executing on.

`--database` = `DATABASE`  
For a database operation, the name of the database the operation is executing on.

`--instance-partition` = `INSTANCE_PARTITION`  
For an instance partition operation, the name of the instance partition the operation is executing on.

GCLOUD WIDE FLAGS

These flags are available to all commands: [`--access-token-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--access-token-file) , [`--account`](https://docs.cloud.google.com/sdk/gcloud/reference#--account) , [`--billing-project`](https://docs.cloud.google.com/sdk/gcloud/reference#--billing-project) , [`--configuration`](https://docs.cloud.google.com/sdk/gcloud/reference#--configuration) , [`--flags-file`](https://docs.cloud.google.com/sdk/gcloud/reference#--flags-file) , [`--flatten`](https://docs.cloud.google.com/sdk/gcloud/reference#--flatten) , [`--format`](https://docs.cloud.google.com/sdk/gcloud/reference#--format) , [`--help`](https://docs.cloud.google.com/sdk/gcloud/reference#--help) , [`--impersonate-service-account`](https://docs.cloud.google.com/sdk/gcloud/reference#--impersonate-service-account) , [`--log-http`](https://docs.cloud.google.com/sdk/gcloud/reference#--log-http) , [`--project`](https://docs.cloud.google.com/sdk/gcloud/reference#--project) , [`--quiet`](https://docs.cloud.google.com/sdk/gcloud/reference#--quiet) , [`--trace-token`](https://docs.cloud.google.com/sdk/gcloud/reference#--trace-token) , [`--user-output-enabled`](https://docs.cloud.google.com/sdk/gcloud/reference#--user-output-enabled) , [`--verbosity`](https://docs.cloud.google.com/sdk/gcloud/reference#--verbosity) .

Run `$ `[`gcloud help`](https://docs.cloud.google.com/sdk/gcloud/reference) for details.

NOTES

These variants are also available:

```
gcloud alpha spanner operations cancel
```

```
gcloud beta spanner operations cancel
```
