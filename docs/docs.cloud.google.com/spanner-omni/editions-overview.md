---
name: documents/docs.cloud.google.com/spanner-omni/editions-overview
uri: https://docs.cloud.google.com/spanner-omni/editions-overview
title: Spanner Omni editions overview
description: Learn about the Developer and Commercial editions of Spanner Omni, including licensing, features, and support options.
data_source: docs.cloud.google.com
---

> [Download Spanner Omni](https://docs.cloud.google.com/spanner-omni/download) to try it out at no charge. If you decide you want to use Spanner Omni for production use, [contact Google](https://cloud.google.com/consulting/spanner-omni) to learn about acquiring a license for the [commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition) .

This document compares the Developer and Commercial editions of Spanner Omni across features, terms, support, expiration, and cost so that you can choose the right licensing strategy for your workload:

  - **[Developer edition](https://docs.cloud.google.com/spanner-omni/editions-overview#developer-edition)** : An edition available at no charge for non-production environments, such as local development, testing, prototyping, and demonstrations. It doesn't include premium support or the workers feature.
  - **[Commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition)** : A paid subscription option for production and commercial workloads that includes all enterprise features and qualifies for [premium support](https://docs.cloud.google.com/support) .

## Developer edition

The Developer edition is activated by default at no charge when you deploy Spanner Omni, without requiring acquisition of a license key. How the deployment operates depends on its size and whether you install a perpetual license key:

  - **Single server with four vCPUs or fewer** : The deployment doesn't expire and supports all single-server features, including [backup and restore](https://docs.cloud.google.com/spanner-omni/backup-restore) , without a license key.
  - **More than one server or more than four vCPUs** : The deployment expires 90 days after creation, including if you later scale up from a smaller deployment. After 90 days, Spanner Omni disables writes while data remains queryable.
  - **Perpetual license key** : To run a Developer edition deployment beyond 90 days or scale beyond a single server with four vCPUs, you can [request a perpetual license key](https://forms.gle/Ex9NcszwJFuHbtnB9) at no charge and install it at any time. Spanner Omni also provides a link in the Spanner Omni console and the Spanner Omni CLI to request a key. Each key is non-transferable and valid for a single deployment only. Backups and restore aren't supported when a perpetual license key is installed.

## Commercial edition

The Commercial edition supports commercial and production workloads and includes all features. The Spanner Omni Commercial edition supports two license types:

  - <span id="commercial-license">**Proof of concept** : This paid pre-production license is for testing and evaluation in commercial or production environments. Google charges a quarterly subscription fee per vCPU. It expires 90 days after issuance. To let you evaluate the full capabilities of Spanner Omni at production scale, this license supports all features, including data protection features, for deployments of any size. To purchase or extend a proof-of-concept license, contact [Google](https://cloud.google.com/consulting/spanner-omni) .</span>
  - **Commercial license** : This paid production license has an annual subscription. Google charges this fee per vCPU. The license terms align with your commercial contract. To purchase a non-expiring production license, contact [Google](https://cloud.google.com/consulting/spanner-omni) . It supports all features and lets you purchase [Cloud Customer Care for premium support](https://docs.cloud.google.com/support/docs/purchasing-setting-up-premium) . You must [maintain compliance](https://docs.cloud.google.com/spanner-omni/install-manage-license#maintain-license-compliance) with your license agreement.

## Compare editions and licenses

The following table compares the different Spanner Omni editions and license types:

Feature

Developer edition

Commercial edition

Proof of concept

Commercial

Primary purpose

Develop, test, prototype, and demonstrate for non-production and non-commercial workloads for personal use to prepare for commercial deployment.

Test and evaluate pre-production commercial or production workloads.

Deploy production commercial workloads.

Expiration

90-day expiration by default. You can [request a non-expiring perpetual Developer license from Google](https://forms.gle/Ex9NcszwJFuHbtnB9) if you want to extend the license beyond 90 days.

Expires 90 days after issuance. You can extend the license by [contacting Google](https://cloud.google.com/consulting/spanner-omni) .

Subject to contract terms.

Enterprise features

All core features are supported under the Developer license. If deployed on a single server with up to four vCPUs, all single-server enterprise features, including backup and restore, are supported. Backups and restore aren't supported when a perpetual license key is installed. The workers feature isn't supported regardless of how many servers or vCPUs are used.

Both the proof of concept and production options support all features, including workers and backup and restore.

Support

Community support through the Google Cloud Community [forums](https://discuss.google.dev/tag/cloud-spanner/264) or [Slack channel](https://googlecloud-community.slack.com/archives/C49R7DSTH) . You can also file [issues](https://issuetracker.google.com/issues/new?component=190851&template=844402) or [enhancement requests](https://issuetracker.google.com/issues/new?component=190851&template=1163127) (include "\[Spanner Omni\]" in the issue title).

Both the proof of concept and commercial licenses are eligible for premium support with [Customer Care](https://docs.cloud.google.com/support) .

Cost

No charge.

90-day subscription fee per vCPU.

Annual subscription fee per vCPU.

## What's next

  - [Install and manage a Spanner Omni license](https://docs.cloud.google.com/spanner-omni/install-manage-license) .

  - [Download Spanner Omni](https://docs.cloud.google.com/spanner-omni/download) .

  - [Maintain Commercial license compliance](https://docs.cloud.google.com/spanner-omni/install-manage-license#maintain-license-compliance) .
