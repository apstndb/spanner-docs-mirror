This page explains how [fine-grained access control](/spanner/docs/fgac-about) works with Spanner models.

For fine-grained access control users, you can control access to MODEL entities with the following privilege:

  - Grant `  EXECUTE  ` on the model to allow [machine learning functions](/spanner/docs/reference/standard-sql/ml-functions) to use it.
    
    ### GoogleSQL
    
    ``` text
    GRANT EXECUTE ON MODEL MODEL_NAME TO ROLE ROLE_NAME;
    ```

## `     INFORMATION_SCHEMA    ` views for models

The following views show the database roles and privileges information for models:

  - GoogleSQL-dialect databases: [`  INFORMATION_SCHEMA.MODEL_PRIVILEGES  `](/spanner/docs/information-schema#model-privileges)

The rows in this view are filtered based on the current database role's privileges on models. This ensures that principals can view only the roles, privileges, and models that they have access to.

Row filtering also applies to the following model-related views:

### GoogleSQL

  - [`  INFORMATION_SCHEMA.MODELS  `](/spanner/docs/information-schema#models)
  - [`  INFORMATION_SCHEMA.MODEL_OPTIONS  `](/spanner/docs/information-schema#model-options)
  - [`  INFORMATION_SCHEMA.MODEL_COLUMNS  `](/spanner/docs/information-schema#model-columns)
  - [`  INFORMATION_SCHEMA.MODEL_COLUMN_OPTIONS  `](/spanner/docs/information-schema#model-column-options)

The system role `  spanner_info_reader  ` and its members always see an unfiltered `  INFORMATION_SCHEMA  ` .

## More information

  - [About models](/spanner/docs/ml)
  - [Create and manage models](/spanner/docs/ml-tutorial)
  - [About fine-grained access control](/spanner/docs/fgac-about)
