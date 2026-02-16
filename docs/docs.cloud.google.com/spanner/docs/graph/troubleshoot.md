**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document describes errors you might encounter when you work with Spanner Graph. Examples of errors and recommended fixes are also provided.

If you require further support after reviewing this troubleshooting guide, see [Get support](/spanner/docs/graph/overview#support) .

## Schema errors

Schema results are based on the dataset used in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

### Element keys must have a uniqueness guarantee

#### Error message

``  Neither the primary keys nor any unique index defined on the property graph element source table `Person` provides the uniqueness guarantee for graph element `Person` belonging to the graph `FinGraph`. You want to redefine the element key columns (`name`) based on the source table's primary keys, or create a unique index on the element's key columns.  ``

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person KEY (name)
  );
```

#### Recommended fix

Create a unique index on the element key columns and redefine the element key columns based on the source table primary keys.

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person KEY (id)
  );
```

Alternatively, create a unique index on the element key columns.

``` text
CREATE UNIQUE INDEX PersonNameIndex ON Person(name);
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person KEY (name)
  );
```

### Names for element definitions must be unique

#### Error message

`  Account is defined more than once; use a unique name.  `

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Account,
    Person
  )
  EDGE TABLES (
    Account
      SOURCE KEY(owner_id) REFERENCES Person
      DESTINATION KEY(account_id) REFERENCES Account
  );
```

#### Recommended fix

Use a unique name for the edge definition.

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Account,
    Person
  )
  EDGE TABLES (
    Account AS Owns
      SOURCE KEY(owner_id) REFERENCES Person
      DESTINATION KEY(account_id) REFERENCES Account
  );
```

### Label definition must be consistent for properties

#### Error message

`  The label Entity is defined with different property declarations. There is one instance of this label defined with properties of [id]. Another instance is defined with properties of [name].  `

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person LABEL Entity PROPERTIES (name),
    Account LABEL Entity PROPERTIES (id)
  );
```

#### Recommended fix

You must use the same set of property names under the same label.

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person LABEL Entity PROPERTIES (id, name),
    Account LABEL Entity PROPERTIES (id, name)
  );
```

### Property declaration must be consistent for property type

#### Error message

`  The property declaration of name has type conflicts. There is an existing declaration of type INT64. There is a conflicting one of type STRING.  `

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person PROPERTIES (name),
    Account PROPERTIES (id AS name)
  );
```

#### Recommended fix

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person PROPERTIES (name),
    Account PROPERTIES (CAST(id AS STRING) AS name)
  );
```

### Property definition must not be a subquery

#### Error message

`  Property value expression of count cannot contain a subquery.  `

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person PROPERTIES ((SELECT COUNT(*) FROM Person) AS count)
  );
```

#### Recommended fix

N/A. This condition is disallowed.

### Property definition must be consistent within the same element definition

#### Error message

`  Property location has more than one definition in the element table Person  `

#### Example error

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person
      LABEL Person PROPERTIES (country AS location)
      LABEL Entity PROPERTIES (city AS location)
  );
```

#### Recommended fix

Use the same property definition.

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person
      LABEL Person PROPERTIES (country AS location)
      LABEL Entity PROPERTIES (country AS location)
  );
```

Alternatively, assign different property names.

``` text
CREATE OR REPLACE PROPERTY GRAPH FinGraph
  NODE TABLES (
    Person
      LABEL Person PROPERTIES (country AS location)
      LABEL Entity PROPERTIES (city AS city)
  );
```

## Query errors

Query results are based on the dataset used in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

### Graph elements cannot be returned as query results

#### Error message

`  Returning expressions of type GRAPH_ELEMENT is not allowed  `

#### Example error

``` text
GRAPH FinGraph
MATCH (n:Account)
RETURN n;
```

#### Recommended fix

``` text
GRAPH FinGraph
MATCH (n:Account)
RETURN TO_JSON(n) AS n;
```

### Property specification can't be used with `     WHERE    ` clause

#### Error message

`  WHERE clause cannot be used together with property specification  `

#### Example error

``` text
GRAPH FinGraph
MATCH (n:Account {id: 1} WHERE n.is_blocked)
RETURN n.id;
```

#### Recommended fix

You can use one of the following suggested fixes.

``` text
GRAPH FinGraph
MATCH (n:Account {id: 1})
WHERE n.is_blocked
RETURN n.id;
```

``` text
GRAPH FinGraph
MATCH (n:Account WHERE n.id = 1 AND n.is_blocked )
RETURN n.id;
```

``` text
GRAPH FinGraph
MATCH (n:Account {id: 1, is_blocked: TRUE})
RETURN n.id;
```

#### Reference to variables defined in previous statements is not allowed

##### Error message

`  Name 'account_id', defined in the previous statement, can only be referenced in the outermost WHERE clause of MATCH  `

##### Description

Reference to variables defined in previous statements is not allowed within the `  MATCH  ` pattern. In the graph query, names defined by previous statements can only be used in the outermost `  WHERE  ` clause of `  MATCH  ` .

##### Example error

``` text
GRAPH FinGraph
LET account_id = 1
MATCH (n:Account {id: account_id})
RETURN n.id;
```

##### Recommended fix

``` text
GRAPH FinGraph
LET account_id = 1
MATCH (n:Account)
WHERE n.id = account_id
RETURN n.id;
```

### Redefining a correlated graph variable is not allowed

#### Error message

`  The name account is already defined; redefining graph element variables in a subquery is not allowed. To refer to the same graph element, use a different name and add an explicit filter that checks for equality.  `

#### Description

In the graph query, graph element names cannot be redefined in an inner graph subquery. This scenario might be interpreted as referencing the same graph element as the outer scope or as binding to new graph elements, which shadows the outer scope name. Redefining is disallowed.

#### Example error

``` text
GRAPH FinGraph
MATCH (account:Account)
RETURN account.id AS account_id, VALUE {
  MATCH (account:Account)-[transfer:Transfers]->(:Account)
  RETURN SUM(transfer.amount) AS total_transfer
} AS total_transfer;
```

#### Recommended fix

``` text
GRAPH FinGraph
MATCH (account:Account)
RETURN account.id AS account_id, VALUE {
  MATCH (a:Account)-[transfer:Transfers]->(:Account)
  WHERE a = account
  RETURN SUM(transfer.amount) AS total_transfer
} AS total_transfer;
```

## Query semantics issues

Query results are based on the dataset used in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

### Different `     WHERE    ` and `     FILTER    ` result in different outputs

#### Description

`  FILTER  ` is a statement; `  WHERE  ` is a clause, as part of the `  MATCH  ` , `  OPTIONAL MATCH  ` statements.

In the first example, the `  WHERE  ` clause adds additional constraints to the patterns described in the `  OPTIONAL MATCH  ` statement. This isn't a filter after the matching is finished.

In the second example, the `  FILTER  ` statement is a filter after the matching is finished.

#### Example issue

The following examples have different outputs because `  WHERE  ` and `  FILTER  ` are different.

**Example 1**

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
OPTIONAL MATCH (m:Account)
WHERE FALSE
RETURN n.id AS n_id, m.id AS m_id;
```

<table>
<thead>
<tr class="header">
<th><strong>n_id</strong></th>
<th><strong>m_id</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
<td>null</td>
</tr>
</tbody>
</table>

**Example 2**

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
OPTIONAL MATCH (m:Account)
FILTER FALSE
RETURN n.id AS n_id, m.id AS m_id;
```

Empty results.

### Different variables propagated across statements result in different outputs

#### Description

In the graph query language, a variable declared multiple times refers to the same graph element in all occurrences.

In Example 1, there is no `  Account  ` node whose `  id  ` is both `  7  ` and `  16  ` . As a result, empty results are returned.

In Example 2, the name `  n  ` is not returned from the previous statement (only `  id  ` is returned). So the second `  MATCH  ` finds the `  Account  ` node whose `  id  ` is `  16  ` .

#### Example issue

The following examples have different outputs because different variables are propagated across statements.

**Example 1**

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
RETURN n

NEXT

MATCH (n:Account {id: 16})
RETURN n.id AS n_id;
```

Empty results.

**Example 2**

``` text
GRAPH FinGraph
MATCH (n:Account {id: 7})
RETURN n.id AS id

NEXT

MATCH (n:Account {id: 16})
RETURN n.id AS n_id;
```

<table>
<thead>
<tr class="header">
<th><strong>n_id</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16</td>
</tr>
</tbody>
</table>

### `     ORDER BY    ` is ignored if there is a succeeding statement that is not `     LIMIT    `

#### Description

In the graph query language, the `  ORDER BY  ` statement is ignored unless one of the following is true:

  - `  ORDER BY  ` is the last statement.
  - `  ORDER BY  ` is immediately followed by `  LIMIT  ` .

In Example 1, `  LIMIT  ` doesn't immediately follow `  ORDER BY  ` ; the final `  LIMIT  ` is separated. This means that `  ORDER BY  ` is ignored by the engine.

In Example 2, `  ORDER BY  ` is applicable because `  LIMIT  ` immediately follows `  ORDER BY  ` .

#### Example issue

The following examples have different outputs because the `  ORDER BY  ` statement is ignored when it's used without `  LIMIT  ` in Example 1.

**Example 1**

``` text
GRAPH FinGraph
MATCH (n:Account)
ORDER BY n.id DESC
RETURN n.id
LIMIT 3;
```

<table>
<thead>
<tr class="header">
<th><strong>n_id</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>7</td>
</tr>
</tbody>
</table>

**Example 2**

``` text
GRAPH FinGraph
MATCH (n:Account)
ORDER BY n.id DESC
LIMIT 3
RETURN n.id;
```

<table>
<thead>
<tr class="header">
<th><strong>n_id</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>20</td>
</tr>
</tbody>
</table>

### Different edge patterns result in different outputs

#### Description

In the dataset used in the error example, the `  ANY  ` direction edge pattern matches each `  Transfers  ` edge in the graph twice.

In Example 1, a `  Transfers  ` edge from `  Account(id=x)  ` to `  Account(id=y)  ` can be matched twice, as follows:

  - n= `  Account(id=x)  ` , m= `  Account(id=y)  `
  - n= `  Account(id=y)  ` , m= `  Account(id=x)  `

There is only one match in Example 2, where n= `  Account(id=x)  ` and m= `  Account(id=y)  ` .

As a result, the query in Example 1 returns `  10  ` and the query in Example 2 returns `  5  ` .

#### Example issue

The following examples have different outputs because different edge patterns are used.

**Example 1**

``` text
GRAPH FinGraph
MATCH (n:Account)-[:Transfers]-(m:Account)
RETURN COUNT(*) AS num_transfer_edges;
```

<table>
<thead>
<tr class="header">
<th><strong>num_transfer_edges</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>10</td>
</tr>
</tbody>
</table>

**Example 2**

``` text
GRAPH FinGraph
MATCH (n:Account)-[:Transfers]->(m:Account)
RETURN COUNT(*) AS num_transfer_edges;
```

<table>
<thead>
<tr class="header">
<th><strong>num_transfer_edges</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>5</td>
</tr>
</tbody>
</table>

## Mutation errors

Mutation results are based on the dataset used in [Set up and query Spanner Graph](/spanner/docs/graph/set-up) .

### Missing source node violates foreign key constraint

#### Error message

`  Parent row for row [...] in table AccountTransferAccount is missing. Row cannot be written.  `

#### Description

`  AccountTransferAccount  ` edge table is `  INTERLEAVED INTO PARENT Account node  ` table. To create the `  Transfer  ` edge, its parent `  Account  ` node must already exist.

#### Example error

``` text
INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
VALUES (100, 1, PENDING_COMMIT_TIMESTAMP(), 200);
```

#### Recommended fix

Create the leading `  Account  ` node first, then create the `  Transfer  ` edge.

### Missing destination node violates foreign key constraint

#### Error message

`  Foreign key constraint FK_TransferTo is violated on table AccountTransferAccount. Cannot find referenced values in Account(id)  `

#### Description

The `  AccountTransferAccount  ` table refers to `  Accounttable  ` through a `  ForeignKey  ` called `  FK_TransferTo  ` . To create the `  Transfer  ` edge, the referenced tailing node `  Account  ` node must already exist.

#### Example error

``` text
INSERT INTO AccountTransferAccount (id, to_id, create_time, amount)
VALUES (1, 100, PENDING_COMMIT_TIMESTAMP(), 200);
```

#### Recommended fix

Create the tailing Account node first, then create the `  Transfer  ` edge.

### Orphaned outgoing edge violates parent-child relationship

#### Error message

`  Integrity constraint violation during DELETE/REPLACE. Found child row [...] in table AccountTransferAccount  `

#### Description

`  AccountTransferAccount  ` edge table is `  INTERLEAVED INTO PARENT  ` `  Account  ` node table and the `  Account  ` node to be deleted still has outgoing edges attached to it.

#### Example error

``` text
DELETE FROM Account WHERE id = 1;
```

#### Recommended fix

Delete all outgoing `  Transfer  ` edges first, then delete the `  Account  ` node. Alternatively, define [`  ON DELETE CASCADE  `](/spanner/docs/graph/best-practices-designing-schema#on-delete-cascade) for `  INTERLEAVE  ` and have Spanner automatically delete those edges.

### Orphaned incoming edge violates parent-child relationship

#### Error message

`  Foreign key constraint violation when deleting or updating referenced row(s): referencing row(s) found in table AccountTransferAccount  `

#### Description

`  AccountTransferAccount  ` edge table refers to `  Account  ` node table through a `  ForeignKey  ` , and the `  Account  ` node to be deleted still has incoming edges attached to it.

#### Example error

``` text
DELETE FROM Account WHERE id = 1;
```

#### Recommended fix

Delete all incoming `  Transfer  ` edges first, then delete the `  Account  ` node. Alternatively, define [`  ON DELETE CASCADE  `](/spanner/docs/graph/best-practices-designing-schema#on-delete-cascade) for `  ForeignKey  ` and have Spanner automatically delete those edges.
