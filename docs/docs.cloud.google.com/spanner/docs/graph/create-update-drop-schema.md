> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This document provides a guide on managing Spanner Graph schemas, detailing the processes for [creating](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#create-property-graph-schema) , [updating](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#update-property-graph-schema) , and [dropping](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#drop-property-graph-schema) schemas using DDL statements.

For more information about property graph schemas, see the [Spanner Graph schema overview](https://docs.cloud.google.com/spanner/docs/graph/schema-overview) . If you encounter errors when you create a property graph schema, see [Troubleshoot Spanner Graph](https://docs.cloud.google.com/spanner/docs/graph/troubleshoot) .

## Create a property graph schema

You can create a property graph schema using views or tables. To create a property graph schema using tables, do the following:

1.  [Create the node input tables](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#create-node-input-tables) .
2.  [Create the edge input tables](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#create-edge-input-tables) .
3.  [Define the property graph](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#define-property-graph) .

To learn how to use SQL views to create a property graph schema, see [Create a Spanner Graph from a SQL view](https://docs.cloud.google.com/spanner/docs/graph/graph-with-views-how-to) .

When you create a property graph schema, be sure to consider the [best practices](https://docs.cloud.google.com/spanner/docs/graph/best-practices-designing-schema) . The following sections show how to use tables to create an example property graph schema:

### Create node input tables

The following creates two node input tables, `Person` and `Account` , which serve as input for the node definitions in the example property graph:

``` 
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

The following code creates two edge input tables, `PersonOwnAccount` and `AccountTransferAccount` , as input for the edge definitions in the example property graph:

``` 
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

The following code uses tables and the `CREATE PROPERTY GRAPH` statement to define a property graph. This statement defines a property graph named `FinGraph` with `Account` and `Person` nodes, and `PersonOwnAccount` and `AccountTransferAccount` edges:

``` 
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

After you create a property graph schema, you update it by using the `CREATE OR REPLACE PROPERTY GRAPH` statement. This statement applies the changes by recreating the graph schema with the desired update.

You can make the following changes to a property graph schema:

  - [Add a node or edge definition](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#add-new-node-or-edge) : Create the new input tables for the nodes and edges, and then use the `CREATE OR REPLACE PROPERTY GRAPH` statement to add the new definitions to the graph.

  - [Update a node or edge definition](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#update-node-or-edge) : Update the underlying input table with new node and edge definitions. Then, use the `CREATE OR REPLACE PROPERTY GRAPH` statement to update the definitions in the graph.

  - [Remove a node or edge definition](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema#remove-node-or-edge) : Use the `CREATE OR REPLACE PROPERTY GRAPH` statement and omit the definitions that you want to remove from the graph.

### Add new node or edge definitions

To add a new node and a new edge definition, follow these steps:

1.  Add a new node definition input table, `Company` , and a new edge definition input table, `PersonInvestCompany` .
    
        CREATE TABLE Company (
          id               INT64 NOT NULL,
          name             STRING(MAX),
        ) PRIMARY KEY (id);
        
        CREATE TABLE PersonInvestCompany (
          id               INT64 NOT NULL,
          company_id       INT64 NOT NULL,
          FOREIGN KEY (company_id) REFERENCES Company (id)
        ) PRIMARY KEY (id, company_id),
          INTERLEAVE IN PARENT Person ON DELETE CASCADE;

2.  Update the `FinGraph` schema by adding the new `Company` node definition and the new `PersonInvestCompany` edge definition.
    
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

### Update node or edge definitions

To update an existing node or edge definition, you first alter the underlying input table, and then use the `CREATE OR REPLACE PROPERTY GRAPH` statement to apply the schema changes to the graph. To customize the properties exposed from the input tables, use the [`PROPERTIES clause`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-schema-statements#element_table_property_definition) . For more information, see [Customize labels and properties](https://docs.cloud.google.com/spanner/docs/graph/schema-overview#customize-labels-properties) .

The following steps show how to update the underlying table of a schema, then apply the update to the schema.

1.  Add the `mailing_address` column to the `Person` underlying input table.
    
        ALTER TABLE Person
        ADD COLUMN mailing_address STRING(MAX);

2.  Apply the changes to the `Person` table to the schema. Use the `CREATE OR REPLACE PROPERTY GRAPH` statement. The `Person` node definition reflects the updated `Person` table definition because the input table schema changed.
    
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

### Remove node or edge definitions

To remove existing node or edge definitions, recreate the property graph without those node or edge tables.

The following removes the `Person` node definition and the `PersonOwnAccount` edge definition by omitting them in the `CREATE OR REPLACE PROPERTY GRAPH` statement.

``` 
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

To drop a graph schema from the underlying input tables, use the `DROP PROPERTY GRAPH` DDL statement. You can't delete the data from the underlying table when you drop a schema.

The following code drops the `FinGraph` property graph schema:

    DROP PROPERTY GRAPH FinGraph;

## Create a property graph in a named schema

Spanner Graph supports creating property graphs within a [named schema](https://docs.cloud.google.com/spanner/docs/schema-and-data-model#named-schemas) .

The following example creates two named schemas `sch1` and `sch2` , a node and edge table in the `sch1` schema, and a node and property graph in the `sch2` schema:

    CREATE SCHEMA sch1;
    CREATE SCHEMA sch2;
    
    CREATE TABLE sch1.Person (
      id               INT64 NOT NULL,
      name             STRING(MAX),
      birthday         TIMESTAMP,
      country          STRING(MAX),
      city             STRING(MAX),
    ) PRIMARY KEY (id);
    
    CREATE TABLE sch2.Account (
      id               INT64 NOT NULL,
      create_time      TIMESTAMP,
      is_blocked       BOOL,
      nick_name        STRING(MAX),
    ) PRIMARY KEY (id);
    
    CREATE TABLE sch1.PersonOwnAccount (
      id               INT64 NOT NULL,
      account_id       INT64 NOT NULL,
      create_time      TIMESTAMP,
      FOREIGN KEY (account_id) REFERENCES sch2.Account (id)
    ) PRIMARY KEY (id, account_id),
      INTERLEAVE IN PARENT sch1.Person ON DELETE CASCADE;
    
    CREATE OR REPLACE PROPERTY GRAPH sch2.FinGraph
      NODE TABLES (
        sch1.Person,
        sch2.Account
      )
      EDGE TABLES (
        sch1.PersonOwnAccount
          SOURCE KEY (id) REFERENCES Person (id)
          DESTINATION KEY (account_id) REFERENCES Account (id)
          LABEL Owns
      );

## What's next

  - [Manage Spanner Graph data](https://docs.cloud.google.com/spanner/docs/graph/insert-update-delete-data) .
  - [Learn about Spanner Graph queries](https://docs.cloud.google.com/spanner/docs/graph/queries-overview) .
  - [Learn best practices for tuning Spanner Graph queries](https://docs.cloud.google.com/spanner/docs/graph/best-practices-tuning-queries) .
