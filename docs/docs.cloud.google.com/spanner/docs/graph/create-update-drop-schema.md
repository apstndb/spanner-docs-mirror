**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document provides a guide on managing Spanner Graph schemas, detailing the processes for [creating](#create-property-graph-schema) , [updating](#update-property-graph-schema) , and [dropping](#drop-property-graph-schema) schemas using DDL statements.

For more information about property graph schemas, see the [Spanner Graph schema overview](/spanner/docs/graph/schema-overview) . If you encounter errors when you create a property graph schema, see [Troubleshoot Spanner Graph](/spanner/docs/graph/troubleshoot) .

## Create a property graph schema

You can create a property graph schema using views or tables. To create a property graph schema using tables, do the following:

1.  [Create the node input tables](#create-node-input-tables) .
2.  [Create the edge input tables](#create-edge-input-tables) .
3.  [Define the property graph](#define-property-graph) .

To learn how to use SQL views to create a property graph schema, see [Create a Spanner Graph from a SQL view](/spanner/docs/graph/graph-with-views-how-to) .

When you create a property graph schema, be sure to consider the [best practices](/spanner/docs/graph/best-practices-designing-schema) . The following sections show how to use tables to create an example property graph schema:

### Create node input tables

The following creates two node input tables, `  Person  ` and `  Account  ` , which serve as input for the node definitions in the example property graph:

``` text
  CREATE TABLE Person (
    id               INT64 NOT NULL,
    name             STRING(MAX),
    birthday         TIMESTAMP,
    country          STRING(MAX),
    city             STRING(MAX),
  ) PRIMARY KEY (id);

  CREATE TABLE Account (
    id               INT64 NOT NULL,
    create_time      TIMESTAMP,
    is_blocked       BOOL,
    nick_name        STRING(MAX),
  ) PRIMARY KEY (id);
```

### Create edge input tables

The following code creates two edge input tables, `  PersonOwnAccount  ` and `  AccountTransferAccount  ` , as input for the edge definitions in the example property graph:

``` text
  CREATE TABLE PersonOwnAccount (
    id               INT64 NOT NULL,
    account_id       INT64 NOT NULL,
    create_time      TIMESTAMP,
    FOREIGN KEY (account_id) REFERENCES Account (id)
  ) PRIMARY KEY (id, account_id),
    INTERLEAVE IN PARENT Person ON DELETE CASCADE;

  CREATE TABLE AccountTransferAccount (
    id               INT64 NOT NULL,
    to_id            INT64 NOT NULL,
    amount           FLOAT64,
    create_time      TIMESTAMP NOT NULL,
    order_number     STRING(MAX),
    FOREIGN KEY (to_id) REFERENCES Account (id)
  ) PRIMARY KEY (id, to_id, create_time),
    INTERLEAVE IN PARENT Account ON DELETE CASCADE;
```

### Define a property graph

The following code uses tables and the `  CREATE PROPERTY GRAPH  ` statement to define a property graph. This statement defines a property graph named `  FinGraph  ` with `  Account  ` and `  Person  ` nodes, and `  PersonOwnAccount  ` and `  AccountTransferAccount  ` edges:

``` text
  CREATE PROPERTY GRAPH FinGraph
    NODE TABLES (
      Account,
      Person
    )
    EDGE TABLES (
      PersonOwnAccount
        SOURCE KEY (id) REFERENCES Person (id)
        DESTINATION KEY (account_id) REFERENCES Account (id)
        LABEL Owns,
      AccountTransferAccount
        SOURCE KEY (id) REFERENCES Account (id)
        DESTINATION KEY (to_id) REFERENCES Account (id)
        LABEL Transfers
    );
```

## Update a property graph schema

After you create a property graph schema, you update it by using the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement. This statement applies the changes by recreating the graph schema with the desired update.

You can make the following changes to a property graph schema:

  - [Add a node or edge definition](#add-new-node-or-edge) : Create the new input tables for the nodes and edges, and then use the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement to add the new definitions to the graph.

  - [Update a node or edge definition](#update-node-or-edge) : Update the underlying input table with new node and edge definitions. Then, use the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement to update the definitions in the graph.

  - [Remove a node or edge definition](#remove-node-or-edge) : Use the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement and omit the definitions that you want to remove from the graph.

### Add new node or edge definitions

To add a new node and a new edge definition, follow these steps:

1.  Add a new node definition input table, `  Company  ` , and a new edge definition input table, `  PersonInvestCompany  ` .
    
    ``` text
    CREATE TABLE Company (
      id INT64 NOT NULL,
      name STRING(MAX)
    ) PRIMARY KEY (id);
    
    CREATE TABLE PersonInvestCompany (
      id INT64 NOT NULL,
      company_id INT64 NOT NULL,
      FOREIGN KEY (company_id) REFERENCES Company (id)
    ) PRIMARY KEY (id, company_id),
      INTERLEAVE IN PARENT Person ON DELETE CASCADE;
    ```

2.  Update the `  FinGraph  ` schema by adding the new `  Company  ` node definition and the new `  PersonInvestCompany  ` edge definition.
    
    ``` text
    CREATE OR REPLACE PROPERTY GRAPH FinGraph
      NODE TABLES (
        Person,
        Account,
        Company
      )
      EDGE TABLES (
        AccountTransferAccount
          SOURCE KEY (id) REFERENCES Account
          DESTINATION KEY (to_id) REFERENCES Account
          LABEL Transfers,
        PersonOwnAccount
          SOURCE KEY (id) REFERENCES Person
          DESTINATION KEY (account_id) REFERENCES Account
          LABEL Owns,
        PersonInvestCompany
          SOURCE KEY (id) REFERENCES Person
          DESTINATION KEY (company_id) REFERENCES Company
          LABEL Invests
      );
    ```

### Update node or edge definitions

To update an existing node or edge definition, you first alter the underlying input table, and then use the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement to apply the schema changes to the graph. To customize the properties exposed from the input tables, use the [`  PROPERTIES clause  `](/spanner/docs/reference/standard-sql/graph-schema-statements#element_table_property_definition) . For more information, see [Customize labels and properties](/spanner/docs/graph/schema-overview#customize-labels-properties) .

The following steps show how to update the underlying table of a schema, then apply the update to the schema.

1.  Add the `  mailing_address  ` column to the `  Person  ` underlying input table.
    
    ``` text
    ALTER TABLE Person
    ADD COLUMN mailing_address STRING(MAX);
    ```

2.  Apply the changes to the `  Person  ` table to the schema. Use the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement. The `  Person  ` node definition reflects the updated `  Person  ` table definition because the input table schema changed.
    
    ``` text
    CREATE OR REPLACE PROPERTY GRAPH FinGraph
      NODE TABLES (
        Person,
        Account
      )
      EDGE TABLES (
        AccountTransferAccount
          SOURCE KEY (id) REFERENCES Account
          DESTINATION KEY (to_id) REFERENCES Account
          LABEL Transfers,
        PersonOwnAccount
          SOURCE KEY (id) REFERENCES Person
          DESTINATION KEY (account_id) REFERENCES Account
          LABEL Owns
      );
    ```

### Remove node or edge definitions

To remove existing node or edge definitions, recreate the property graph without those node or edge tables.

The following removes the `  Person  ` node definition and the `  PersonOwnAccount  ` edge definition by omitting them in the `  CREATE OR REPLACE PROPERTY GRAPH  ` statement.

``` text
  CREATE OR REPLACE PROPERTY GRAPH FinGraph
    NODE TABLES (
      Account
    )
    EDGE TABLES (
      AccountTransferAccount
        SOURCE KEY (id) REFERENCES Account
        DESTINATION KEY (to_id) REFERENCES Account
        LABEL Transfers
    );
```

## Drop a property graph schema

To drop a graph schema from the underlying input tables, use the `  DROP PROPERTY GRAPH  ` DDL statement. You can't delete the data from the underlying table when you drop a schema.

The following code drops the `  FinGraph  ` property graph schema:

``` text
DROP PROPERTY GRAPH FinGraph;
```

## What's next

  - [Manage Spanner Graph data](/spanner/docs/graph/insert-update-delete-data) .
  - [Learn about Spanner Graph queries](/spanner/docs/graph/queries-overview) .
  - [Learn best practices for tuning Spanner Graph queries](/spanner/docs/graph/best-practices-tuning-queries) .
