---
name: documents/docs.cloud.google.com/spanner-omni/faq
uri: https://docs.cloud.google.com/spanner-omni/faq
title: Spanner Omni FAQ
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

## About Spanner Omni

This section answers general questions about Spanner Omni.

### What is Spanner Omni?

Spanner Omni is a downloadable version of Spanner that lets you deploy Google's distributed database technology across on-premises data centers, public clouds, and on your laptop. For more information, see the [Spanner Omni overview](https://docs.cloud.google.com/spanner-omni/overview) .

## Billing and support

This section answers questions about Spanner Omni billing, pricing, and support options.

### How does pricing and purchasing work?

To purchase a subscription for the Spanner Omni Commercial edition, contact your [Google account team](https://cloud.google.com/consulting/spanner-omni) . For pricing details, see [Spanner Omni pricing](https://cloud.google.com/products/spanner/omni?e=48754805#pricing) . To compare edition features and options, see [Editions overview](https://docs.cloud.google.com/spanner-omni/editions-overview) .

### Do you offer paid support?

You can get [Premium Support](https://docs.cloud.google.com/support/docs/premium) for Spanner Omni. For demanding or complex workloads, Premium Support customers can sign up for [Mission Critical Services](https://cloud.google.com/blog/topics/inside-google-cloud/introducing-google-cloud-mission-critical-services) for guided consultative recommendations on assessment, remediation, and Spanner Omni onboarding. For more information, see [Google Cloud Customer Care](https://cloud.google.com/support) .

## License management

This section answers questions about managing and renewing Spanner Omni licenses.

### How do I renew or extend an expiring license?

To extend an expiring Developer license, submit the [request form](https://forms.gle/Ex9NcszwJFuHbtnB9) . You can also configure a single-server deployment with four vCPUs or fewer to use all advanced features without requiring a license.

To extend a Commercial proof-of-concept license, renew a Commercial license, or upgrade to a paid Commercial license, contact [Google sales](https://cloud.google.com/consulting/spanner-omni) .

For instructions on installing and updating keys, see [Install and manage a license](https://docs.cloud.google.com/spanner-omni/install-manage-license) .

## Managed Spanner and Spanner Omni

This section answers questions comparing Spanner Omni with managed Spanner on Google Cloud.

### How does Spanner Omni provide external consistency without GPS or atomic clocks available on managed Spanner on Google Cloud?

Spanner Omni implements a software-defined TrueTime API that provides bounded time uncertainty intervals without dedicated hardware. For more information, see [TrueTime and external consistency on Spanner Omni](https://docs.cloud.google.com/spanner-omni/true-time-external-consistency) .

### How does Spanner Omni provide distributed storage without Google Colossus available on Google Cloud with managed Spanner?

Spanner Omni replaces Google's internal Colossus distributed file system with a storage abstraction layer that mounts local block devices or storage area networks (SAN) formatted with standard files systems (ext4 recommended). For more information, read the *Tech behind Spanner Omni* section of the [Spanner Omni launch blog](https://cloud.google.com/blog/products/databases/introducing-spanner-omni) .

### What is the difference between Spanner Omni and managed Spanner on Google Cloud?

While both Spanner Omni and managed Spanner on Google Cloud share the same distributed database engine, they differ in how you deploy, scale, secure, and manage them. For more information, see [Differences between Spanner and Spanner Omni](https://docs.cloud.google.com/spanner-omni/differences) .
