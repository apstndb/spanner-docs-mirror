---
name: documents/docs.cloud.google.com/spanner/docs/labels
uri: https://docs.cloud.google.com/spanner/docs/labels
title: Organize instances and view costs using labels
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

This page describes how to use Google Cloud labels to organize your Spanner instances and view a granular cost breakdown.

A [Google Cloud label](https://docs.cloud.google.com/resource-manager/docs/creating-managing-labels#what-are-labels) is a key-value pair that you can assign to individual Google Cloud resources, such as Spanner instances.

Labels help you organize these resources and view your costs at the granularity you need. Information about labels is forwarded to the billing system that lets you break down your billed charges by label. To learn more about labels, see [Labels overview](https://docs.cloud.google.com/resource-manager/docs/labels-overview) .

## Common use cases for labels

Some common use cases for labels include:

  - **Team or cost center labels:** Distinguish resources owned by different teams (for example, `team:research` and `team:analytics` ) for cost accounting or budgeting.
  - **Environment labels:** Specify development, testing, or production environments (for example, `env:dev` , `env:test` , and `env:prod` ).
  - **Component labels:** Categorize resources by application component or workload type (for example, `component:frontend` and `component:backend` ).
  - **Granular instance-level tracking:** Label Spanner instances with their own ID or a specific identifier (for example, `instance_id:my-instance-1` ) to directly break down costs per instance in billing reports or BigQuery queries.

## Requirements and constraints

Labels applied to Spanner instances must meet the following requirements:

  - Each instance can have up to 64 labels.
  - Keys and values must be 63 characters or less.
  - Keys and values can contain only lowercase letters, numeric characters, underscores ( `_` ), and dashes ( `-` ).
  - Keys must start with a lowercase letter or international character. Keys cannot be empty.
  - The key portion of a label must be unique within a single instance.

## Add and manage labels on Spanner instances

You can add labels when creating a Spanner instance, or update the labels on an existing instance.

### Required permissions

To view and manage labels on Spanner instances, you need the following IAM permissions:

  - `spanner.instances.get`
  - `spanner.instances.update`

These permissions are included in the `roles/spanner.admin` role.

### Add, update, or remove labels

### Console

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  Select the checkbox next to the instance you want to label. The **Info panel** appears.

3.  Select **Labels** in the **Info panel** .

4.  Add, update, or delete labels for the instance, then click **Save** .

### gcloud

To create a Spanner instance with labels, run the `gcloud spanner instances create` command with the `--labels` flag:

    gcloud spanner instances create INSTANCE_ID \
        --config=INSTANCE_CONFIG \
        --description=INSTANCE_NAME \
        --nodes=NODE_COUNT \
        --labels=KEY=VALUE,KEY=VALUE

To add or update labels on an existing instance, run the `gcloud spanner instances update` command with the `--update-labels` flag:

    gcloud spanner instances update INSTANCE_ID \
        --update-labels=KEY=VALUE,KEY=VALUE

To remove specific labels from an existing instance, use the `--remove-labels` flag:

    gcloud spanner instances update INSTANCE_ID \
        --remove-labels=KEY1,KEY2

To clear all labels from an existing instance, use the `--clear-labels` flag:

    gcloud spanner instances update INSTANCE_ID --clear-labels

## Analyze Spanner costs by labels in Cloud Billing

After you apply labels to your Spanner instances, you can use them to analyze your costs. The labels are forwarded to your cost data, allowing you to filter and group charges.

### View costs grouped by labels in billing reports

You can view and analyze your Spanner costs grouped by label keys directly in the Google Cloud console:

1.  In the Google Cloud console, go to the **Billing** section.
2.  Select your **Billing Account** and click **Reports** in the navigation pane.
3.  In the **Filters** panel:
    1.  Under **Services** , select **Spanner** to isolate your Spanner database costs.
    2.  Under **Group by** , select **Label keys** , and choose the label key you want to analyze (for example, `environment` or `instance_id` ).
4.  The chart and cost table updates to show a detailed breakdown of costs grouped by each label value (for example, `environment:prod` and `environment:dev` , or `instance_id:my-instance-1` and `instance_id:my-instance-2` ).
