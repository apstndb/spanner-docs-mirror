**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document describes best practices for designing a Spanner Graph schema, focusing on efficient queries, optimized edge traversal, and effective data management techniques.

For information about the design of Spanner schemas (not Spanner Graph schemas), see [Schema design best practices](/spanner/docs/schema-design) .

## Choose a schema design

Your schema design affects graph performance. The following topics help you choose an effective strategy.

### Schematized versus schemaless designs

  - A schematized design stores the graph definition in the Spanner Graph schema, which is suitable for stable graphs with infrequent definition changes. The schema enforces the graph definition, and properties support all Spanner data types.

  - A [schemaless](/spanner/docs/graph/manage-schemaless-data) design infers the graph definition from the data, offering more flexibility without requiring schema changes. Dynamic labels and properties aren't enforced by default. Properties must be valid JSON values.

The following summarizes the primary differences between schema and schemaless data management. Also [consider your graph queries](#choose-design-based-on-queries) to help decide which type of schema to use.

<table>
<thead>
<tr class="header">
<th>Feature</th>
<th>Schematized data management</th>
<th>Schemaless data management</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Storing graph definition</td>
<td>Graph definition is stored in the Spanner Graph schema.</td>
<td>Graph definition is evident from the data. However, Spanner Graph doesn't inspect the data to infer the definition.</td>
</tr>
<tr class="even">
<td>Updating graph definition</td>
<td>Requires a Spanner Graph schema change. Suitable when the definition is well-defined and changes infrequently.</td>
<td>No Spanner Graph schema change needed.</td>
</tr>
<tr class="odd">
<td>Enforcing graph definition</td>
<td>A property graph schema enforces the allowed node types for an edge. It also enforces the allowed properties and property types of a graph node or edge type.</td>
<td>Not enforced by default. You can use <a href="/spanner/docs/graph/manage-schemaless-data#constraints">check constraints</a> to enforce label and property data integrity.</td>
</tr>
<tr class="even">
<td>Property data types</td>
<td>Support any Spanner data type, for example, <code dir="ltr" translate="no">       timestamp      </code> .</td>
<td><a href="/spanner/docs/graph/manage-schemaless-data#create-a-property-graph">Dynamic properties</a> must be a valid JSON value.</td>
</tr>
</tbody>
</table>

### Choose a schema design based on graph queries

Schematized and schemaless designs often offer comparable performance. However, when queries use [quantified path patterns](/spanner/docs/graph/queries-overview#quantified-path-patterns) that span multiple node or edge types, a schemaless design offers better performance.

The underlying data model is a key reason for this. A schemaless design stores all data in single node and edge tables, which `  DYNAMIC LABEL  ` [enforces](/spanner/docs/graph/manage-schemaless-data#single-table-dynamic-label) . Queries that traverse multiple types execute with minimal table scans.

In contrast, schematized designs commonly use separate tables for each node and edge type, so queries spanning multiple types must scan and combine data from all corresponding tables.

The following are sample queries that work well with schemaless designs, and a sample query that works well with both designs:

### Schemaless design

The following queries perform better with a schemaless design because they use quantified path patterns that can match multiple types of nodes and edges:

  - This query's quantified path pattern uses multiple edge types ( `  Transfer  ` or `  Withdraw  ` ) and doesn't specify intermediate node types for paths longer than one hop.
    
    ``` text
    GRAPH FinGraph
    MATCH p = (:Account {id:1})-[:Transfer|Withdraw]->{1,3}(:Account)
    RETURN TO_JSON(p) AS p;
    ```

  - This query's quantified path pattern finds paths of one to three hops between `  Person  ` and `  Account  ` nodes, using multiple edge types ( `  Owns  ` or `  Transfers  ` ), without specifying intermediate node types for longer paths. This allows paths to traverse intermediate nodes of various types. For example, `  (:Person)-[:Owns]->(:Account)-[:Transfers]->(:Account)  ` .
    
    ``` text
    GRAPH FinGraph
    MATCH p = (:Person {id:1})-[:Owns|Transfers]->{1,3}(:Account)
    RETURN TO_JSON(p) AS p;
    ```

  - This query's quantified path pattern finds paths of one to three hops between `  Person  ` and `  Account  ` nodes, without specifying any edge labels. Similar to the prior query, it allows paths to traverse intermediate nodes of various types.
    
    ``` text
    GRAPH FinGraph
    MATCH p = (:Person {id:1})-[]->{1,3}(:Account)
    RETURN TO_JSON(p) AS p;
    ```

  - This query finds paths of one to three hops between `  Account  ` nodes using edges of type `  Owns  ` in any direction ( `  -[:Owns]-  ` ). Because paths can traverse edges in either direction and intermediate nodes aren't specified, a two-hop path might go through nodes of different types. For example, `  (:Account)-[:Owns]-(:Person)-[:Owns]-(:Account)  ` .
    
    ``` text
    GRAPH FinGraph
    MATCH p = (:Account {id:1})-[:Owns]-{1,3}(:Account)
    RETURN TO_JSON(p) AS p;
    ```

### Both designs

The following query performs comparably with both schematized and schemaless designs. Its quantified path, `  (:Account)-[:Transfer]->{1,3}(:Account)  ` , involves one node type, `  Account  ` , and one edge type, `  Transfer  ` . Because the path involves only one node type and one edge type, performance is comparable for both designs. Even though intermediate nodes aren't explicitly labeled, the pattern constrains them to be `  Account  ` nodes. The `  Person  ` node appears outside of this quantified path.

``` text
GRAPH FinGraph
MATCH p = (:Person {id:1})-[:Owns]->(:Account)-[:Transfer]->{1,3}(:Account)
RETURN TO_JSON(p) AS p;
```

## Optimize Spanner Graph schema performance

After you choose to use a schematized or schemaless Spanner Graph schema, you can optimize its performance in the following ways:

  - [Optimize its edge traversal](#optimize-edge-traversal)
  - [Use secondary indexes to filter properties](#use-secondary-indexes)

### Optimize edge traversal

Edge traversal is the process of navigating through a graph by following its edges, starting at a particular node and moving along connected edges to reach other nodes. The schema defines the direction of the edge. Edge traversal is a fundamental operation in Spanner Graph, so improving edge traversal efficiency can significantly improve your application's performance.

You can traverse an edge in two directions:

  - *Forward edge traversal* follows outgoing edges of the source node.
  - *Reverse edge traversal* follows incoming edges of the destination node.

#### Forward and reverse edge traversal query examples

The following example query performs forward edge traversal of `  Owns  ` edges for a given person:

``` text
GRAPH FinGraph
MATCH (person:Person {id: 1})-[owns:Owns]->(accnt:Account)
RETURN accnt.id;
```

The following example query performs reverse edge traversal of `  Owns  ` edges for a given account:

``` text
GRAPH FinGraph
MATCH (accnt:Account {id: 1})<-[owns:Owns]-(person:Person)
RETURN person.name;
```

#### Optimize forward edge traversal

To improve forward edge traversal performance, optimize traversal from source to edge and from edge to destination.

  - To optimize source to edge traversal, interleave the edge input table into the source node input table using the `  INTERLEAVE IN PARENT  ` clause. Interleaving is a storage optimization technique in Spanner that colocates child table rows with their corresponding parent rows in storage. For more information about interleaving, see [Schemas overview](/spanner/docs/schema-and-data-model) .

  - To optimize edge to destination traversal, create a [foreign key constraint](#use-ref-constraints) between the edge and the destination  
    node. This enforces the edge-to-destination constraint, which can improve performance by eliminating destination table scans. If enforced foreign keys cause write performance bottlenecks (for example, when updating hub nodes), use an [informational foreign key](/spanner/docs/foreign-keys/overview#informational-foreign-keys) instead.

The following examples show how to use interleaving with an enforced and an informational foreign key constraint.

### Enforced foreign key

In this edge table example, `  PersonOwnAccount  ` does the following:

  - Interleaves into the source node table `  Person  ` .

  - Creates an enforced foreign key to the destination node table `  Account  ` .

<!-- end list -->

``` text
CREATE TABLE Person (
  id               INT64 NOT NULL,
  name             STRING(MAX),
) PRIMARY KEY (id);

CREATE TABLE Account (
  id               INT64 NOT NULL,
  create_time      TIMESTAMP,
  close_time       TIMESTAMP,
) PRIMARY KEY (id)

CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
  CONSTRAINT FK_Account FOREIGN KEY (account_id)
    REFERENCES Account (id)
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;
```

### Informational foreign key

In this edge table example, `  PersonOwnAccount  ` does the following:

  - Interleaves into the source node table `  Person  ` .

  - Creates an informational foreign key to the destination node table `  Account  ` .

<!-- end list -->

``` text
CREATE TABLE Person (
  id               INT64 NOT NULL,
  name             STRING(MAX),
) PRIMARY KEY (id);

CREATE TABLE Account (
  id               INT64 NOT NULL,
  create_time      TIMESTAMP,
  close_time       TIMESTAMP,
) PRIMARY KEY (id)

CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
  CONSTRAINT FK_Account FOREIGN KEY (account_id)
    REFERENCES Account (id) NOT ENFORCED
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;
```

#### Optimize reverse edge traversal

Optimize reverse edge traversal unless your queries use only forward traversal, because queries involving reverse or [any-directional](/spanner/docs/graph/queries-overview#any-direction-edge-pattern) traversal are common.

To optimize reverse edge traversal, you can do the following:

  - Create a secondary index on the edge table.

  - Interleave the index into the destination node input table to colocate the edges with the destination nodes.

  - Store the edge properties in the index.

This example shows a secondary index to optimize reverse edge traversal for the edge table `  PersonOwnAccount  ` :

  - The `  INTERLEAVE IN  ` clause colocates the index data with the destination node table `  Account  ` .

  - The `  STORING  ` clause stores edge properties in the index.

For more information about interleaving indexes, see [Indexes and interleaving](/spanner/docs/secondary-indexes#indexes_and_interleaving) .

``` text
CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;

CREATE INDEX AccountOwnedByPerson
ON PersonOwnAccount (account_id)
STORING (create_time),
INTERLEAVE IN Account;
```

### Use secondary indexes to filter properties

A secondary index enables efficient lookup of nodes and edges based on specific property values. Using an index helps avoid a full-table scan and is especially useful for large graphs.

#### Speed up filtering nodes by property

The following query that finds accounts for a specified nickname. Because it doesn't use a secondary index, all `  Account  ` nodes must be scanned to find the matching results:

``` text
GRAPH FinGraph
MATCH (acct:Account)
WHERE acct.nick_name = "abcd"
RETURN acct.id;
```

Create a secondary index on the filtered property in your schema to speed up the filtering process:

``` text
CREATE TABLE Account (
  id               INT64 NOT NULL,
  create_time      TIMESTAMP,
  is_blocked       BOOL,
  nick_name        STRING(MAX),
) PRIMARY KEY (id);

CREATE INDEX AccountByNickName
ON Account (nick_name);
```

**Tip:** Use NULL-filtered indexes for sparse properties. For more information, see [Disable indexing of `  NULL  ` values](/spanner/docs/secondary-indexes#null-indexing-disable) .

#### Speed up filtering edges by property

You can use a secondary index to improve the performance of filtering edges based on property values.

### Forward edge traversal

Without a secondary index, this query must scan all of a person's edges to find the edges that match the `  create_time  ` filter:

``` text
GRAPH FinGraph
MATCH (person:Person)-[owns:Owns]->(acct:Account)
WHERE person.id = 1
  AND owns.create_time >= PARSE_TIMESTAMP("%c", "Thu Dec 25 07:30:00 2008")
RETURN acct.id;
```

The following code improves query efficiency by creating a secondary index on the edge source node reference ( `  id  ` ) and the edge property ( `  create_time  ` ). The query also defines the index as an interleaved child of the source node input table, which colocates the index with the source node.

``` text
CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;

CREATE INDEX PersonOwnAccountByCreateTime
ON PersonOwnAccount (id, create_time),
INTERLEAVE IN Person;
```

### Reverse edge traversal

Without a secondary index, the following reverse edge traversal query must read all the edges before it can find the person that owns the specified account after the specified `  create_time  ` :

``` text
GRAPH FinGraph
MATCH (acct:Account)<-[owns:Owns]-(person:Person)
WHERE acct.id = 1
  AND owns.create_time >= PARSE_TIMESTAMP("%c", "Thu Dec 25 07:30:00 2008")
RETURN person.id;
```

The following code improves query efficiency by creating a secondary index on the edge destination node reference ( `  account_id  ` ) and the edge property ( `  create_time  ` ). The query also defines the index as the interleaved child of the destination node table, which colocates the index with the destination node.

``` text
CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE;

CREATE INDEX AccountOwnedByPersonByCreateTime
ON PersonOwnAccount (account_id, create_time),
INTERLEAVE IN Account;
```

## Prevent dangling edges

An edge that connects zero or one node, a *dangling edge* , can compromise Spanner Graph query efficiency and graph structure integrity. A dangling edge can occur if you delete a node without deleting its associated edges. A dangling edge can also occur if you create an edge but its source or destination node doesn't exist. To prevent dangling edges, incorporate the following in your Spanner Graph schema:

  - [Use referential constraints](#use-ref-constraints) .
  - Optional: [Use the `  ON DELETE CASCADE  ` clause](#on-delete-cascade) when you delete a node with edges that are still attached. If you don't use `  ON DELETE CASCADE  ` , then attempts to delete a node without deleting corresponding edges fails.

### Use referential constraints

You can use interleaving and enforced foreign keys on both endpoints to prevent dangling edges by following these steps:

1.  Interleave the edge input table into the source node input table to ensure that the source node of an edge always exists.

2.  Create an enforced foreign key constraint on edges to ensure that the destination node of an edge always exists. While enforced foreign keys prevent dangling edges, they make inserting and deleting edges more expensive.

**Note:** You can't prevent a dangling edge with an informational foreign key.

The following example uses an enforced foreign key and interleaves the edge input table into the source node input table using the `  INTERLEAVE IN PARENT  ` clause. Together, using an enforced foreign key and interleaving can also help [optimize forward edge traversal](#optimize-forward-edge-traversal) .

``` text
  CREATE TABLE PersonOwnAccount (
    id               INT64 NOT NULL,
    account_id       INT64 NOT NULL,
    create_time      TIMESTAMP,
    CONSTRAINT FK_Account FOREIGN KEY (account_id) REFERENCES Account (id) ON DELETE CASCADE,
  ) PRIMARY KEY (id, account_id),
    INTERLEAVE IN PARENT Person ON DELETE CASCADE;
```

### Delete edges with `     ON DELETE CASCADE    `

When you use interleaving or an enforced foreign key to prevent dangling edges, use the `  ON DELETE CASCADE  ` clause in your Spanner Graph schema to delete a node's associated edges in the same transaction that deletes the node. For more information, see [Delete cascading for interleaved tables](/spanner/docs/schema-and-data-model#create-interleaved-tables) and [Foreign key actions](/spanner/docs/foreign-keys/overview#how-to-define-foreign-key-action) .

#### Delete cascade for edges connecting different types of nodes

The following examples show how to use `  ON DELETE CASCADE  ` in your Spanner Graph schema to delete dangling edges when you delete a source or destination node. In both cases, the deleted node's type and the type of the node connected to it by an edge are different.

### Source node

Use interleaving to delete dangling edges when the source node is deleted. The following shows how to use interleaving to delete the outgoing edges when the source node ( `  Person  ` ) is deleted. For more information, see [Create interleaved tables](/spanner/docs/schema-and-data-model#create-interleaved-tables) .

``` text
CREATE TABLE PersonOwnAccount (
  id               INT64 NOT NULL,
  account_id       INT64 NOT NULL,
  create_time      TIMESTAMP,
  CONSTRAINT FK_Account FOREIGN KEY (account_id) REFERENCES Account (id) ON DELETE CASCADE,
) PRIMARY KEY (id, account_id),
  INTERLEAVE IN PARENT Person ON DELETE CASCADE
```

**Note:** We also recommend interleaving for [optimizing forward edge traversal](#optimize-forward-edge-traversal) .

### Destination node

Use a foreign key constraint to delete dangling edges when the destination node is deleted. The following example shows how to use a foreign key with `  ON DELETE CASCADE  ` in an edge table to delete incoming edges when the destination node ( `  Account  ` ) is deleted:

``` text
CONSTRAINT FK_Account FOREIGN KEY(account_id)
  REFERENCES Account(id) ON DELETE CASCADE
```

#### Delete cascade for edges connecting the same type of nodes

When an edge's source and destination nodes are of the same type and the edge is interleaved into the source node, you can define `  ON DELETE CASCADE  ` for either the source or destination node, but not both.

To prevent dangling edges in these scenarios, don't interleave into the source node input table. Instead, create two enforced foreign keys on the source and destination node references.

The following example uses `  AccountTransferAccount  ` as the edge input table. It defines two foreign keys, one on each end node of the transfer edge, both with the `  ON DELETE CASCADE  ` action.

``` text
CREATE TABLE AccountTransferAccount (
  id               INT64 NOT NULL,
  to_id            INT64 NOT NULL,
  amount           FLOAT64,
  create_time      TIMESTAMP NOT NULL,
  order_number     STRING(MAX),
  CONSTRAINT FK_FromAccount FOREIGN KEY (id) REFERENCES Account (id) ON DELETE CASCADE,
  CONSTRAINT FK_ToAccount FOREIGN KEY (to_id) REFERENCES Account (id) ON DELETE CASCADE,
) PRIMARY KEY (id, to_id);
```

## Configure time to live (TTL) on nodes and edges

[TTL](/spanner/docs/ttl) lets you expire and remove data after a specified period. You can use TTL in your schema to maintain database size and performance by removing data that has a limited lifespan or relevance. For example, you can configure it to remove session information, temporary caches, or event logs.

The following example uses TTL to delete accounts 90 days after their closure:

``` text
  CREATE TABLE Account (
    id               INT64 NOT NULL,
    create_time      TIMESTAMP,
    close_time       TIMESTAMP,
  ) PRIMARY KEY (id),
    ROW DELETION POLICY (OLDER_THAN(close_time, INTERVAL 90 DAY));
```

When you define a TTL policy on a node table, you must configure how related edges are handled to prevent unintended dangling edges:

  - **For interleaved edge tables:** If an edge table is interleaved in the node table, you can define the interleave relationship with [`  ON DELETE CASCADE  `](#on-delete-cascade) . This ensures that when TTL deletes a node, its associated interleaved edges are also deleted.

  - **For edge tables with foreign keys:** If an edge table references the node table with a foreign key, you have two options:
    
      - To automatically delete edges when the referenced node is deleted by TTL, use `  ON DELETE CASCADE  ` on the foreign key. This maintains referential integrity.
      - To allow edges to remain after the referenced node is deleted (creating a dangling edge), define the foreign key as an [informational foreign key](/spanner/docs/foreign-keys/overview#informational-foreign-keys) .

In the following example, the `  AccountTransferAccount  ` edge table is subject to two data deletion policies:

  - A TTL policy deletes transfer records that are more than ten years old.
  - The `  ON DELETE CASCADE  ` clause deletes all transfer records associated with a source when that account is deleted.

<!-- end list -->

``` text
CREATE TABLE AccountTransferAccount (
  id               INT64 NOT NULL,
  to_id            INT64 NOT NULL,
  amount           FLOAT64,
  create_time      TIMESTAMP NOT NULL,
  order_number     STRING(MAX),
) PRIMARY KEY (id, to_id),
  INTERLEAVE IN PARENT Account ON DELETE CASCADE,
  ROW DELETION POLICY (OLDER_THAN(create_time, INTERVAL 3650 DAY));
```

## Merge node and edge input tables

You can define a node and its incoming or outgoing edges in a single table if your table's columns define a relationship to another table. This approach offers the following benefits:

  - Fewer tables: Reduces the number of tables in your schema, which simplifies data management.

  - Improved query performance: Eliminates traversal that uses joins to a separate edge table.

To learn more, see [Define nodes and edges within a single table](/spanner/docs/graph/schema-overview#define-nodes-and-edges-single-table) .

## What's next

  - [Create, update, or drop a Spanner Graph schema](/spanner/docs/graph/create-update-drop-schema) .
  - [Insert, update, or delete Spanner Graph data](/spanner/docs/graph/insert-update-delete-data) .
