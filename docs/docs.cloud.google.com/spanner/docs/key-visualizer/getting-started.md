Key Visualizer is enabled in Spanner by default.

## Before you begin

To view Key Visualizer, you need the following [Identity and Access Management (IAM) permission](../iam) :

  - `  spanner.databases.read  `

If you are a fine-grained access control user, you must have been granted access to:

  - the `  spanner_sys_reader  ` system role or one of its member roles.

For more information, see [About fine-grained access control](../fgac-about) and [Fine-grained access control system roles](../fgac-system-roles) .

## Access the Key Visualizer interface

You access the Key Visualizer tool from the [Google Cloud console](https://console.cloud.google.com/) .

To access Key Visualizer:

1.  From the [Spanner page](https://console.cloud.google.com/spanner/) of the Google Cloud console, select an instance.

2.  Select a database to investigate.

3.  In the navigation menu, under **Observability** , select **Key Visualizer** .

## Disable Key Visualizer

Key Visualizer is controlled by the `  enable_key_visualizer  ` database option. The default value is `  true  ` .

To disable Key Visualizer, set the value of the `  enable_key_visualizer  ` database option to `  false  ` . The DDL syntax to disable Key Visualizer is:

``` text
ALTER DATABASE `database_id` SET OPTIONS (enable_key_visualizer=false)
```

If your database ID contains characters other than letters, numbers, or underscores, be sure to enclose the ID with backticks (\`\`).

The statement can be sent using a gcloud command, or in an `  UpdateDatabaseDdl  ` gRPC/REST request. For example:

``` text
gcloud spanner databases ddl update database_id --instance=instance_id \
    --ddl='ALTER DATABASE `database_id` SET OPTIONS ( enable_key_visualizer=false )'
```

Once you've explicitly set the value for `  enable_key_visualizer  ` , you can check its value by clicking **SHOW EQUIVALENT DDL** in the overview page, or using the `  ddl describe  ` gcloud command:

``` text
gcloud spanner databases ddl describe `database_id` --instance=instance_id
```

## Re-enable Key Visualizer

To re-enable Key Visualizer, follow the previous instructions, setting the value of the `  enable_key_visualizer  ` database option to `  true  ` .

When first re-enabled for a database, the Key Visualizer heatmap will be empty for the first few minutes while Spanner collects metrics.

## What's next

  - Learn to recognize [common patterns in heatmaps](/spanner/docs/key-visualizer/patterns) .
  - Find out how to [explore a heatmap in depth](/spanner/docs/key-visualizer/exploring-heatmaps) .
  - Read about the [metrics you can view in a heatmap](/spanner/docs/key-visualizer/metrics) .
