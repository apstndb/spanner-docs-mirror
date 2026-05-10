---
name: documents/docs.cloud.google.com/spanner-omni/beam
uri: https://docs.cloud.google.com/spanner-omni/beam
title: Use Apache Beam and SpannerIO to connect to Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
update_time: "2026-05-08T21:32:37Z"
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

[Apache Beam](https://beam.apache.org/) is an open source, unified model for defining both batch and streaming data-parallel processing pipelines. This document describes how to use the SpannerIO connector within an Apache Beam pipeline to read from or write to Spanner Omni databases.

## Before you begin

To connect SpannerIO to Spanner Omni, ensure that you meet the following requirements:

  - Initialize a database within your Spanner Omni environment.

  - Use Apache Beam release 2.69.0 or later.

  - Set up authentication credentials for your environment.

## Configure SpannerIO to connect to Spanner Omni

To connect SpannerIO to Spanner Omni, you must configure the `SpannerConfig` with your database details and connection parameters. Note that SpannerIO supports only plain-text communications when connecting to Spanner Omni.

To configure the connection, do the following:

1.  Specify the Spanner Omni database instance and endpoint.

2.  Enable experimental host support using the `withExperimentalHost` method.

3.  Configure the pipeline to use a plain-text channel.

The following example shows how to create a `SpannerConfig` object for Spanner Omni:

    SpannerConfig spannerConfig =
        SpannerConfig.create()
            .withDatabaseId("DATABASE_ID")
            // Define the Spanner Omni endpoint
            .withExperimentalHost("http://ENDPOINT")
            // SpannerIO supports only plain-text connections
            .withUsingPlainTextChannel(true);
