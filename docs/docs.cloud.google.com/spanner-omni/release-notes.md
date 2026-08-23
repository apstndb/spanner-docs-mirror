---
name: documents/docs.cloud.google.com/spanner-omni/release-notes
uri: https://docs.cloud.google.com/spanner-omni/release-notes
title: Spanner Omni release notes
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This page documents production updates to Spanner Omni. Check this page for announcements about new or updated features, bug fixes, known issues, and deprecated functionality.

You can see the latest product updates for all of Google Cloud on the [Google Cloud](https://docs.cloud.google.com/release-notes) page, browse and filter all release notes in the [Google Cloud console](https://console.cloud.google.com/release-notes) , or programmatically access release notes in [BigQuery](https://console.cloud.google.com/bigquery?p=bigquery-public-data&d=google_cloud_release_notes&t=release_notes&page=table) .

## August 20, 2026

Fixed

Spanner Omni patch release `2026.r2.1-beta` is available.

In-place upgrades from earlier versions to `2026.r2.1-beta` aren't supported. If you upgrade from an earlier release, you must migrate data from earlier deployments to a new deployment created with release `2026.r2.1-beta` . For more information, see [Restore a Spanner Omni backup](https://docs.cloud.google.com/spanner-omni/restores) and [Import and export data](https://docs.cloud.google.com/spanner-omni/import-export-data) .

Patch release `2026.r2.1-beta` includes the following updates:

  - Two editions are available: [developer](https://docs.cloud.google.com/spanner-omni/editions-overview#developer-edition) and [commercial](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) . For more information, see [Spanner Omni editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) .

  - A standalone server package is available to run on virtual machines. This package doesn't require Kubernetes. For more information, see [Download Spanner Omni](https://docs.cloud.google.com/spanner-omni/download#server_binaries) .

  - Support for the `--insecure-mode` and `--enable-client-certificate-authentication` flags for the `spanner start` command has been deprecated. Specify this information when you configure your deployment. For more information, see [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms) and [Create a Helm chart configuration](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) .

## June 18, 2026

Fixed

Spanner Omni patch release `2026.r1-beta.2` is available. This patch includes the following update:

  - Fixed an issue related to importing a backup in regional and multi-regional deployments. For more information, see [Restore a Spanner Omni backup](https://docs.cloud.google.com/spanner-omni/restores) .

## June 04, 2026

Fixed

Spanner Omni patch release `2026.r1-beta.1` is available. This patch includes the following updates:

  - Fixed a [TrueTime](https://docs.cloud.google.com/spanner-omni/true-time-external-consistency) issue in which time servers entered a repeated failover mode, causing the uncertainty window to spike.

  - Enables any client with a network path to the servers to reach health check and metrics endpoints. Earlier releases restricted access to internal IP addresses.

  - The following [Helm chart](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) updates:
    
      - Added support for `nodeSelector` , `tolerations` , and job resource configurations.
    
      - Added an `init` container to provide permissions to `/dev/vmclock` in Amazon Web Services (AWS) deployments.

## April 22, 2026

Feature

Spanner Omni is available in [Preview](https://cloud.google.com/products?e=48754805#product-launch-stages) . Spanner Omni a self-managed version of [Spanner](https://docs.cloud.google.com/spanner/docs/overview) that you run in your own environment, such as on-premises data centers, public clouds or on your laptop. For more information, see the following:

  - [Spanner Omni overview](https://docs.cloud.google.com/spanner-omni/overview) .

  - [Create a deployment on VMs](https://docs.cloud.google.com/spanner-omni/deploy-on-vms) .

  - [Create a deployment on Kubernetes](https://docs.cloud.google.com/spanner-omni/deploy-on-kubernetes) .
