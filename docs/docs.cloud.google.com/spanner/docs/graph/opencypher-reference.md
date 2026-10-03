---
name: documents/docs.cloud.google.com/spanner/docs/graph/opencypher-reference
uri: https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference
title: Spanner Graph reference for openCypher users
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

> **Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](https://docs.cloud.google.com/spanner/docs/editions-overview) .

This document compares openCypher and Spanner Graph in the following ways:

- Terminology
- Data model
- Schema
- Query
- Mutation

This document assumes you're familiar with [openCypher v9](https://opencypher.org/resources/) .

## Before you begin

[Set up and query Spanner Graph using the Google Cloud console](https://docs.cloud.google.com/spanner/docs/graph/set-up) .

## Terminology

| openCypher                                                                                        | Spanner Graph                                                                                                                                                                                                                        |
|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| nodes                                                                                             | nodes                                                                                                                                                                                                                                |
| relationships                                                                                     | edges                                                                                                                                                                                                                                |
| node labels                                                                                       | node labels                                                                                                                                                                                                                          |
| relationship types                                                                                | edge labels                                                                                                                                                                                                                          |
| clauses                                                                                           | Spanner Graph uses the term `statement` for a complete unit of execution, and `clause` for a modifier to statements. For example, `MATCH` is a statement whereas `WHERE` is a clause.                                                |
| relationship uniqueness openCypher doesn't return results with repeating edges in a single match. | `TRAIL` path When uniqueness is desired in Spanner Graph, use [`TRAIL` mode](https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference#relationship-uniqueness-and-trail-mode) to return unique edges in a single match. |

## Standards compliance

Spanner Graph adopts ISO [Graph Query Language](https://www.iso.org/standard/76120.html) (GQL) and [SQL/Property Graph Queries](https://www.iso.org/standard/79473.html) (SQL/PGQ) standards.

## Data model

Both Spanner Graph and openCypher adopt the property graph data model with some differences.

| openCypher                                           | Spanner Graph                                 |
|------------------------------------------------------|-----------------------------------------------|
| Each relationship has exactly one relationship type. | Both nodes and edges have one or more labels. |

## Schema

| openCypher                        | Spanner Graph                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| A graph has no predefined schema. | A graph schema must be explicitly defined by using the [`CREATE PROPERTY GRAPH` statement](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph) . Labels are statically defined in the schema. To update labels, you need to update the schema. For more information, see [Create, update, or drop a Spanner Graph schema](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema) . |

## Query

Spanner Graph query capabilities are similar to those of openCypher. The differences between Spanner Graph and openCypher are described in this section.

### Specify the graph

In openCypher, there is one default graph, and queries operate on the default graph. In Spanner Graph, you can define more than one graph and a query must start with the `GRAPH` clause to specify the graph to query. For example:

```
   GRAPH FinGraph
   MATCH (p:Person)
   RETURN p.name;
```

For more information, see the [graph query syntax](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements) .

### Graph pattern matching

Spanner Graph supports graph pattern matching capabilities similar to openCypher. The differences are explained in the following sections.

#### Relationship uniqueness and TRAIL mode

openCypher doesn't return results with repeating edges in a single match; this is called *relationship uniqueness* in openCypher. In Spanner Graph, repeating edges are returned by default. When uniqueness is desired, use `TRAIL` mode to ensure no repeating edge exists in the single match. For detailed semantics of `TRAIL` and other different path modes, see [Path mode](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#path_mode) .

The following example shows how the results of a query change with `TRAIL` mode:

- The openCypher and Spanner Graph `TRAIL` mode queries return empty results because the only possible path is to repeat `t1` twice.
- By default, the Spanner Graph query returns a valid path.

![Example graph](https://docs.cloud.google.com/static/spanner/docs/images/spanner-graph-opencypher-image.png)

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph (TRAIL mode)</th>
<th>Spanner Graph (default mode)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre data-fenced=""><code>MATCH
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt;(dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH TRAIL
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt;(dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt; (dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;</code></pre></td>
</tr>
<tr class="even">
<td><strong>Empty result.</strong></td>
<td><strong>Empty result.</strong></td>
<td><strong>Result:</strong><br />

<table>
<thead>
<tr class="header">
<th>src_id</th>
<th>dst_id</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>16</td>
<td>20</td>
</tr>
</tbody>
</table></td>
</tr>
</tbody>
</table>

#### Return graph elements as query results

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre data-fenced=""><code>MATCH (account:Account)
WHERE account.id = 16
RETURN account;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (account:Account)
WHERE account.id = 16
RETURN TO_JSON(account) AS account;</code></pre></td>
</tr>
</tbody>
</table>

In Spanner Graph, query results don't return graph elements. Use the `TO_JSON` function to return graph elements as JSON.

#### Variable-length pattern matching and pattern quantification

Variable-length pattern matching in openCypher is called *path quantification* in Spanner Graph. Path quantification uses a different syntax, as shown in the following example. For more information, see [Quantified path pattern](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#quantified_paths) .

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre data-fenced=""><code>MATCH (src:Account)-[:Transfers*1..2]-&gt;(dst:Account)
WHERE src.id = 16
RETURN dst.id
ORDER BY dst.id;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (src:Account)-[:Transfers]-&gt;{1,2}(dst:Account)
WHERE src.id = 16
RETURN dst.id
ORDER BY dst.id;</code></pre></td>
</tr>
</tbody>
</table>

#### Variable-length pattern: list of elements

Spanner Graph lets you directly access the variables used in path quantifications. In the following example, `e` in Spanner Graph is the same as `edges(p)` in openCypher.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre data-fenced=""><code>MATCH p=(src:Account)-[:Transfers*1..3]-&gt;(dst:Account)
WHERE src.id = 16
RETURN edges(p);</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (src:Account) -[e:Transfers]-&gt;{1,3} (dst:Account)
WHERE src.id = 16
RETURN TO_JSON(e) AS e;</code></pre></td>
</tr>
</tbody>
</table>

#### Shortest path

openCypher has two built-in functions to find the shortest path between nodes: `shortestPath` and `allShortestPath` .

- `shortestPath` finds a single shortest path between nodes.
- `allShortestPath` finds all the shortest paths between nodes. There can be multiple paths of the same length.

Spanner Graph uses a different syntax to find a single shortest path between nodes: `ANY SHORTEST` for `shortestPath.` The `allShortestPath` function isn't supported in Spanner Graph.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre data-fenced=""><code>MATCH
  (src:Account {id: 7}),
  (dst:Account {id: 20}),
  p = shortestPath((src)-[*1..10]-&gt;(dst))
RETURN length(p) AS path_length;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH ANY SHORTEST
  (src:Account {id: 7})-[e:Transfers]-&gt;{1, 3}
  (dst:Account {id: 20})
RETURN ARRAY_LENGTH(e) AS path_length;</code></pre></td>
</tr>
</tbody>
</table>

### Statements and clauses

The following table lists the openCypher clauses, and indicates whether or not they're supported in Spanner Graph.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>MATCH</code></td>
<td>Supported. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference#graph-pattern-matching">graph pattern matching</a> .</td>
</tr>
<tr class="even">
<td><code>OPTIONAL MATCH</code></td>
<td>Supported. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference#graph-pattern-matching">graph pattern matching</a> .</td>
</tr>
<tr class="odd">
<td><code>RETURN / WITH</code></td>
<td>Supported. For more information, see the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_return"><code>RETURN</code> statement</a> and the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_with"><code>WITH</code> statement</a> .<br />
Spanner Graph requires explicit aliasing for complicated expressions.</td>
</tr>
<tr class="even">
<td><br />
Supported.</td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (p:Person)
RETURN EXTRACT(YEAR FROM p.birthday) AS birthYear;</code></pre></td>
</tr>
<tr class="odd">
<td><br />
Not supported.</td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (p:Person)
RETURN EXTRACT(YEAR FROM p.birthday); -- No aliasing</code></pre></td>
</tr>
<tr class="even">
<td><code>WHERE</code></td>
<td>Supported. For more information, see the definition for <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition">graph pattern</a> .</td>
</tr>
<tr class="odd">
<td><code>ORDER BY</code></td>
<td>Supported. For more information, see the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_order_by"><code>ORDER BY</code> statement</a> .</td>
</tr>
<tr class="even">
<td><code>SKIP / LIMIT</code></td>
<td>Supported. For more information, see the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_skip"><code>SKIP</code> statement</a> and the <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_limit"><code>LIMIT</code> statement</a> .<br />
<br />
Spanner Graph requires a constant expression for the offset and the limit.</td>
</tr>
<tr class="odd">
<td><br />
Supported.</td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (n:Account)
RETURN n.id
SKIP @offsetParameter
LIMIT 3;</code></pre></td>
</tr>
<tr class="even">
<td><br />
Not supported.</td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (n:Account)
RETURN n.id
LIMIT VALUE {
  MATCH (m:Person)
  RETURN COUNT(*) AS count
} AS count; -- Not a constant expression</code></pre></td>
</tr>
<tr class="odd">
<td><code>UNION</code></td>
<td>Supported. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro#composite_graph_query">Composite graph query</a> .</td>
</tr>
<tr class="even">
<td><code>UNION ALL</code></td>
<td>Supported. For more information, see <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-intro#composite_graph_query">Composite graph query</a> .</td>
</tr>
<tr class="odd">
<td><code>UNWIND</code></td>
<td>Supported by <a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-query-statements#gql_for"><code>FOR</code> statement</a> .</td>
</tr>
<tr class="even">
<td></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
LET arr = [1, 2, 3]
FOR num IN arr
RETURN num;</code></pre></td>
</tr>
<tr class="odd">
<td><code>MANDATORY MATCH</code></td>
<td>Not supported.</td>
</tr>
<tr class="even">
<td><code>CALL[YIELD...]</code></td>
<td>Not supported.</td>
</tr>
<tr class="odd">
<td><code>CREATE</code> , <code>DELETE</code> , <code>SET</code> , <code>REMOVE</code> , <code>MERGE</code></td>
<td>To learn more, see the <a href="https://docs.cloud.google.com/spanner/docs/graph/opencypher-reference#mutation">Mutation</a> section and <a href="https://docs.cloud.google.com/spanner/docs/graph/insert-update-delete-data">Insert, update, or delete data in Spanner Graph</a> .</td>
</tr>
</tbody>
</table>

### Data types

Spanner Graph supports all GoogleSQL data types. For more information, see [Data types in GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types) .

The following sections compare openCypher data types with Spanner Graph data types.

#### Structural type

| openCypher | Spanner Graph                                                                                                 |
|------------|---------------------------------------------------------------------------------------------------------------|
| Node       | [Node](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) |
| Edge       | [Edge](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type) |
| Path       | [Path](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type)    |

#### Property type

| openCypher                                                                                                                                    | Spanner Graph                                                                                                  |
|-----------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `INT`                                                                                                                                         | [`INT64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#integer_types)          |
| `FLOAT`                                                                                                                                       | [`FLOAT64`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#floating_point_types) |
| `STRING`                                                                                                                                      | [`STRING`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#string_type)           |
| `BOOLEAN`                                                                                                                                     | [`BOOL`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#boolean_type)            |
| `LIST` A homogeneous list of simple types. For example, List of `INT` , List of `STRING` . You can't mix `INT` and `STRING` in a single list. | [`ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#array_type)             |

#### Composite type

| openCypher | Spanner Graph                                                                                                                                                                                            |
|------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `LIST`     | [`ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#array_type) or [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#json_type)   |
| `MAP`      | [`STRUCT`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#struct_type) or [`JSON`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types#json_type) |

Spanner Graph doesn't support heterogeneous lists of different types or maps of a dynamic key list and heterogeneous element value types. Use JSON for these use cases.

#### Type Coercion

| openCypher        | Spanner Graph |
|-------------------|---------------|
| `INT` -\> `FLOAT` | Supported.    |

For more information about type conversion rules, see [Conversion rules in GoogleSQL](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conversion_rules) .

### Functions and expressions

Besides graph functions and expressions, Spanner Graph also supports all GoogleSQL built-in functions and expressions.

This section lists openCypher functions and expressions and their equivalents in Spanner Graph.

> **Note:** For a complete list of functions, see [GoogleSQL functions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/functions-all) . For a complete list of operators, see [GoogleSQL operators](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/operators) . For a complete list of conditional expressions, see [GoogleSQL conditional expressions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/conditional_expressions) .

#### Structural type functions and expressions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Type</th>
<th>openCypher<br />
function or expression</th>
<th>Spanner Graph<br />
function or expression<br />
</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><br />
Node and Edge</td>
<td><code>exists(n.prop)</code></td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#property_exists_predicate"><code>PROPERTY_EXISTS(n, prop)</code></a></td>
</tr>
<tr class="even">
<td><code>id</code> (returns integer)</td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="odd">
<td><code>properties</code></td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/json_functions#to_json"><code>TO_JSON</code></a><br />
</td>
<td></td>
</tr>
<tr class="even">
<td><code>keys</code><br />
(property type names, but not property values)</td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#property_names"><code>PROPERTY_NAMES</code></a><br />
</td>
<td></td>
</tr>
<tr class="odd">
<td><code>labels</code></td>
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#labels"><code>LABELS</code></a></td>
<td></td>
</tr>
<tr class="even">
<td>Edge</td>
<td><code>endNode</code></td>
<td>Not supported.</td>
</tr>
<tr class="odd">
<td><code>startNode</code></td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="even">
<td><code>type</code></td>
<td><code>LABELS</code></td>
<td></td>
</tr>
<tr class="odd">
<td>Path</td>
<td><code>length</code></td>
<td>Not supported.</td>
</tr>
<tr class="even">
<td><code>nodes</code></td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="odd">
<td><code>relationships</code></td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="even">
<td>Node and Edge</td>
<td><code>.</code><br />
<br />
property reference</td>
<td><code>.</code></td>
</tr>
<tr class="odd">
<td><code>[]</code><br />
<br />
dynamic property reference<br />

<pre data-fenced=""><code>MATCH (n)
RETURN n[n.name]</code></pre></td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="even">
<td>Pattern As Expression</td>
<td><code>size(pattern)</code></td>
<td>Not supported. Use a subquery as following<br />

<pre data-fenced=""><code>VALUE {
  MATCH pattern
  RETURN COUNT(*) AS count;
}</code></pre></td>
</tr>
</tbody>
</table>

#### Property type functions and expressions

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th>Type</th>
<th>openCypher<br />
function or expression</th>
<th>Spanner Graph<br />
function or expression<br />
</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Scalar</td>
<td><code>coalesce</code></td>
<td><code>COALESCE</code></td>
</tr>
<tr class="even">
<td><code>head</code></td>
<td><code>ARRAY_FIRST</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>last</code></td>
<td><code>ARRAY_LAST</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>size(list)</code></td>
<td><code>ARRAY_LENGTH</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>size(string)</code></td>
<td><code>LENGTH</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>timestamp</code></td>
<td><code>UNIX_MILLIS(CURRENT_TIMESTAMP())</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>toBoolean</code> / <code>toFloat</code> / <code>toInteger</code></td>
<td><code>CAST(expr AS type)</code></td>
<td></td>
</tr>
<tr class="even">
<td>Aggregate</td>
<td><code>avg</code></td>
<td><code>AVG</code></td>
</tr>
<tr class="odd">
<td><code>collect</code></td>
<td><code>ARRAY_AGG</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>count</code> &lt;</td>
<td><code>COUNT</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>max</code></td>
<td><code>MAX</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>min</code></td>
<td><code>MIN</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>percentileCont</code></td>
<td><code>PERCENTILE_CONT</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>percentileDisc</code></td>
<td><code>PERCENTILE_DISC</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>stDev</code></td>
<td><code>STDDEV</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>stDevP</code></td>
<td>Not supported.</td>
<td></td>
</tr>
<tr class="odd">
<td><code>sum</code></td>
<td><code>SUM</code></td>
<td></td>
</tr>
<tr class="even">
<td>List</td>
<td><code>range</code></td>
<td><code>GENERATE_ARRAY</code></td>
</tr>
<tr class="odd">
<td><code>reverse</code></td>
<td><code>ARRAY_REVERSE</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>tail</code></td>
<td>Spanner Graph doesn't support <code>tail</code> .<br />
Use <code>ARRAY_SLICE</code> and <code>ARRAY_LENGTH</code> instead.</td>
<td></td>
</tr>
<tr class="odd">
<td>Mathematical</td>
<td><code>abs</code></td>
<td><code>ABS</code></td>
</tr>
<tr class="even">
<td><code>ceil</code></td>
<td><code>CEIL</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>floor</code></td>
<td><code>FLOOR</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>rand</code></td>
<td><code>RAND</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>round</code></td>
<td><code>ROUND</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>sign</code></td>
<td><code>SIGN</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>e</code></td>
<td><code>EXP(1)</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>exp</code></td>
<td><code>EXP</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>log</code></td>
<td><code>LOG</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>log10</code></td>
<td><code>LOG10</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>sqrt</code></td>
<td><code>SQRT</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>acos</code></td>
<td><code>ACOS</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>asin</code></td>
<td><code>ASIN</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>atan</code></td>
<td><code>ATAN</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>atan2</code></td>
<td><code>ATAN2</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>cos</code></td>
<td><code>COS</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>cot</code></td>
<td><code>COT</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>degrees</code></td>
<td><code>r * 90 / ASIN(1)</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>pi</code></td>
<td><code>ACOS(-1)</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>radians</code></td>
<td><code>d * ASIN(1) / 90</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>sin</code></td>
<td><code>SIN</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>tan</code></td>
<td><code>TAN</code></td>
<td></td>
</tr>
<tr class="odd">
<td>String</td>
<td><code>left</code></td>
<td><code>LEFT</code></td>
</tr>
<tr class="even">
<td><code>ltrim</code></td>
<td><code>LTRIM</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>replace</code></td>
<td><code>REPLACE</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>reverse</code></td>
<td><code>REVERSE</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>right</code></td>
<td><code>RIGHT</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>rtrim</code></td>
<td><code>RTRIM</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>split</code></td>
<td><code>SPLIT</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>substring</code></td>
<td><code>SUBSTR</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>tolower</code></td>
<td><code>LOWER</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>tostring</code></td>
<td><code>CAST(expr AS STRING)</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>toupper</code></td>
<td><code>UPPER</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>trim</code></td>
<td><code>TRIM</code></td>
<td></td>
</tr>
<tr class="odd">
<td>DISTINCT</td>
<td><code>DISTINCT</code></td>
<td><code>DISTINCT</code></td>
</tr>
<tr class="even">
<td>Mathematical</td>
<td><code>+</code></td>
<td><code>+</code></td>
</tr>
<tr class="odd">
<td><code>-</code></td>
<td><code>-</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>*</code></td>
<td><code>*</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>/</code></td>
<td><code>/</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>%</code></td>
<td><code>MOD</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>^</code></td>
<td><code>POW</code></td>
<td></td>
</tr>
<tr class="even">
<td>Comparison</td>
<td><code>=</code></td>
<td><code>=</code></td>
</tr>
<tr class="odd">
<td><code>&lt;&gt;</code></td>
<td><code>&lt;&gt;</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>&lt;</code></td>
<td><code>&lt;</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>&gt;</code></td>
<td><code>&gt;</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>&lt;=</code></td>
<td><code>&lt;=</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>&gt;=</code></td>
<td><code>&gt;=</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>IS [NOT] NULL</code></td>
<td><code>IS [NOT] NULL</code></td>
<td></td>
</tr>
<tr class="odd">
<td>Chain of comparison<br />

<pre data-fenced=""><code>a &lt; b &lt; c</code></pre></td>
<td>Spanner Graph doesn't support a chain of comparison. This is equivalent to comparisons conjuncted with <code>AND</code> .<br />
For example:<br />

<pre data-fenced=""><code>a &lt; b AND b &lt; C</code></pre></td>
<td></td>
</tr>
<tr class="even">
<td>Boolean</td>
<td><code>AND</code></td>
<td><code>AND</code></td>
</tr>
<tr class="odd">
<td><code>OR</code></td>
<td><code>OR</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>XOR</code><br />
</td>
<td>Spanner Graph doesn't support <code>XOR</code> . Write the query with <code>&lt;&gt;</code> .<br />
<br />
For example:<br />

<pre data-fenced=""><code>boolean_1 &lt;&gt; boolean_2</code></pre></td>
<td></td>
</tr>
<tr class="odd">
<td><code>NOT</code></td>
<td><code>NOT</code></td>
<td></td>
</tr>
<tr class="even">
<td>String</td>
<td><code>STARTS WITH</code></td>
<td><code>STARTS_WITH</code></td>
</tr>
<tr class="odd">
<td><code>ENDS WITH</code></td>
<td><code>ENDS_WITH</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>CONTAINS</code></td>
<td><code>REGEXP_CONTAINS</code></td>
<td></td>
</tr>
<tr class="odd">
<td><code>+</code></td>
<td><code>CONCAT</code></td>
<td></td>
</tr>
<tr class="even">
<td>List</td>
<td><code>+</code></td>
<td><code>ARRAY_CONCAT</code></td>
</tr>
<tr class="odd">
<td><code>IN</code></td>
<td><code>ARRAY_INCLUDES</code></td>
<td></td>
</tr>
<tr class="even">
<td><code>[]</code></td>
<td><code>[]</code></td>
<td></td>
</tr>
</tbody>
</table>

#### Other expressions

| openCypher         | Spanner Graph                                                                                                                                                                                                                                                                                   |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Case expression    | Supported.                                                                                                                                                                                                                                                                                      |
| Exists subquery    | Supported.                                                                                                                                                                                                                                                                                      |
| Map projection     | Not supported. `STRUCT` types provide similar functionalities.                                                                                                                                                                                                                                  |
| List comprehension | Not supported. [`GENERATE_ARRAY`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#generate_array) and [`ARRAY_TRANSFORM`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/array_functions#array_transform) cover the majority of use cases. |

### Query parameter

The following queries show the difference between using parameters in openCypher and in Spanner Graph.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Parameter</td>
<td><pre data-fenced=""><code>MATCH (n:Person)
WHERE n.id = $id
RETURN n.name;</code></pre></td>
<td><pre data-fenced=""><code>GRAPH FinGraph
MATCH (n:Person)
WHERE n.id = @id
RETURN n.name;</code></pre></td>
</tr>
</tbody>
</table>

## Mutation

Spanner Graph uses [GoogleSQL DML](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/dml-syntax) to mutate the node and edge input tables. For more information, see [Insert, update, or delete Spanner Graph data](https://docs.cloud.google.com/spanner/docs/graph/insert-update-delete-data) .

### Create node and edge

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Create nodes and edges</td>
<td><pre data-fenced=""><code>CREATE (:Person {id: 100, name: &#39;John&#39;});
CREATE (:Account {id: 1000, is_blocked: FALSE});


MATCH (p:Person {id: 100}),
      (a:Account {id: 1000})
CREATE (p)-[:Owns {create_time: timestamp()}]-&gt;(a);</code></pre></td>
<td><pre data-fenced=""><code>INSERT INTO
Person (id, name)
VALUES (100, &quot;John&quot;);


INSERT INTO
Account (id, is_blocked)
VALUES (1000, FALSE);


INSERT INTO PersonOwnAccount (id, account_id, create_time)
VALUES (100, 1000, CURRENT_TIMESTAMP());</code></pre></td>
</tr>
<tr class="even">
<td>Create nodes and edges with query results<br />
</td>
<td><pre data-fenced=""><code>MATCH (a:Account {id: 1}), (oa:Account)
WHERE oa &lt;&gt; a
CREATE (a)-[:Transfers {amount: 100, create_time: timestamp()}]-&gt;(oa);</code></pre></td>
<td><pre data-fenced=""><code>INSERT INTO AccountTransferAccount(id, to_id, create_time, amount)
SELECT a.id, oa.id, CURRENT_TIMESTAMP(), 100
FROM GRAPH_TABLE(
  FinGraph
  MATCH
    (a:Account {id:1000}),
    (oa:Account)
  WHERE oa &lt;&gt; a
);</code></pre></td>
</tr>
</tbody>
</table>

In Spanner Graph, the labels are statically assigned according to the `CREATE PROPERTY GRAPH` DDL statement.

### Update node and edge

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Update properties</td>
<td><pre data-fenced=""><code>MATCH (p:Person {id: 100})
SET p.country = &#39;United States&#39;;</code></pre></td>
<td><pre data-fenced=""><code>UPDATE Person AS p
SET p.country = &#39;United States&#39;
WHERE p.id = 100;</code></pre></td>
</tr>
</tbody>
</table>

To update Spanner Graph labels, see [Create, update, or drop a Spanner Graph schema](https://docs.cloud.google.com/spanner/docs/graph/create-update-drop-schema) .

### Merge node and edge

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Insert new element or update properties</td>
<td><pre data-fenced=""><code>MERGE (p:Person {id: 100, country: &#39;United States&#39;});</code></pre></td>
<td><pre data-fenced=""><code>INSERT OR UPDATE INTO Person
(id, country)
VALUES (100, &#39;United States&#39;);</code></pre></td>
</tr>
</tbody>
</table>

### Delete node and edge

Deleting edges is the same as deleting the input table.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Delete nodes and edges</td>
<td><pre data-fenced=""><code>MATCH (p:Person {id:100}), (a:Account {id:1000})
DELETE (p)-[:Owns]-&gt;(a);</code></pre></td>
<td><pre data-fenced=""><code>DELETE PersonOwnAccount
WHERE id = 100 AND account_id = 1000;</code></pre></td>
</tr>
</tbody>
</table>

Deleting nodes requires handling potential dangling edges. When `DELETE CASCADE` is specified, `DELETE` removes the associated edges of nodes like `DETACH DELETE` in openCypher. For more information, see Spanner [schema overview](https://docs.cloud.google.com/spanner/docs/schema-and-data-model) .

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Delete nodes and associated edges</td>
<td><pre data-fenced=""><code>DETACH DELETE (:Account {id: 1000});</code></pre></td>
<td><pre data-fenced=""><code>DELETE Account
WHERE id = 1000;</code></pre></td>
</tr>
</tbody>
</table>

### Return mutation results

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th></th>
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Return results after insertion or update</td>
<td><pre data-fenced=""><code>MATCH (p:Person {id: 100})
SET p.country = &#39;United States&#39;
RETURN p.id, p.name;</code></pre></td>
<td><pre data-fenced=""><code>UPDATE Person AS p
SET p.country = &#39;United States&#39;
WHERE p.id = 100
THEN RETURN id, name;</code></pre></td>
</tr>
<tr class="even">
<td>Return results after deletion</td>
<td><pre data-fenced=""><code>DELETE (p:Person {id: 100})
RETURN p.country;</code></pre></td>
<td><pre data-fenced=""><code>DELETE FROM Person
WHERE id = 100
THEN RETURN country;</code></pre></td>
</tr>
</tbody>
</table>

## What's next

- [Spanner Graph overview](https://docs.cloud.google.com/spanner/docs/graph/overview) .
