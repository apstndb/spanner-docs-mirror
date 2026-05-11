---
name: documents/docs.cloud.google.com/spanner-omni/download
uri: https://docs.cloud.google.com/spanner-omni/download
title: Download Spanner Omni
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document provides links and instructions to download Spanner Omni components, including container images, Helm charts, and standalone binaries.

The [Preview](https://cloud.google.com/products#product-launch-stages) version of Spanner Omni doesn't support TLS encryption. To get the features that let you create deployments with TLS encryption, [contact Google](https://cloud.google.com/consulting/spanner-omni) to request early access to the full version of Spanner Omni.

## Container images

Artifact Registry hosts Spanner Omni container images at `us-docker.pkg.dev/spanner-omni/images/` .

The following container images are available:

  - `spanner-omni` : includes Spanner Omni, the Spanner Omni CLI, and the Spanner Omni console.

  - `spanner-omni-server` : includes Spanner Omni and the Spanner Omni CLI.

  - `spanner-omni-ui` : includes only the Spanner Omni console.

Specify the exact version tag when pulling a container image.

### Container image versions

The following table lists the available container image versions.

| Version tag    | Release date   |
| -------------- | -------------- |
| `2026.r1-beta` | April 22, 2026 |

For example, to download the `spanner-omni` image for the 2026.r1-beta release, run the following command:

    docker pull us-docker.pkg.dev/spanner-omni/images/spanner-omni:2026.r1-beta

## Helm charts

Artifact Registry hosts Helm charts for Spanner Omni deployments at `us-docker.pkg.dev/spanner-omni/charts` . These charts support various deployment topologies, from single-server to multi-cluster. For more information, see [Create a Helm chart configuration for Spanner Omni](https://docs.cloud.google.com/spanner-omni/create-helm-configuration) .

Specify the exact version tag when you download a chart.

### Helm chart versions

The following table lists the available Helm chart versions.

| Version tag | Release date   |
| ----------- | -------------- |
| `0.1.0`     | April 22, 2026 |

For example, to download the Helm chart for version 0.1.0, run the following command:

    helm pull oci://us-docker.pkg.dev/spanner-omni/charts/spanner-omni --version 0.1.0

## Standalone binaries

The following Google Cloud bucket contains standalone binaries for the Spanner Omni and Spanner Omni CLI as TAR files: `https://storage.googleapis.com/spanner-omni/` .

Each release is in a folder named after the version tag.

### Spanner Omni CLI binaries

The following table lists the available Spanner Omni CLI binaries.

| Filename                                             | Description                           |
| ---------------------------------------------------- | ------------------------------------- |
| `spanner-omni-cli-2026.r1-beta-darwin-arm.tar.gz`    | Spanner Omni CLI, Mac (M1, M2 and M3) |
| `spanner-omni-cli-2026.r1-beta-darwin-x86_64.tar.gz` | Spanner Omni CLI for Mac (x86)        |
| `spanner-omni-cli-2026.r1-beta-linux-arm.tar.gz`     | Spanner Omni CLI for Linux (ARM)      |
| `spanner-omni-cli-2026.r1-beta-linux-x86_64.tar.gz`  | Spanner Omni CLI for Linux (x86)      |

For example, to download the current version of the Spanner Omni CLI for Linux (x86), run the following command:

    # Download CLI for Linux (x86)
    curl -O https://storage.googleapis.com/spanner-omni/2026.r1-beta/spanner-omni-cli-2026.r1-beta-linux-x86_64.tar.gz
