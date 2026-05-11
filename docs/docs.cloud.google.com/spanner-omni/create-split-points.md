---
name: documents/docs.cloud.google.com/spanner-omni/create-split-points
uri: https://docs.cloud.google.com/spanner-omni/create-split-points
title: Create and manage Spanner Omni database split points
description: A downloadable, self-managed version of Spanner. {% setvar launch_stage %}preview{% endsetvar %} {% include "cloud/_shared/_info_launch_stage_disclaimer.html" %}
data_source: docs.cloud.google.com
---

> **Preview**
> 
> This product or feature is a preview offering subject to the "Pre-GA Offerings Terms" in the [General Service Terms](https://cloud.google.com/terms/service-terms) section of the Service Specific Terms, and can only be used for the purposes of developing, testing, prototyping, and demonstrating software programs. It cannot be used for any data processing or commercial purposes. Pre-GA products and features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products#product-launch-stages) .

This document describes how to create and manage splits in your Spanner Omni database to prepare for increased traffic. This process is mostly identical to [creating and managing Spanner split points](https://docs.cloud.google.com/spanner/docs/create-manage-split-points) , with the following distinctions:

  - To *add* manual splits to your Spanner Omni database, use the following command:
    
        spanner databases splits add DATABASE_ID \
        --splits-file=FILE_PATH \
        [--split-expiration-time=DATE]
    
    Replace the following:
    
      - DATABASE\_ID : the database identifier—for example, `MY_DATABASE` .
    
      - FILE\_PATH : the path to the file containing the [split point definitions](https://docs.cloud.google.com/spanner/docs/create-manage-split-points#gcloud) —for example, `/home/user/project/spanner/manual_splits.txt` .
    
      - DATE : optional, the date after which the splits expire. Accepts [timestamp values](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#timestamp_type) and relative time values—for example, `2072-09-27T12:30:00.45Z` or `now+7d` .

  - To *view* the manual split points in your Spanner Omni database, use the following command:
    
        spanner sql \
        --database DATABASE_ID --execute \
        "SELECT * \
        FROM SPANNER_SYS.USER_SPLIT_POINTS"
