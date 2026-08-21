---
name: documents/docs.cloud.google.com/spanner-omni/editions-overview
uri: https://docs.cloud.google.com/spanner-omni/editions-overview
title: Spanner Omni editions overview
description: Learn about the Developer and Commercial editions of Spanner Omni, including licensing, features, and support options.
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document compares the Developer and Commercial editions of Spanner Omni. These two Spanner Omni editions support your operational requirements, ranging from local prototyping to production-grade deployments. Compare features, terms, support, expiration, and costs to choose the best licensing strategy.

Select an edition based on your project phase and infrastructure needs:

  - **[Developer edition](https://docs.cloud.google.com/spanner-omni/editions-overview#developer-edition)** : A free option for non-production environments, such as local development, testing, and prototyping.
  - **[Commercial edition](https://docs.cloud.google.com/spanner-omni/editions-overview#commercial-edition)** : A paid subscription option for production and commercial workloads. The Commercial edition includes all enterprise features and qualifies for [premium support](https://cloud.google.com/support) .

## Developer edition

Use the Developer edition to evaluate, test, build prototypes, and run demonstrations of Spanner Omni. The Developer edition is free. Configure a single-server deployment with four vCPUs or fewer to use enterprise security features and backups and restores.

The Developer edition supports two license types:

  - **Default license** : Each new deployment installs this license. It expires 90 days after you create the deployment. After 90 days, the deployment disables writes. The default license can be extended by requesting a perpetual license key from Google.
    
    If you configure a single-server deployment with four vCPUs or fewer, the license doesn't expire. In this scenario, the Developer edition supports advanced security and backup features. If you scale the server beyond four vCPUs, then the deployment no longer supports the advanced features and returns to the default 90-day license.

  - **Developer license** : Request a perpetual license key if you want to use the Developer edition beyond the 90-day period. You can request and install a perpetual license key at any time, including after the default license expires. Spanner Omni shows a link in the Spanner Omni console or the Spanner Omni CLI to request the perpetual license key. Developer license keys aren't transferable or shareable. One license key should be used for one deployment only; using a single license key for multiple deployments is a violation of the license terms.

## Commercial edition

The Commercial edition supports commercial and production workloads and includes all features and enterprise security capabilities. Choose from two versions of the Commercial edition:

  - **Proof of concept** : This paid pre-production license is for testing and evaluation in commercial or production environments. Google charges a quarterly subscription fee per vCPU. It expires 90 days after issuance. It supports all enterprise security and data protection features so you can evaluate the full capabilities of Spanner Omni. Contact Google to request this license.

  - **Production** : This paid production license has an annual subscription. Google charges this fee per vCPU. The license terms align with your commercial contract. It supports all features and lets you purchase Google Cloud Customer Care for premium support.

## Compare editions and licenses

The following table compares the different Spanner Omni editions and license types:

Feature

Developer edition

Commercial edition

Proof of concept

Production

Primary purpose

Develop, test, prototype, and demonstrate for non-production and non-commercial workloads.

Test and evaluate pre-production commercial or production workloads.

Deploy production commercial workloads.

Expiration

Expires 90 days after deployment. You can extend the license by requesting a perpetual license from Google. Single-server deployments with up to four vCPUs don't expire.

Expires 90 days after issuance.

Subject to contract terms.

Enterprise features

Supports all features except advanced security features. If you run a single-server deployment with up to four vCPUs, then the advanced security features are supported.

Both the proof of concept and production options support all features.

Support

Community support.

Both the proof of concept and production options are eligible for premium support with [Google Cloud Customer Care](https://cloud.google.com/support) .
