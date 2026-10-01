---
name: documents/docs.cloud.google.com/spanner-omni/major-compaction
uri: https://docs.cloud.google.com/spanner-omni/major-compaction
title: Manually trigger major compaction in a Spanner Omni database
description: A downloadable, self-managed version of Spanner.
data_source: docs.cloud.google.com
---

This document explains how to manually trigger a major compaction in your Spanner Omni database. This process is mostly identical to [triggering a major compaction in a Spanner database](https://docs.cloud.google.com/spanner/docs/manual-data-compaction) , with the following distinction:

  - To trigger a major compaction for a Spanner Omni database, use the following command in the Spanner Omni CLI:
    
        spanner admin alpha compact database DATABASE_ID
    
    Replace DATABASE\_ID with the database identifier—for example, `MY_DATABASE` .
