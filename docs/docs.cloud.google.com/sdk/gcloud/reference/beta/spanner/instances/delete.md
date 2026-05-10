---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/beta/spanner/instances/delete
uri: https://docs.cloud.google.com/sdk/gcloud/reference/beta/spanner/instances/delete
title: gcloud beta spanner instances delete
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
update_time: "2026-02-18T19:35:28Z"
---

NAME

gcloud beta spanner instances delete - delete a Cloud Spanner instance

SYNOPSIS

`gcloud beta spanner instances delete` `  INSTANCE  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

`(BETA)` Delete a Cloud Spanner instance.

EXAMPLES

To delete a Cloud Spanner instance, run:

    gcloud beta spanner instances delete my-instance-id

POSITIONAL ARGUMENTS

  - `  INSTANCE  `  
    Cloud Spanner instance ID.

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --access-token-file  ` , `  --account  ` , `  --billing-project  ` , `  --configuration  ` , `  --flags-file  ` , `  --flatten  ` , `  --format  ` , `  --help  ` , `  --impersonate-service-account  ` , `  --log-http  ` , `  --project  ` , `  --quiet  ` , `  --trace-token  ` , `  --user-output-enabled  ` , `  --verbosity  ` .

Run ` $ gcloud help  ` for details.

NOTES

This command is currently in beta and might change without notice. These variants are also available:

    gcloud spanner instances delete

    gcloud alpha spanner instances delete
