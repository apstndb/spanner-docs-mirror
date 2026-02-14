**Preview — Data agents**

This feature is subject to the "Pre-GA Offerings Terms" in the General Service Terms section of the [Service Specific Terms](/terms/service-terms#1) , and the [Additional Terms for Generative AI Preview Products](https://cloud.google.com/trustedtester/aitos) . Pre-GA features are available "as is" and might have limited support. For more information, see the [launch stage descriptions](https://cloud.google.com/products/#product-launch-stages) .

For information about access to this release, see the [access request page](https://forms.gle/pJByTWfenZAWbaXo7) .

**PostgreSQL interface note:** The examples in this topic are intended for GoogleSQL-dialect databases. This feature doesn't support PostgreSQL interface.

This document describes how to create a data agent in Spanner Studio using an agent context file. A data agent is associated with a single agent context file.

To learn about data agents, see [Data agents overview](/spanner/docs/data-agent-overview) .

## Before you begin

Complete the following prerequisites before creating an agent.

### Enable required services

Enable the following services for your project:

  - [Gemini Data Analytics API](https://console.cloud.google.com/apis/library/geminidataanalytics.googleapis.com)
  - [Gemini for Google Cloud API](https://console.cloud.google.com/apis/library/cloudaicompanion.googleapis.com)

### Prepare a Spanner instance

  - Make sure that a Spanner instance is available. For more information, see [Create an instance](/spanner/docs/create-manage-instances) .
  - Ensure that you create a database in your instance where you will create the tables. For more information, see [Create a database on the Spanner instance](/spanner/docs/create-manage-databases#create-database)

### Required roles and permissions

  - Add an IAM user or service account to the cluster. For more information, see [Apply IAM roles](/spanner/docs/grant-permissions) .
  - Grant the `  spanner.databaseReader  ` roles to the IAM user at the project level. For more information, see [Add IAM policy binding for a project](/sdk/gcloud/reference/projects/add-iam-policy-binding) .
  - [Grant roles and permissions](/spanner/docs/grant-permissions#project-level_permissions) to the IAM user at the project-level for the required databases.

## Create a data agent

To create a data agent, perform the following steps:

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , next to **Data Agents** , click **View actions** .

5.  Click **Create agent** .

6.  In **Agent name** , provide a unique agent name. The agent name is case-sensitive and can contain letters, numbers, hyphens, and underscores.

7.  Optional. In **Agent description** , add a description for your agent.

8.  Optional. Click **Show Advanced Options** and in **Select a location** , select a region for storing agent context. You can select from the following list of supported regions:
    
      - `  us-central1  `
      - `  us-east1  `
      - `  europe-west4  `
      - `  asia-southeast1  `

9.  Click **Create** .

**Note:** Creating the first agent in a project can take several minutes.

## Build agent context

After creating an agent, follow the steps in [Build contexts using Gemini CLI](/spanner/docs/build-context-gemini-cli) to create an agent context file. You can then edit your agent to upload the context file.

## Edit an agent

To edit a data agent, perform the following steps:

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , next to **Data Agents** , click **View actions** .

5.  Click **Edit agent** .

6.  Optional: Edit **Agent description** .

7.  Click **Browse** in the **Upload agent context file** section, and select the agent context file.

8.  Click **Save** .

## Delete a data agent

To delete a data agent, perform the following steps:

1.  In the Google Cloud console, go to the Spanner page.

2.  Select an instance from the list, and then select a database.

3.  In the navigation menu, click **Spanner Studio** .

4.  In the **Explorer pane** , next to **Data Agents** , click **View actions** .

5.  Click **Delete agent** .

6.  In the **Delete agent** confirmation dialog, enter the name of the agent.

7.  Click **Confirm** to delete the agent.

**Note:** Before you delete a database, you must delete all agents associated with that database.

## What's next

  - Learn more about [data agents](/spanner/docs/data-agent-overview) .
  - Learn how to [inspect and call a data agent](/spanner/docs/inspect-data-agent) .
