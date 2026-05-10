---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/instance-configs/delete
title: gcloud spanner instance-configs delete
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
update_time: "2026-02-18T19:35:46Z"
---

NAME

gcloud spanner instance-configs delete - delete a Cloud Spanner instance configuration

SYNOPSIS

`gcloud spanner instance-configs delete` `  INSTANCE_CONFIG  ` \[ `  --etag  ` = `  ETAG  ` \] \[ `  --validate-only  ` \] \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

Delete a Cloud Spanner instance configuration.

EXAMPLES

To delete a custom Cloud Spanner instance configuration, run:

    gcloud spanner instance-configs delete custom-instance-config

POSITIONAL ARGUMENTS

  - `  INSTANCE_CONFIG  `  
    Cloud Spanner instance config.

FLAGS

  - `--etag` = `  ETAG  `  
    Used for optimistic concurrency control as a way to help prevent simultaneous deletes of an instance config from overwriting each other.
  - `--validate-only`  
    If specified, validate that the deletion will succeed without deleting the instance config.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --access-token-file  ` , `  --account  ` , `  --billing-project  ` , `  --configuration  ` , `  --flags-file  ` , `  --flatten  ` , `  --format  ` , `  --help  ` , `  --impersonate-service-account  ` , `  --log-http  ` , `  --project  ` , `  --quiet  ` , `  --trace-token  ` , `  --user-output-enabled  ` , `  --verbosity  ` .

Run ` $ gcloud help  ` for details.

NOTES

These variants are also available:

    gcloud alpha spanner instance-configs delete

    gcloud beta spanner instance-configs delete
