Graph Query Language (GQL) supports all GoogleSQL [operators](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/operators) , including the following GQL-specific operators:

## Graph operators list

| Name                                                                                                                                                        | Summary                                                                                                                                         |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| [Graph concatenation operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#graph_concatenation_operator)              | Combines multiple graph paths into one and preserves the original order of the nodes and edges.                                                 |
| [Graph logical operators](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#graph_logical_operators)                        | Tests for the truth of a condition in a graph label and produces either `        TRUE       ` or `        FALSE       ` .                       |
| [Graph predicates](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#graph_predicates)                                      | Tests for the truth of a condition for a graph element and produces `        TRUE       ` , `        FALSE       ` , or `        NULL       ` . |
| [`         ALL_DIFFERENT        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#all_different_predicate)     | In a graph, checks to see if the elements in a list are all different.                                                                          |
| [`         IS DESTINATION        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_destination_predicate)   | In a graph, checks to see if a node is or isn't the destination of an edge.                                                                     |
| [`         IS LABELED        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_labeled_predicate)           | In a graph, checks to see if a node or edge label satisfies a label expression.                                                                 |
| [`         IS SOURCE        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_source_predicate)             | In a graph, checks to see if a node is or isn't the source of an edge.                                                                          |
| [`         PROPERTY_EXISTS        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#property_exists_predicate) | In a graph, checks to see if a property exists for an element.                                                                                  |
| [`         SAME        ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#same_predicate)                       | In a graph, checks if all graph elements in a list bind to the same node or edge.                                                               |

## Graph concatenation operator

    graph_path || graph_path [ || ... ]

**Description**

Combines multiple graph paths into one and preserves the original order of the nodes and edges.

Arguments:

  - `  graph_path  ` : A `  GRAPH_PATH  ` value that represents a graph path to concatenate.

**Details**

This operator produces an error if the last node in the first path isn't the same as the first node in the second path.

    -- This successfully produces the concatenated path called `full_path`.
    MATCH
      p=(src:Account)-[t1:Transfers]->(mid:Account),
      q=(mid)-[t2:Transfers]->(dst:Account)
    LET full_path = p || q

    -- This produces an error because the first node of the path to be concatenated
    -- (mid2) isn't equal to the last node of the previous path (mid1).
    MATCH
      p=(src:Account)-[t1:Transfers]->(mid1:Account),
      q=(mid2:Account)-[t2:Transfers]->(dst:Account)
    LET full_path = p || q

The first node in each subsequent path is removed from the concatenated path.

    -- The concatenated path called `full_path` contains these elements:
    -- src, t1, mid, t2, dst.
    MATCH
      p=(src:Account)-[t1:Transfers]->(mid:Account),
      q=(mid)-[t2:Transfers]->(dst:Account)
    LET full_path = p || q

If any `  graph_path  ` is `  NULL  ` , produces `  NULL  ` .

**Example**

In the following query, a path called `  p  ` and `  q  ` are concatenated. Notice that `  mid  ` is used at the end of the first path and at the beginning of the second path. Also notice that the duplicate `  mid  ` is removed from the concatenated path called `  full_path  ` :

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

The following query produces an error because the last node for `  p  ` must be the first node for `  q  ` :

    -- Error: `mid1` and `mid2` aren't equal.
    GRAPH FinGraph
    MATCH
      p=(src:Account)-[t1:Transfers]->(mid1:Account),
      q=(mid2:Account)-[t2:Transfers]->(dst:Account)
    LET full_path = p || q
    RETURN TO_JSON(full_path) AS results

The following query produces an error because the path called `  p  ` is `  NULL  ` :

    -- Error: a graph path is NULL.
    GRAPH FinGraph
    MATCH
      p=NULL,
      q=(mid:Account)-[t2:Transfers]->(dst:Account)
    LET full_path = p || q
    RETURN TO_JSON(full_path) AS results

## Graph logical operators

GoogleSQL supports the following logical operators in [element pattern label expressions](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#element_pattern_definition) :

| Name                 | Syntax                  | Description                                                                                                                               |
| -------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `        NOT       ` | `        !X       `     | Returns `        TRUE       ` if `        X       ` isn't included, otherwise, returns `        FALSE       ` .                           |
| `        OR       `  | `        X \| Y       ` | Returns `        TRUE       ` if either `        X       ` or `        Y       ` is included, otherwise, returns `        FALSE       ` . |
| `        AND       ` | `        X & Y       `  | Returns `        TRUE       ` if both `        X       ` and `        Y       ` are included, otherwise, returns `        FALSE       ` . |

## Graph predicates

GoogleSQL supports the following graph-specific predicates in graph expressions. A predicate can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

  - [`  ALL_DIFFERENT  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#all_different_predicate)
  - [`  PROPERTY_EXISTS  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#property_exists_predicate)
  - [`  IS SOURCE  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_source_predicate)
  - [`  IS DESTINATION  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_destination_predicate)
  - [`  IS LABELED  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#is_labeled_predicate)
  - [`  SAME  ` predicate](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-operators#same_predicate)

## `     ALL_DIFFERENT    ` predicate

    ALL_DIFFERENT(element, element[, ...])

**Description**

In a graph, checks to see if the elements in a list are all different. Returns `  TRUE  ` if none of the elements in the list equal one another, otherwise `  FALSE  ` .

**Definitions**

  - `  element  ` : The graph pattern variable for a node or edge element.

**Details**

Produces an error if `  element  ` is `  NULL  ` .

**Return type**

`  BOOL  `

**Examples**

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

## `     IS DESTINATION    ` predicate

    node IS [ NOT ] DESTINATION [ OF ] edge

**Description**

In a graph, checks to see if a node is or isn't the destination of an edge. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  node  ` : The graph pattern variable for the node element.
  - `  edge  ` : The graph pattern variable for the edge element.

**Examples**

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

## `     IS LABELED    ` predicate

    element IS [ NOT ] LABELED label_expression

**Description**

In a graph, checks to see if a node or edge label satisfies a label expression. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` if `  element  ` is `  NULL  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a graph node or edge element.
  - `  label_expression  ` : The label expression to verify. For more information, see [Label expression definition](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition) .

**Examples**

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

    GRAPH FinGraph
    MATCH (a:Account {Id: 7})
    OPTIONAL MATCH (a)-[:OWNS]->(b)
    RETURN a.Id AS a_id, b.Id AS b_id, b IS LABELED Account AS b_is_account
    
    /*------+-----------------------+
     | a_id | b_id   | b_is_account |
     +------+-----------------------+
     | 7    | NULL   | NULL         |
     +------+-----------------------+*/

## `     IS SOURCE    ` predicate

    node IS [ NOT ] SOURCE [ OF ] edge

**Description**

In a graph, checks to see if a node is or isn't the source of an edge. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  node  ` : The graph pattern variable for the node element.
  - `  edge  ` : The graph pattern variable for the edge element.

**Examples**

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

## `     PROPERTY_EXISTS    ` predicate

    PROPERTY_EXISTS(element, element_property)

**Description**

In a graph, checks to see if a property exists for an element. Can produce `  TRUE  ` , `  FALSE  ` , or `  NULL  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a node or edge element.
  - `  element_property  ` : The name of the property to look for in `  element  ` . The property name must refer to a property in the graph. If the property doesn't exist in the graph, an error is produced. The property name is resolved in a case-insensitive manner.

**Example**

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

## `     SAME    ` predicate

    SAME (element, element[, ...])

**Description**

In a graph, checks if all graph elements in a list bind to the same node or edge. Returns `  TRUE  ` if the elements bind to the same node or edge, otherwise `  FALSE  ` .

Arguments:

  - `  element  ` : The graph pattern variable for a node or edge element.

**Details**

Produces an error if `  element  ` is `  NULL  ` .

**Example**

The following query returns the source and destination IDs for transfers between different accounts:

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
