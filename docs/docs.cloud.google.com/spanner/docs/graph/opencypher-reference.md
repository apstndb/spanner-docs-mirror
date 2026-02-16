**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

This document compares openCypher and Spanner Graph in the following ways:

  - Terminology
  - Data model
  - Schema
  - Query
  - Mutation

This document assumes you're familiar with [openCypher v9](https://opencypher.org/resources/) .

## Before you begin

[Set up and query Spanner Graph using the Google Cloud console](/spanner/docs/graph/set-up) .

## Terminology

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>nodes</td>
<td>nodes</td>
</tr>
<tr class="even">
<td>relationships</td>
<td>edges</td>
</tr>
<tr class="odd">
<td>node labels</td>
<td>node labels</td>
</tr>
<tr class="even">
<td>relationship types</td>
<td>edge labels</td>
</tr>
<tr class="odd">
<td>clauses</td>
<td>Spanner Graph uses the term <code dir="ltr" translate="no">       statement      </code> for a complete unit of execution, and <code dir="ltr" translate="no">       clause      </code> for a modifier to statements.<br />
<br />
For example, <code dir="ltr" translate="no">       MATCH      </code> is a statement whereas <code dir="ltr" translate="no">       WHERE      </code> is a clause.</td>
</tr>
<tr class="even">
<td>relationship uniqueness<br />
<br />
openCypher doesn't return results with repeating edges in a single match.</td>
<td><code dir="ltr" translate="no">       TRAIL      </code> path<br />
<br />
When uniqueness is desired in Spanner Graph, use <a href="/spanner/docs/graph/opencypher-reference#relationship-uniqueness-and-trail-mode"><code dir="ltr" translate="no">        TRAIL       </code> mode</a> to return unique edges in a single match.</td>
</tr>
</tbody>
</table>

## Standards compliance

Spanner Graph adopts ISO [Graph Query Language](https://www.iso.org/standard/76120.html) (GQL) and [SQL/Property Graph Queries](https://www.iso.org/standard/79473.html) (SQL/PGQ) standards.

## Data model

Both Spanner Graph and openCypher adopt the property graph data model with some differences.

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Each relationship has exactly one relationship type.<br />
</td>
<td>Both nodes and edges have one or more labels.</td>
</tr>
</tbody>
</table>

## Schema

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>A graph has no predefined schema.</td>
<td>A graph schema must be explicitly defined by using the <a href="/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph"><code dir="ltr" translate="no">        CREATE PROPERTY GRAPH       </code> statement</a> .<br />
Labels are statically defined in the schema. To update labels, you need to update the schema.<br />
For more information, see <a href="/spanner/docs/graph/create-update-drop-schema">Create, update, or drop a Spanner Graph schema</a> .</td>
</tr>
</tbody>
</table>

## Query

Spanner Graph query capabilities are similar to those of openCypher. The differences between Spanner Graph and openCypher are described in this section.

### Specify the graph

In openCypher, there is one default graph, and queries operate on the default graph. In Spanner Graph, you can define more than one graph and a query must start with the `  GRAPH  ` clause to specify the graph to query. For example:

``` text
   GRAPH FinGraph
   MATCH (p:Person)
   RETURN p.name;
```

For more information, see the [graph query syntax](/spanner/docs/reference/standard-sql/graph-query-statements) .

### Graph pattern matching

Spanner Graph supports graph pattern matching capabilities similar to openCypher. The differences are explained in the following sections.

#### Relationship uniqueness and TRAIL mode

openCypher doesn't return results with repeating edges in a single match; this is called *relationship uniqueness* in openCypher. In Spanner Graph, repeating edges are returned by default. When uniqueness is desired, use `  TRAIL  ` mode to ensure no repeating edge exists in the single match. For detailed semantics of `  TRAIL  ` and other different path modes, see [Path mode](/spanner/docs/reference/standard-sql/graph-patterns#path_mode) .

The following example shows how the results of a query change with `  TRAIL  ` mode:

  - The openCypher and Spanner Graph `  TRAIL  ` mode queries return empty results because the only possible path is to repeat `  t1  ` twice.
  - By default, the Spanner Graph query returns a valid path.

<table style="width:66%;">
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt;(dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;
      </code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH TRAIL
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt;(dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;
      </code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH
  (src:Account)-[t1:Transfers]-&gt;
  (dst:Account)-[t2:Transfers]-&gt;
  (src)-[t1]-&gt; (dst)
WHERE src.id = 16
RETURN src.id AS src_id, dst.id AS dst_id;
      </code></pre></td>
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

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (account:Account)
WHERE account.id = 16
RETURN account;</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH (account:Account)
WHERE account.id = 16
RETURN TO_JSON(account) AS account;</code></pre></td>
</tr>
</tbody>
</table>

In Spanner Graph, query results don't return graph elements. Use the `  TO_JSON  ` function to return graph elements as JSON.

#### Variable-length pattern matching and pattern quantification

Variable-length pattern matching in openCypher is called *path quantification* in Spanner Graph. Path quantification uses a different syntax, as shown in the following example. For more information, see [Quantified path pattern](/spanner/docs/reference/standard-sql/graph-patterns#quantified_paths) .

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (src:Account)-[:Transfers*1..2]-&gt;(dst:Account)
WHERE src.id = 16
RETURN dst.id
ORDER BY dst.id;
     </code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH (src:Account)-[:Transfers]-&gt;{1,2}(dst:Account)
WHERE src.id = 16
RETURN dst.id
ORDER BY dst.id;
      </code></pre></td>
</tr>
</tbody>
</table>

#### Variable-length pattern: list of elements

Spanner Graph lets you directly access the variables used in path quantifications. In the following example, `  e  ` in Spanner Graph is the same as `  edges(p)  ` in openCypher.

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH p=(src:Account)-[:Transfers*1..3]-&gt;(dst:Account)
WHERE src.id = 16
RETURN edges(p);
      </code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH (src:Account) -[e:Transfers]-&gt;{1,3} (dst:Account)
WHERE src.id = 16
RETURN TO_JSON(e) AS e;
     </code></pre></td>
</tr>
</tbody>
</table>

#### Shortest path

openCypher has two built-in functions to find the shortest path between nodes: `  shortestPath  ` and `  allShortestPath  ` .

  - `  shortestPath  ` finds a single shortest path between nodes.
  - `  allShortestPath  ` finds all the shortest paths between nodes. There can be multiple paths of the same length.

Spanner Graph uses a different syntax to find a single shortest path between nodes: `  ANY SHORTEST  ` for `  shortestPath.  ` The `  allShortestPath  ` function isn't supported in Spanner Graph.

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH
  (src:Account {id: 7}),
  (dst:Account {id: 20}),
  p = shortestPath((src)-[*1..10]-&gt;(dst))
RETURN length(p) AS path_length;
      </code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH ANY SHORTEST
  (src:Account {id: 7})-[e:Transfers]-&gt;{1, 3}
  (dst:Account {id: 20})
RETURN ARRAY_LENGTH(e) AS path_length;
      </code></pre></td>
</tr>
</tbody>
</table>

### Statements and clauses

The following table lists the openCypher clauses, and indicates whether or not they're supported in Spanner Graph.

openCypher

Spanner Graph

`  MATCH  `

Supported. For more information, see [graph pattern matching](/spanner/docs/graph/opencypher-reference#graph-pattern-matching) .

`  OPTIONAL MATCH  `

Supported. For more information, see [graph pattern matching](/spanner/docs/graph/opencypher-reference#graph-pattern-matching) .

`  RETURN / WITH  `

Supported. For more information, see the [`  RETURN  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_return) and the [`  WITH  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_with) .  
Spanner Graph requires explicit aliasing for complicated expressions.

  
Supported.

``` text
GRAPH FinGraph
MATCH (p:Person)
RETURN EXTRACT(YEAR FROM p.birthday) AS birthYear;
```

  
Not supported.

``` text
GRAPH FinGraph
MATCH (p:Person)
RETURN EXTRACT(YEAR FROM p.birthday); -- No aliasing
```

`  WHERE  `

Supported. For more information, see the definition for [graph pattern](/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition) .

`  ORDER BY  `

Supported. For more information, see the [`  ORDER BY  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_order_by) .

`  SKIP / LIMIT  `

Supported. For more information, see the [`  SKIP  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_skip) and the [`  LIMIT  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_limit) .  
  
Spanner Graph requires a constant expression for the offset and the limit.

  
Supported.

``` text
GRAPH FinGraph
MATCH (n:Account)
RETURN n.id
SKIP @offsetParameter
LIMIT 3;
```

  
Not supported.

``` text
GRAPH FinGraph
MATCH (n:Account)
RETURN n.id
LIMIT VALUE {
  MATCH (m:Person)
  RETURN COUNT(*) AS count
} AS count; -- Not a constant expression
```

`  UNION  `

Supported. For more information, see [Composite graph query](/spanner/docs/reference/standard-sql/graph-intro#composite_graph_query) .

`  UNION ALL  `

Supported. For more information, see [Composite graph query](/spanner/docs/reference/standard-sql/graph-intro#composite_graph_query) .

`  UNWIND  `

Supported by [`  FOR  ` statement](/spanner/docs/reference/standard-sql/graph-query-statements#gql_for) .

``` text
GRAPH FinGraph
LET arr = [1, 2, 3]
FOR num IN arr
RETURN num;
```

`  MANDATORY MATCH  `

Not supported.

`  CALL[YIELD...]  `

Not supported.

`  CREATE  ` , `  DELETE  ` , `  SET  ` , `  REMOVE  ` , `  MERGE  `

To learn more, see the [Mutation](/spanner/docs/graph/opencypher-reference#mutation) section and [Insert, update, or delete data in Spanner Graph](/spanner/docs/graph/insert-update-delete-data) .

### Data types

Spanner Graph supports all GoogleSQL data types. For more information, see [Data types in GoogleSQL](/spanner/docs/reference/standard-sql/data-types) .

The following sections compare openCypher data types with Spanner Graph data types.

#### Structural type

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Node</td>
<td><a href="/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type">Node</a></td>
</tr>
<tr class="even">
<td>Edge</td>
<td><a href="/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type">Edge</a></td>
</tr>
<tr class="odd">
<td>Path</td>
<td><a href="/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type">Path</a></td>
</tr>
</tbody>
</table>

#### Property type

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INT      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#integer_types"><code dir="ltr" translate="no">        INT64       </code></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       FLOAT      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#floating_point_types"><code dir="ltr" translate="no">        FLOAT64       </code></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       STRING      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#string_type"><code dir="ltr" translate="no">        STRING       </code></a></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       BOOLEAN      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#boolean_type"><code dir="ltr" translate="no">        BOOL       </code></a></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       LIST      </code><br />
A homogeneous list of simple types.<br />
For example, List of <code dir="ltr" translate="no">       INT      </code> , List of <code dir="ltr" translate="no">       STRING      </code> .<br />
You can't mix <code dir="ltr" translate="no">       INT      </code> and <code dir="ltr" translate="no">       STRING      </code> in a single list.</td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#array_type"><code dir="ltr" translate="no">        ARRAY       </code></a></td>
</tr>
</tbody>
</table>

#### Composite type

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       LIST      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#array_type"><code dir="ltr" translate="no">        ARRAY       </code></a> or <a href="/spanner/docs/reference/standard-sql/data-types#json_type"><code dir="ltr" translate="no">        JSON       </code></a><br />
</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       MAP      </code></td>
<td><a href="/spanner/docs/reference/standard-sql/data-types#struct_type"><code dir="ltr" translate="no">        STRUCT       </code></a> or <a href="/spanner/docs/reference/standard-sql/data-types#json_type"><code dir="ltr" translate="no">        JSON       </code></a><br />
</td>
</tr>
</tbody>
</table>

Spanner Graph doesn't support heterogeneous lists of different types or maps of a dynamic key list and heterogeneous element value types. Use JSON for these use cases.

#### Type Coercion

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       INT      </code> -&gt; <code dir="ltr" translate="no">       FLOAT      </code></td>
<td>Supported.</td>
</tr>
</tbody>
</table>

For more information about type conversion rules, see [Conversion rules in GoogleSQL](/spanner/docs/reference/standard-sql/conversion_rules) .

### Functions and expressions

Besides graph functions and expressions, Spanner Graph also supports all GoogleSQL built-in functions and expressions.

This section lists openCypher functions and expressions and their equivalents in Spanner Graph.

**Note:** For a complete list of functions, see [GoogleSQL functions](/spanner/docs/reference/standard-sql/functions-all) . For a complete list of operators, see [GoogleSQL operators](/spanner/docs/reference/standard-sql/operators) . For a complete list of conditional expressions, see [GoogleSQL conditional expressions](/spanner/docs/reference/standard-sql/conditional_expressions) .

#### Structural type functions and expressions

Type

openCypher  
function or expression

Spanner Graph  
function or expression  

  
Node and Edge

`  exists(n.prop)  `

[`  PROPERTY_EXISTS(n, prop)  `](/spanner/docs/reference/standard-sql/graph-operators#property_exists_predicate)

`  id  ` (returns integer)

Not supported.

`  properties  `

[`  TO_JSON  `](/spanner/docs/reference/standard-sql/json_functions#to_json)  

`  keys  `  
(property type names, but not property values)

[`  PROPERTY_NAMES  `](/spanner/docs/reference/standard-sql/graph-gql-functions#property_names)  

`  labels  `

[`  LABELS  `](/spanner/docs/reference/standard-sql/graph-gql-functions#labels)

Edge

`  endNode  `

Not supported.

`  startNode  `

Not supported.

`  type  `

`  LABELS  `

Path

`  length  `

Not supported.

`  nodes  `

Not supported.

`  relationships  `

Not supported.

Node and Edge

`  .  `  
  
property reference

`  .  `

`  []  `  
  
dynamic property reference  

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (n)
RETURN n[n.name]</code></pre></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Not supported.

Pattern As Expression

`  size(pattern)  `

Not supported. Use a subquery as following  

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>VALUE {
  MATCH pattern
  RETURN COUNT(*) AS count;
}</code></pre></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

#### Property type functions and expressions

Type

openCypher  
function or expression

Spanner Graph  
function or expression  

Scalar

`  coalesce  `

`  COALESCE  `

`  head  `

`  ARRAY_FIRST  `

`  last  `

`  ARRAY_LAST  `

`  size(list)  `

`  ARRAY_LENGTH  `

`  size(string)  `

`  LENGTH  `

`  timestamp  `

`  UNIX_MILLIS(CURRENT_TIMESTAMP())  `

`  toBoolean  ` / `  toFloat  ` / `  toInteger  `

`  CAST(expr AS type)  `

Aggregate

`  avg  `

`  AVG  `

`  collect  `

`  ARRAY_AGG  `

`  count  ` \<

`  COUNT  `

`  max  `

`  MAX  `

`  min  `

`  MIN  `

`  percentileCont  `

`  PERCENTILE_CONT  `

`  percentileDisc  `

`  PERCENTILE_DISC  `

`  stDev  `

`  STDDEV  `

`  stDevP  `

Not supported.

`  sum  `

`  SUM  `

List

`  range  `

`  GENERATE_ARRAY  `

`  reverse  `

`  ARRAY_REVERSE  `

`  tail  `

Spanner Graph doesn't support `  tail  ` .  
Use `  ARRAY_SLICE  ` and `  ARRAY_LENGTH  ` instead.

Mathematical

`  abs  `

`  ABS  `

`  ceil  `

`  CEIL  `

`  floor  `

`  FLOOR  `

`  rand  `

`  RAND  `

`  round  `

`  ROUND  `

`  sign  `

`  SIGN  `

`  e  `

`  EXP(1)  `

`  exp  `

`  EXP  `

`  log  `

`  LOG  `

`  log10  `

`  LOG10  `

`  sqrt  `

`  SQRT  `

`  acos  `

`  ACOS  `

`  asin  `

`  ASIN  `

`  atan  `

`  ATAN  `

`  atan2  `

`  ATAN2  `

`  cos  `

`  COS  `

`  cot  `

`  COT  `

`  degrees  `

`  r * 90 / ASIN(1)  `

`  pi  `

`  ACOS(-1)  `

`  radians  `

`  d * ASIN(1) / 90  `

`  sin  `

`  SIN  `

`  tan  `

`  TAN  `

String

`  left  `

`  LEFT  `

`  ltrim  `

`  LTRIM  `

`  replace  `

`  REPLACE  `

`  reverse  `

`  REVERSE  `

`  right  `

`  RIGHT  `

`  rtrim  `

`  RTRIM  `

`  split  `

`  SPLIT  `

`  substring  `

`  SUBSTR  `

`  tolower  `

`  LOWER  `

`  tostring  `

`  CAST(expr AS STRING)  `

`  toupper  `

`  UPPER  `

`  trim  `

`  TRIM  `

DISTINCT

`  DISTINCT  `

`  DISTINCT  `

Mathematical

`  +  `

`  +  `

`  -  `

`  -  `

`  *  `

`  *  `

`  /  `

`  /  `

`  %  `

`  MOD  `

`  ^  `

`  POW  `

Comparison

`  =  `

`  =  `

`  <>  `

`  <>  `

`  <  `

`  <  `

`  >  `

`  >  `

`  <=  `

`  <=  `

`  >=  `

`  >=  `

`  IS [NOT] NULL  `

`  IS [NOT] NULL  `

Chain of comparison  

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>a &lt; b &lt; c</code></pre></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Spanner Graph doesn't support a chain of comparison. This is equivalent to comparisons conjuncted with `  AND  ` .  
For example:  

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>      a &lt; b AND b &lt; C
      </code></pre></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

Boolean

`  AND  `

`  AND  `

`  OR  `

`  OR  `

`  XOR  `  

Spanner Graph doesn't support `  XOR  ` . Write the query with `  <>  ` .  
  
For example:  

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>      boolean_1 &lt;&gt; boolean_2
      </code></pre></th>
</tr>
</thead>
<tbody>
</tbody>
</table>

`  NOT  `

`  NOT  `

String

`  STARTS WITH  `

`  STARTS_WITH  `

`  ENDS WITH  `

`  ENDS_WITH  `

`  CONTAINS  `

`  REGEXP_CONTAINS  `

`  +  `

`  CONCAT  `

List

`  +  `

`  ARRAY_CONCAT  `

`  IN  `

`  ARRAY_INCLUDES  `

`  []  `

`  []  `

#### Other expressions

<table style="width:50%;">
<colgroup>
<col style="width: 50%" />
<col style="width: 0%" />
</colgroup>
<thead>
<tr class="header">
<th>openCypher</th>
<th>Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Case expression</td>
<td>Supported.</td>
</tr>
<tr class="even">
<td>Exists subquery</td>
<td>Supported.</td>
</tr>
<tr class="odd">
<td>Map projection</td>
<td>Not supported.<br />
<code dir="ltr" translate="no">       STRUCT      </code> types provide similar functionalities.</td>
</tr>
<tr class="even">
<td>List comprehension</td>
<td>Not supported.<br />
<a href="/spanner/docs/reference/standard-sql/array_functions#generate_array"><code dir="ltr" translate="no">        GENERATE_ARRAY       </code></a> and <a href="/spanner/docs/reference/standard-sql/array_functions#array_transform"><code dir="ltr" translate="no">        ARRAY_TRANSFORM       </code></a> cover the majority of use cases.</td>
</tr>
</tbody>
</table>

### Query parameter

The following queries show the difference between using parameters in openCypher and in Spanner Graph.

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (n:Person)
WHERE n.id = $id
RETURN n.name;</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>GRAPH FinGraph
MATCH (n:Person)
WHERE n.id = @id
RETURN n.name;</code></pre></td>
</tr>
</tbody>
</table>

## Mutation

Spanner Graph uses [GoogleSQL DML](/spanner/docs/reference/standard-sql/dml-syntax) to mutate the node and edge input tables. For more information, see [Insert, update, or delete Spanner Graph data](/spanner/docs/graph/insert-update-delete-data) .

### Create node and edge

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>CREATE (:Person {id: 100, name: &#39;John&#39;});
CREATE (:Account {id: 1000, is_blocked: FALSE});


MATCH (p:Person {id: 100}),
      (a:Account {id: 1000})
CREATE (p)-[:Owns {create_time: timestamp()}]-&gt;(a);</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>INSERT INTO
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (a:Account {id: 1}), (oa:Account)
WHERE oa &lt;&gt; a
CREATE (a)-[:Transfers {amount: 100, create_time: timestamp()}]-&gt;(oa);</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>INSERT INTO AccountTransferAccount(id, to_id, create_time, amount)
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

In Spanner Graph, the labels are statically assigned according to the `  CREATE PROPERTY GRAPH  ` DDL statement.

### Update node and edge

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (p:Person {id: 100})
SET p.country = &#39;United States&#39;;</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>UPDATE Person AS p
SET p.country = &#39;United States&#39;
WHERE p.id = 100;</code></pre></td>
</tr>
</tbody>
</table>

To update Spanner Graph labels, see [Create, update, or drop a Spanner Graph schema](/spanner/docs/graph/create-update-drop-schema) .

### Merge node and edge

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MERGE (p:Person {id: 100, country: &#39;United States&#39;});</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>INSERT OR UPDATE INTO Person
(id, country)
VALUES (100, &#39;United States&#39;);</code></pre></td>
</tr>
</tbody>
</table>

### Delete node and edge

Deleting edges is the same as deleting the input table.

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (p:Person {id:100}), (a:Account {id:1000})
DELETE (p)-[:Owns]-&gt;(a);</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>DELETE PersonOwnAccount
WHERE id = 100 AND account_id = 1000;</code></pre></td>
</tr>
</tbody>
</table>

Deleting nodes requires handling potential dangling edges. When `  DELETE CASCADE  ` is specified, `  DELETE  ` removes the associated edges of nodes like `  DETACH DELETE  ` in openCypher. For more information, see Spanner [schema overview](/spanner/docs/schema-and-data-model) .

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>DETACH DELETE (:Account {id: 1000});</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>DELETE Account
WHERE id = 1000;</code></pre></td>
</tr>
</tbody>
</table>

### Return mutation results

<table style="width:55%;">
<colgroup>
<col style="width: 10%" />
<col style="width: 45%" />
<col style="width: 0%" />
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
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>MATCH (p:Person {id: 100})
SET p.country = &#39;United States&#39;
RETURN p.id, p.name;</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>UPDATE Person AS p
SET p.country = &#39;United States&#39;
WHERE p.id = 100
THEN RETURN id, name;</code></pre></td>
</tr>
<tr class="even">
<td>Return results after deletion</td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>DELETE (p:Person {id: 100})
RETURN p.country;</code></pre></td>
<td><pre class="text" dir="ltr" data-is-upgraded="" data-syntax="SQL" translate="no"><code>DELETE FROM Person
WHERE id = 100
THEN RETURN country;</code></pre></td>
</tr>
</tbody>
</table>

## What's next

  - [Spanner Graph overview](/spanner/docs/graph/overview) .
