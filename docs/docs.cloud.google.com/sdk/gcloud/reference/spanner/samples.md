---
name: documents/docs.cloud.google.com/sdk/gcloud/reference/spanner/samples
uri: https://docs.cloud.google.com/sdk/gcloud/reference/spanner/samples
title: gcloud spanner samples
description: Offers tools and libraries that allow you to create and manage resources across Google Cloud.
data_source: docs.cloud.google.com
---

NAME

gcloud spanner samples - cloud Spanner sample apps

SYNOPSIS

`gcloud spanner samples` `  COMMAND  ` \[ `  GCLOUD_WIDE_FLAG …  ` \]

DESCRIPTION

Each Cloud Spanner sample application includes a backend gRPC service backed by a Cloud Spanner database and a workload script that generates service traffic.

These sample apps are open source and available at <https://github.com/GoogleCloudPlatform/cloud-spanner-samples> .

To see a list of available sample apps, run:

    gcloud spanner samples list

GCLOUD WIDE FLAGS

These flags are available to all commands: `  --help  ` .

Run ` $ gcloud help  ` for details.

COMMANDS

`  COMMAND  ` is one of the following:

  - `  backend  `  
    Run the backend gRPC service for the given Cloud Spanner sample app.
  - `  init  `  
    Initialize a Cloud Spanner sample app.
  - `  list  `  
    List available sample applications.
  - `  run  `  
    Run the given Cloud Spanner sample app.
  - `  workload  `  
    Generate gRPC traffic for a given sample app's backend service.

NOTES

These variants are also available:

    gcloud alpha spanner samples

    gcloud beta spanner samples
