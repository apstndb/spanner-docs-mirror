  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [QueryPlan](#QueryPlan)
      - [JSON representation](#QueryPlan.SCHEMA_REPRESENTATION)
  - [PlanNode](#PlanNode)
      - [JSON representation](#PlanNode.SCHEMA_REPRESENTATION)
  - [Kind](#Kind)
  - [ChildLink](#ChildLink)
      - [JSON representation](#ChildLink.SCHEMA_REPRESENTATION)
  - [ShortRepresentation](#ShortRepresentation)
      - [JSON representation](#ShortRepresentation.SCHEMA_REPRESENTATION)
  - [QueryAdvisorResult](#QueryAdvisorResult)
      - [JSON representation](#QueryAdvisorResult.SCHEMA_REPRESENTATION)
  - [IndexAdvice](#IndexAdvice)
      - [JSON representation](#IndexAdvice.SCHEMA_REPRESENTATION)

Additional statistics about a `  ResultSet  ` or `  PartialResultSet  ` .

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;queryPlan&quot;: {
    object (QueryPlan)
  },
  &quot;queryStats&quot;: {
    object
  },

  // Union field row_count can be only one of the following:
  &quot;rowCountExact&quot;: string,
  &quot;rowCountLowerBound&quot;: string
  // End of list of possible types for union field row_count.
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  queryPlan  `

`  object ( QueryPlan  ` )

`  QueryPlan  ` for the query associated with this result.

`  queryStats  `

`  object ( Struct  ` format)

Aggregated statistics from the execution of the query. Only present when the query is profiled. For example, a query could return the statistics as follows:

``` text
{
  "rows_returned": "3",
  "elapsed_time": "1.22 secs",
  "cpu_time": "1.19 secs"
}
```

Union field `  row_count  ` . The number of rows modified by the DML statement. `  row_count  ` can be only one of the following:

`  rowCountExact  `

`  string ( int64 format)  `

Standard DML returns an exact count of rows that were modified.

`  rowCountLowerBound  `

`  string ( int64 format)  `

Partitioned DML doesn't offer exactly-once semantics, so it returns a lower bound of the rows modified.

## QueryPlan

Contains an ordered list of nodes appearing in the query plan.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;planNodes&quot;: [
    {
      object (PlanNode)
    }
  ],
  &quot;queryAdvice&quot;: {
    object (QueryAdvisorResult)
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  planNodes[]  `

`  object ( PlanNode  ` )

The nodes in the query plan. Plan nodes are returned in pre-order starting with the plan root. Each `  PlanNode  ` 's `  id  ` corresponds to its index in `  planNodes  ` .

`  queryAdvice  `

`  object ( QueryAdvisorResult  ` )

Optional. The advise/recommendations for a query. Currently this field will be serving index recommendations for a query.

## PlanNode

Node information for nodes appearing in a `  QueryPlan.plan_nodes  ` .

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;index&quot;: integer,
  &quot;kind&quot;: enum (Kind),
  &quot;displayName&quot;: string,
  &quot;childLinks&quot;: [
    {
      object (ChildLink)
    }
  ],
  &quot;shortRepresentation&quot;: {
    object (ShortRepresentation)
  },
  &quot;metadata&quot;: {
    object
  },
  &quot;executionStats&quot;: {
    object
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  index  `

`  integer  `

The `  PlanNode  ` 's index in `  node list  ` .

`  kind  `

`  enum ( Kind  ` )

Used to determine the type of node. May be needed for visualizing different kinds of nodes differently. For example, If the node is a `  SCALAR  ` node, it will have a condensed representation which can be used to directly embed a description of the node in its parent.

`  displayName  `

`  string  `

The display name for the node.

`  childLinks[]  `

`  object ( ChildLink  ` )

List of child node `  index  ` es and their relationship to this parent.

`  shortRepresentation  `

`  object ( ShortRepresentation  ` )

Condensed representation for `  SCALAR  ` nodes.

`  metadata  `

`  object ( Struct  ` format)

Attributes relevant to the node contained in a group of key-value pairs. For example, a Parameter Reference node could have the following information in its metadata:

``` text
{
  "parameter_reference": "param1",
  "parameterType": "array"
}
```

`  executionStats  `

`  object ( Struct  ` format)

The execution statistics associated with the node, contained in a group of key-value pairs. Only present if the plan was returned as a result of a profile query. For example, number of executions, number of rows/time per execution etc.

## Kind

The kind of `  PlanNode  ` . Distinguishes between the two different kinds of nodes that can appear in a query plan.

Enums

`  KIND_UNSPECIFIED  `

Not specified.

`  RELATIONAL  `

Denotes a Relational operator node in the expression tree. Relational operators represent iterative processing of rows during query execution. For example, a `  TableScan  ` operation that reads rows from a table.

`  SCALAR  `

Denotes a Scalar node in the expression tree. Scalar nodes represent non-iterable entities in the query plan. For example, constants or arithmetic operators appearing inside predicate expressions or references to column names.

## ChildLink

Metadata associated with a parent-child relationship appearing in a `  PlanNode  ` .

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;childIndex&quot;: integer,
  &quot;type&quot;: string,
  &quot;variable&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  childIndex  `

`  integer  `

The node to which the link points.

`  type  `

`  string  `

The type of the link. For example, in Hash Joins this could be used to distinguish between the build child and the probe child, or in the case of the child being an output variable, to represent the tag associated with the output variable.

`  variable  `

`  string  `

Only present if the child node is `  SCALAR  ` and corresponds to an output variable of the parent node. The field carries the name of the output variable. For example, a `  TableScan  ` operator that reads rows from a table will have child links to the `  SCALAR  ` nodes representing the output variables created for each column that is read by the operator. The corresponding `  variable  ` fields will be set to the variable names assigned to the columns.

## ShortRepresentation

Condensed representation of a node and its subtree. Only present for `  SCALAR  ` `  PlanNode(s)  ` .

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;description&quot;: string,
  &quot;subqueries&quot;: {
    string: integer,
    ...
  }
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  description  `

`  string  `

A string representation of the expression subtree rooted at this node.

`  subqueries  `

`  map (key: string, value: integer)  `

A mapping of (subquery variable name) -\> (subquery node id) for cases where the `  description  ` string of this node references a `  SCALAR  ` subquery contained in the expression subtree rooted at this node. The referenced `  SCALAR  ` subquery may not necessarily be a direct child of this node.

## QueryAdvisorResult

Output of query advisor analysis.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;indexAdvice&quot;: [
    {
      object (IndexAdvice)
    }
  ]
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  indexAdvice[]  `

`  object ( IndexAdvice  ` )

Optional. Index Recommendation for a query. This is an optional field and the recommendation will only be available when the recommendation guarantees significant improvement in query performance.

## IndexAdvice

Recommendation to add new indexes to run queries more efficiently.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<thead>
<tr class="header">
<th>JSON representation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><pre class="text" dir="ltr" data-is-upgraded="" style="border: 0;margin: 0;" translate="no"><code>{
  &quot;ddl&quot;: [
    string
  ],
  &quot;improvementFactor&quot;: number
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  ddl[]  `

`  string  `

Optional. DDL statements to add new indexes that will improve the query.

`  improvementFactor  `

`  number  `

Optional. Estimated latency improvement factor. For example if the query currently takes 500 ms to run and the estimated latency with new indexes is 100 ms this field will be 5.
