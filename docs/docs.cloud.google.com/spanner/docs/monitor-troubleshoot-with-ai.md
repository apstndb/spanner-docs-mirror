---
name: documents/docs.cloud.google.com/spanner/docs/monitor-troubleshoot-with-ai
uri: https://docs.cloud.google.com/spanner/docs/monitor-troubleshoot-with-ai
title: Monitor and troubleshoot with AI assistance
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Important** : As of April 10, 2026, you can create, run, and edit [Gemini Cloud Assist investigations](https://docs.cloud.google.com/cloud-assist/investigations) only if you have a [Premium Support contract](https://cloud.google.com/support/premium) .

> **Preview**
> 
> This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](https://docs.cloud.google.com/terms/service-terms#1) . You can process personal data for this feature as outlined in the [Cloud Data Processing Addendum](https://docs.cloud.google.com/terms/data-processing-addendum) , subject to the obligations and restrictions described in the agreement under which you access Google Cloud. Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

This document describes how you can use AI assistance to help you monitor and troubleshoot your Spanner resources. You can use the AI-assisted troubleshooting tools of Spanner and [Gemini Cloud Assist](https://docs.cloud.google.com/cloud-assist/overview) to [troubleshoot high database load](https://docs.cloud.google.com/spanner/docs/monitor-troubleshoot-with-ai#troubleshoot-high-database-load) .

> As an early-stage technology, Gemini for Google Cloud products can generate output that seems plausible but is factually incorrect. We recommend that you validate all output from Gemini for Google Cloud products before you use it. For more information, see [Gemini for Google Cloud and responsible AI](https://docs.cloud.google.com/gemini/docs/discover/responsible-ai) .

## Before you begin

[Set up Gemini Cloud Assist for your Google Cloud user account and project](https://docs.cloud.google.com/cloud-assist/set-up-gemini) .

After you set up Gemini Cloud Assist, the service can take a few minutes to propagate. Wait for propagation to complete before you enable AI-assisted troubleshooting in Spanner.

### Required roles and permissions

To get the permissions that you need to use AI-assisted troubleshooting, ask your administrator to grant you the following IAM roles on the project that hosts the Spanner instance:

  - [Cloud Spanner Database User](https://docs.cloud.google.com/iam/docs/roles-permissions/spanner#spanner.databaseUser) ( `roles/spanner.databaseUser` )
  - [Database Insights viewer](https://docs.cloud.google.com/iam/docs/roles-permissions/databaseinsights#databaseinsights.viewer) ( `roles/databaseinsights.viewer` )
  - [Gemini Cloud Assist Investigation Owner](https://docs.cloud.google.com/iam/docs/roles-permissions/geminicloudassist#geminicloudassist.investigationOwner) ( `roles/geminicloudassist.investigationOwner` )

For more information about granting roles, see [Manage access to projects, folders, and organizations](https://docs.cloud.google.com/iam/docs/granting-changing-revoking-access) .

You might also be able to get the required permissions through [custom roles](https://docs.cloud.google.com/iam/docs/creating-custom-roles) or other [predefined roles](https://docs.cloud.google.com/iam/docs/roles-overview#predefined) .

For more information about required roles and permissions for using Gemini Cloud Assist investigations, see [Troubleshoot issues with Gemini Cloud Assist Investigations](https://docs.cloud.google.com/cloud-assist/investigations#consideration) .

## Open Gemini Cloud Assist

To use Gemini Cloud Assist with Spanner, do the following:

1.  In the Google Cloud console, go to the **Spanner Instances** page.

2.  To open the **Overview** page of an instance, click the instance name.

3.  To open the **[Cloud Assist panel](https://docs.cloud.google.com/cloud-assist/chat-panel)** , click auto\_awesome **Open or close Gemini Cloud Assist chat** .

4.  In the **Cloud Assist panel** , enter a prompt that describes the information that you're interested in.

5.  After you enter the prompt, click send **Send prompt** .
    
    Gemini returns a response to your prompt based on information from the last hour.

## Troubleshoot high database load

By accessing the **Query insights** dashboard or the **System insights** dashboard in the Google Cloud console, you can analyze your database and troubleshoot events when your system experiences a higher database load than average. Spanner uses the 24 hours of data that occurs prior to your selected time range to calculate the expected load of your database. You can look into the reasons for the higher load events and analyze the evidence behind reduced performance. Spanner also provides recommendations for optimizing your database to improve performance.

To use AI assistance with troubleshooting high database load, go to the **System insights** dashboard or the **Query insights** dashboard in the Google Cloud console.

### Query insights dashboard

Troubleshoot high database load with AI assistance in the **Query insights** dashboard using the following steps:

In the Google Cloud console, go to the **Spanner instances** page.

To open the **Overview** page of an instance, click the instance name.

Optional: In the **Databases** list, click a database.

Click **Query insights** to open the **Query insights** dashboard.

Optional: Use the **Time range** filter to select either 1 hour, 6 hours, 1 day, 7 days, 30 days or a custom range.

You can zoom in to specific sections of the chart where you notice areas of higher database load by query execution time. To zoom in, you can click and select a portion of the chart.

In the **Total CPU Utilization (All Queries)** chart, click the **Investigate performance** button to start troubleshooting high database load with AI assistance from [Gemini Cloud Assist](https://docs.cloud.google.com/cloud-assist/investigations) .

After about two minutes, the **Investigation details** pane opens with the following sections:

  - **Issue** . A description of the issue being investigated, including the investigation’s start and stop time.
  - **Observations** . A list of observations about the issue. For example, these can include lock contention details, such as a longer than expected lock wait ratio for the query.
  - **Hypotheses** . A list of AI-recommended actions to take to help address the slow running query.

### System insights dashboard

Troubleshoot high database load with AI assistance in the **System insights** dashboard using the following steps:

1.  In the Google Cloud console, go to the **Spanner instances** page.

2.  To open the **Overview** page of an instance, click the instance name.

3.  Optional: Under **Databases** , click a database.

4.  In the navigation menu, click **System insights** .

5.  Optional: Use the **Time range** filter to select either 1 hour, 6 hours, 1 day, 7 days, 30 days or a custom range.
    
    You can zoom in to specific sections of the chart where you notice areas of high load that you want to analyze. For example, an area of high load might display CPU utilization levels closer to 100%. To zoom in, you can click and select a portion of the chart.
    
    Click the **Explore Investigations** button to start troubleshooting database load with AI assistance from [Gemini Cloud Assist](https://docs.cloud.google.com/cloud-assist/investigations) .
    
    After about two minutes, the **Investigation details** pane opens with the following sections:
    
      - **Issue** . A description of the issue being investigated, including the investigation's start and stop time.
      - **Observations** . A list of observations about the issue. For example, these can include lock contention details, such as a longer than expected lock wait ratio for the query.
      - **Hypotheses** . A list of AI-recommended actions to take to help address the slow running query.

### Analyze high database load

Using AI assistance, you can analyze and troubleshoot the details of your database load.

#### Analysis time period

Spanner analyzes your database for the time period that you select in your database load chart from the **Query insights** dashboard or the **System insights** dashboard. If you select a time period of less than 24 hours, then Spanner analyzes the entire time period. If you select a time period greater than 24 hours, then Spanner selects only the last 24 hours of the time period for analysis.

To calculate the baseline performance analysis of your database, Spanner includes 24 hours of a baseline time period in its analysis time period. If your selected time period occurs on a day other than Monday, then Spanner uses a baseline time period of the *24 hours previous* to your selected time period. If your selected time period occurs on a Monday, then Spanner uses a baseline time period of the *7th day previous* to your selected time period.

#### Metrics analysis

When Spanner starts the analysis, Spanner checks for significant changes in the various metrics, including but not limited to the following:

  - CPU utilization
  - Read and write latencies, P50 and P99
  - Read and write queries per second (QPS)
  - Node count
  - Session metrics
  - Lock wait time
  - Transaction abort count
  - Query statistics
  - Transaction statistics
  - Lock statistics
  - Split statistics

Spanner compares the baseline aggregated data for your database within the performance data of your analysis time window. If Spanner detects a significant change in threshold for a key metric, then Spanner indicates a possible situation with your database. The identified situation might explain a root cause for the high load on your database over the selected time period.

#### Recommendations

When Gemini Cloud Assist completes analysis, the **Hypotheses** section of the **Investigation details** pane lists actionable insights to help remediate the issue.

For some situations, based on the analysis, there might not be a recommendation.

## What's next

  - Learn how to [write better prompts](https://docs.cloud.google.com/gemini/docs/discover/write-prompts) .

  - Learn how to use the [Gemini Cloud Assist panel](https://docs.cloud.google.com/cloud-assist/chat-panel) .

  - Read [Use Gemini for AI assistance and development](https://docs.cloud.google.com/gemini/docs/overview)

  - Learn [how and when Gemini for Google Cloud uses your data](https://docs.cloud.google.com/gemini/docs/discover/data-governance) .

  - [Write SQL with Gemini assistance](https://docs.cloud.google.com/spanner/docs/write-sql-gemini) .

  - [Understand latency metrics](https://docs.cloud.google.com/spanner/docs/latency-metrics) .

  - [Investigate high CPU utilization](https://docs.cloud.google.com/spanner/docs/cpu-utilization) .

  - [Learn more about Spanner performance](https://docs.cloud.google.com/spanner/docs/performance) .

  - [Monitor instances with system insights](https://docs.cloud.google.com/spanner/docs/monitoring-console) .
