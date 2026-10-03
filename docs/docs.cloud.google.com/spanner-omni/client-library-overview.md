---
name: documents/docs.cloud.google.com/spanner-omni/client-library-overview
uri: https://docs.cloud.google.com/spanner-omni/client-library-overview
title: Client library support for Spanner Omni
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

You can use existing [Spanner client libraries](https://docs.cloud.google.com/spanner/docs/reference/libraries) to connect to Spanner Omni. However, there are some differences in how to connect, specifically in connection strings and authentication methods. This document describes these differences and provides guidance for accessing Spanner Omni with existing Spanner client libraries and drivers.

Although the Spanner Omni resource model references entities such as projects, outside of Google Cloud, these entities have no significance. Additionally, other common entities, such as instances, might differ based on the deployment method.

When using Spanner client libraries with Spanner Omni, note the following differences:

- The `project` and `instance` must be set to `default` when required by client libraries or tools.

- Multi-tenancy isn't supported within a Spanner Omni deployment. Each deployment provides a single, precreated default instance (named `instances/default` ), which you can use to create databases and other resources.

## What's next

- [Use the Go client library to connect to Spanner Omni](https://docs.cloud.google.com/spanner-omni/go) .

- [Use the Java client library to connect to Spanner Omni](https://docs.cloud.google.com/spanner-omni/java) .

- [Use the Python client library to connect to Spanner Omni](https://docs.cloud.google.com/spanner-omni/python) .
