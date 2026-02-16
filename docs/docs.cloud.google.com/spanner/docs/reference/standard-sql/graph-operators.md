Graph Query Language (GQL) supports all GoogleSQL [operators](/spanner/docs/reference/standard-sql/operators) , including the following GQL-specific operators:

## Graph operators list

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="#graph_concatenation_operator">Graph concatenation operator</a></td>
<td>Combines multiple graph paths into one and preserves the original order of the nodes and edges.</td>
</tr>
<tr class="even">
<td><a href="#graph_logical_operators">Graph logical operators</a></td>
<td>Tests for the truth of a condition in a graph and produces either <code dir="ltr" translate="no">       TRUE      </code> or <code dir="ltr" translate="no">       FALSE      </code> .</td>
</tr>
<tr class="odd">
<td><a href="#graph_predicates">Graph predicates</a></td>
<td>Tests for the truth of a condition for a graph element and produces <code dir="ltr" translate="no">       TRUE      </code> , <code dir="ltr" translate="no">       FALSE      </code> , or <code dir="ltr" translate="no">       NULL      </code> .</td>
</tr>
<tr class="even">
<td><a href="#all_different_predicate"><code dir="ltr" translate="no">        ALL_DIFFERENT       </code> predicate</a></td>
<td>In a graph, checks to see if the elements in a list are mutually distinct.</td>
</tr>
<tr class="odd">
<td><a href="#is_destination_predicate"><code dir="ltr" translate="no">        IS DESTINATION       </code> predicate</a></td>
<td>In a graph, checks to see if a node is or isn't the destination of an edge.</td>
</tr>
<tr class="even">
<td><a href="#is_labeled_predicate"><code dir="ltr" translate="no">        IS LABELED       </code> predicate</a></td>
<td>In a graph, checks to see if a node or edge label satisfies a label expression.</td>
</tr>
<tr class="odd">
<td><a href="#is_source_predicate"><code dir="ltr" translate="no">        IS SOURCE       </code> predicate</a></td>
<td>In a graph, checks to see if a node is or isn't the source of an edge.</td>
</tr>
<tr class="even">
<td><a href="#property_exists_predicate"><code dir="ltr" translate="no">        PROPERTY_EXISTS       </code> predicate</a></td>
<td>In a graph, checks to see if a property exists for an element.</td>
</tr>
<tr class="odd">
<td><a href="#same_predicate"><code dir="ltr" translate="no">        SAME       </code> predicate</a></td>
<td>In a graph, checks if all graph elements in a list bind to the same node or edge.</td>
</tr>
</tbody>
</table>

## Graph concatenation operator

``` text
graph_path || graph_path [ || ... ]
```

**Description**

Combines multiple graph paths into one and preserves the original order of the nodes and edges.

Arguments:

  - `  graph_path  ` : A `  GRAPH_PATH  ` value that represents a graph path to concatenate.

**Details**

This operator produces an error if the last node in the first path isn't the same as the first node in the second path.

``` text
-- This successfully produces the concatenated path called `full_path`.
MATCH
  p=(src:Account)-[t1:Transfers]->(mid:Account),
  q=(mid)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
```

``` text
-- This produces an error because the first node of the path to be concatenated
-- (mid2) isn't equal to the last node of the previous path (mid1).
MATCH
  p=(src:Account)-[t1:Transfers]->(mid1:Account),
  q=(mid2:Account)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
```

The first node in each subsequent path is removed from the concatenated path.

``` text
-- The concatenated path called `full_path` contains these elements:
-- src, t1, mid, t2, dst.
MATCH
  p=(src:Account)-[t1:Transfers]->(mid:Account),
  q=(mid)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
```

If any `  graph_path  ` is `  NULL  ` , produces `  NULL  ` .

**Example**

In the following query, a path called `  p  ` and `  q  ` are concatenated. Notice that `  mid  ` is used at the end of the first path and at the beginning of the second path. Also notice that the duplicate `  mid  ` is removed from the concatenated path called `  full_path  ` :

``` text
GRAPH FinGraph
MATCH
  p=(src:Account)-[t1:Transfers]->(mid:Account),
  q = (mid)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
RETURN
  JSON_QUERY(TO_JSON(full_path)[0], '$.labels') AS element_a,
  JSON_QUERY(TO_JSON(full_path)[1], '$.labels') AS element_b,
  JSON_QUERY(TO_JSON(full_path)[2], '$.labels') AS element_c,
  JSON_QUERY(TO_JSON(full_path)[3], '$.labels') AS element_d,
  JSON_QUERY(TO_JSON(full_path)[4], '$.labels') AS element_e,
  JSON_QUERY(TO_JSON(full_path)[5], '$.labels') AS element_f

/*-------------------------------------------------------------------------------------+
 | element_a   | element_b     | element_c   | element_d     | element_e   | element_f |
 +-------------------------------------------------------------------------------------+
 | ["Account"] | ["Transfers"] | ["Account"] | ["Transfers"] | ["Account"] |           |
 | ...         | ...           | ...         | ...           | ...         | ...       |
 +-------------------------------------------------------------------------------------/*
```

The following query produces an error because the last node for `  p  ` must be the first node for `  q  ` :

``` text
-- Error: `mid1` and `mid2` aren't equal.
GRAPH FinGraph
MATCH
  p=(src:Account)-[t1:Transfers]->(mid1:Account),
  q=(mid2:Account)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
RETURN TO_JSON(full_path) AS results
```

The following query produces an error because the path called `  p  ` is `  NULL  ` :

``` text
-- Error: a graph path is NULL.
GRAPH FinGraph
MATCH
  p=NULL,
  q=(mid:Account)-[t2:Transfers]->(dst:Account)
LET full_path = p || q
RETURN TO_JSON(full_path) AS results
```

## Graph logical operators

GoogleSQL supports the following logical operators in [element pattern label expressions](/spanner/docs/reference/standard-sql/graph-patterns#element_pattern_definition) :

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Syntax</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       NOT      </code></td>
<td><code dir="ltr" translate="no">       !X      </code></td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if <code dir="ltr" translate="no">       X      </code> isn't included, otherwise, returns <code dir="ltr" translate="no">       FALSE      </code> .</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       OR      </code></td>
<td><code dir="ltr" translate="no">       X | Y      </code></td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if either <code dir="ltr" translate="no">       X      </code> or <code dir="ltr" translate="no">       Y      </code> is included, otherwise, returns <code dir="ltr" translate="no">       FALSE      </code> .</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       AND      </code></td>
<td><code dir="ltr" translate="no">       X &amp; Y      </code></td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if both <code dir="ltr" translate="no">       X      </code> and <code dir="ltr" translate="no">       Y      </code> are included, otherwise, returns <code dir="ltr" translate="no">       FALSE      </code> .</td>
</tr>
</tbody>
</table>

## Graph predicates

GoogleSQL supports the following graph-specific predicates in graph expressions. A predicate can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

  - [`  ALL_DIFFERENT  ` predicate](#all_different_predicate)
  - [`  PROPERTY_EXISTS  ` predicate](#property_exists_predicate)
  - [`  IS SOURCE  ` predicate](#is_source_predicate)
  - [`  IS DESTINATION  ` predicate](#is_destination_predicate)
  - [`  IS LABELED  ` predicate](#is_labeled_predicate)
  - [`  SAME  ` predicate](#same_predicate)

## `     ALL_DIFFERENT    ` predicate

``` text
ALL_DIFFERENT(element, element[, ...])
```

**Description**

In a graph, checks to see if the elements in a list are mutually distinct. Returns `  TRUE  ` if the elements are distinct, otherwise `  FALSE  ` .

**Definitions**

  - `  element  ` : The graph pattern variable for a node or edge element.

**Details**

Produces an error if `  element  ` is `  NULL  ` .

**Return type**

`  BOOL  `

**Examples**

``` text
GRAPH FinGraph
MATCH
  (a1:Account)-[t1:Transfers]->(a2:Account)-[t2:Transfers]->
  (a3:Account)-[t3:Transfers]->(a4:Account)
WHERE a1.id < a4.id
RETURN
  ALL_DIFFERENT(t1, t2, t3) AS results

/*---------+
 | results |
 +---------+
 | FALSE   |
 | TRUE    |
 | TRUE    |
 +---------*/
```

## `     IS DESTINATION    ` predicate

``` text
node IS [ NOT ] DESTINATION [ OF ] edge
```

**Description**

In a graph, checks to see if a node is or isn't the destination of an edge. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  node  ` : The graph pattern variable for the node element.
  - `  edge  ` : The graph pattern variable for the edge element.

**Examples**

``` text
GRAPH FinGraph
MATCH (a:Account)-[transfer:Transfers]-(b:Account)
WHERE a IS DESTINATION of transfer
RETURN a.id AS a_id, b.id AS b_id

/*-------------+
 | a_id | b_id |
 +-------------+
 | 16   | 7    |
 | 16   | 7    |
 | 20   | 16   |
 | 7    | 20   |
 | 16   | 20   |
 +-------------*/
```

``` text
GRAPH FinGraph
MATCH (a:Account)-[transfer:Transfers]-(b:Account)
WHERE b IS DESTINATION of transfer
RETURN a.id AS a_id, b.id AS b_id

/*-------------+
 | a_id | b_id |
 +-------------+
 | 7    | 16   |
 | 7    | 16   |
 | 16   | 20   |
 | 20   | 7    |
 | 20   | 16   |
 +-------------*/
```

## `     IS LABELED    ` predicate

``` text
element IS [ NOT ] LABELED label_expression
```

**Description**

In a graph, checks to see if a node or edge label satisfies a label expression. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` if `  element  ` is `  NULL  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a graph node or edge element.
  - `  label_expression  ` : The label expression to verify. For more information, see [Label expression definition](/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition) .

**Examples**

``` text
GRAPH FinGraph
MATCH (a)
WHERE a IS LABELED Account | Person
RETURN a.id AS a_id, LABELS(a) AS labels

/*----------------+
 | a_id | labels  |
 +----------------+
 | 1    | Person  |
 | 2    | Person  |
 | 3    | Person  |
 | 7    | Account |
 | 16   | Account |
 | 20   | Account |
 +----------------*/
```

``` text
GRAPH FinGraph
MATCH (a)-[e]-(b:Account)
WHERE e IS LABELED Transfers | Owns
RETURN a.Id as a_id, Labels(e) AS labels, b.Id as b_id
ORDER BY a_id, b_id

/*------+-----------------------+------+
 | a_id | labels                | b_id |
 +------+-----------------------+------+
 |    1 | [owns]                |    7 |
 |    2 | [owns]                |   20 |
 |    3 | [owns]                |   16 |
 |    7 | [transfers]           |   16 |
 |    7 | [transfers]           |   16 |
 |    7 | [transfers]           |   20 |
 |   16 | [transfers]           |    7 |
 |   16 | [transfers]           |    7 |
 |   16 | [transfers]           |   20 |
 |   16 | [transfers]           |   20 |
 |   20 | [transfers]           |    7 |
 |   20 | [transfers]           |   16 |
 |   20 | [transfers]           |   16 |
 +------+-----------------------+------*/
```

``` text
GRAPH FinGraph
MATCH (a:Account {Id: 7})
OPTIONAL MATCH (a)-[:OWNS]->(b)
RETURN a.Id AS a_id, b.Id AS b_id, b IS LABELED Account AS b_is_account

/*------+-----------------------+
 | a_id | b_id   | b_is_account |
 +------+-----------------------+
 | 7    | NULL   | NULL         |
 +------+-----------------------+*/
```

## `     IS SOURCE    ` predicate

``` text
node IS [ NOT ] SOURCE [ OF ] edge
```

**Description**

In a graph, checks to see if a node is or isn't the source of an edge. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  node  ` : The graph pattern variable for the node element.
  - `  edge  ` : The graph pattern variable for the edge element.

**Examples**

``` text
GRAPH FinGraph
MATCH (a:Account)-[transfer:Transfers]-(b:Account)
WHERE a IS SOURCE of transfer
RETURN a.id AS a_id, b.id AS b_id

/*-------------+
 | a_id | b_id |
 +-------------+
 | 20   | 7    |
 | 7    | 16   |
 | 7    | 16   |
 | 20   | 16   |
 | 16   | 20   |
 +-------------*/
```

``` text
GRAPH FinGraph
MATCH (a:Account)-[transfer:Transfers]-(b:Account)
WHERE b IS SOURCE of transfer
RETURN a.id AS a_id, b.id AS b_id

/*-------------+
 | a_id | b_id |
 +-------------+
 | 7    | 20   |
 | 16   | 7    |
 | 16   | 7    |
 | 16   | 20   |
 | 20   | 16   |
 +-------------*/
```

## `     PROPERTY_EXISTS    ` predicate

``` text
PROPERTY_EXISTS(element, element_property)
```

**Description**

In a graph, checks to see if a property exists for an element. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a node or edge element.
  - `  element_property  ` : The name of the property to look for in `  element  ` . The property name must refer to a property in the graph. If the property doesn't exist in the graph, an error is produced. The property name is resolved in a case-insensitive manner.

**Example**

``` text
GRAPH FinGraph
MATCH (n:Person|Account WHERE PROPERTY_EXISTS(n, name))
RETURN n.name

/*------+
 | name |
 +------+
 | Alex |
 | Dana |
 | Lee  |
 +------*/
```

## `     SAME    ` predicate

``` text
SAME (element, element[, ...])
```

**Description**

In a graph, checks if all graph elements in a list bind to the same node or edge. Returns `  TRUE  ` if the elements bind to the same node or edge, otherwise `  FALSE  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a node or edge element.

**Details**

Produces an error if `  element  ` is `  NULL  ` .

**Example**

The following query checks to see if `  a  ` and `  b  ` aren't the same person.

``` text
GRAPH FinGraph
MATCH (src:Account)<-[transfer:Transfers]-(dest:Account)
WHERE NOT SAME(src, dest)
RETURN src.id AS source_id, dest.id AS destination_id

/*----------------------------+
 | source_id | destination_id |
 +----------------------------+
 | 7         | 20             |
 | 16        | 7              |
 | 16        | 7              |
 | 16        | 20             |
 | 20        | 16             |
 +----------------------------*/
```
