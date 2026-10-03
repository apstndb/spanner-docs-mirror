---
name: documents/docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats
uri: https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats
title: ResultSetStats
description: A managed, mission-critical, globally consistent and scalable relational database service.
data_source: docs.cloud.google.com
---

- [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#SCHEMA_REPRESENTATION)
- [QueryPlan](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan.SCHEMA_REPRESENTATION)
- [PlanNode](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode.SCHEMA_REPRESENTATION)
- [Kind](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#Kind)
- [ChildLink](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ChildLink)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ChildLink.SCHEMA_REPRESENTATION)
- [ShortRepresentation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ShortRepresentation)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ShortRepresentation.SCHEMA_REPRESENTATION)
- [QueryAdvisorResult](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryAdvisorResult)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryAdvisorResult.SCHEMA_REPRESENTATION)
- [IndexAdvice](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#IndexAdvice)
  - [JSON representation](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#IndexAdvice.SCHEMA_REPRESENTATION)

Additional statistics about a [`ResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSet) or [`PartialResultSet`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/PartialResultSet) .

**JSON representation**

```
{
  "queryPlan": {
    object (QueryPlan)
  },
  "queryStats": {
    object
  },

  // Union field row_count can be only one of the following:
  "rowCountExact": string,
  "rowCountLowerBound": string
  // End of list of possible types for union field row_count.
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>queryPlan</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan"><code>QueryPlan</code></a><code> )</code></p>
<p><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan"><code>QueryPlan</code></a> for the query associated with this result.</p></td>
</tr>
<tr class="even">
<td><code>queryStats</code></td>
<td><p><code>object ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a><code> format)</code></p>
<p>Aggregated statistics from the execution of the query. Only present when the query is profiled. For example, a query could return the statistics as follows:</p>
<pre data-fenced=""><code>{
  &quot;rows_returned&quot;: &quot;3&quot;,
  &quot;elapsed_time&quot;: &quot;1.22 secs&quot;,
  &quot;cpu_time&quot;: &quot;1.19 secs&quot;
}</code></pre></td>
</tr>
<tr class="odd">
<td>Union field <code>row_count</code> . The number of rows modified by the DML statement. <code>row_count</code> can be only one of the following:</td>
<td></td>
</tr>
<tr class="even">
<td><code>rowCountExact</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>int64</code></a><code> format)</code></p>
<p>Standard DML returns an exact count of rows that were modified.</p></td>
</tr>
<tr class="odd">
<td><code>rowCountLowerBound</code></td>
<td><p><code>string ( </code><a href="https://developers.google.com/discovery/v1/type-format"><code>int64</code></a><code> format)</code></p>
<p>Partitioned DML doesn't offer exactly-once semantics, so it returns a lower bound of the rows modified.</p></td>
</tr>
</tbody>
</table>

## QueryPlan

Contains an ordered list of nodes appearing in the query plan.

**JSON representation**

```
{
  "planNodes": [
    {
      object (PlanNode)
    }
  ],
  "queryAdvice": {
    object (QueryAdvisorResult)
  }
}
```

| Fields        |                                                                                                                                                                                                                                                                                                                                                                            |
|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `planNodes[]` | `object ( `[`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode)` )` The nodes in the query plan. Plan nodes are returned in pre-order starting with the plan root. Each [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode) 's `id` corresponds to its index in `planNodes` . |
| `queryAdvice` | `object ( `[`QueryAdvisorResult`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryAdvisorResult)` )` Optional. The advise/recommendations for a query. Currently this field will be serving index recommendations for a query.                                                                                                            |

## PlanNode

Node information for nodes appearing in a [`QueryPlan.plan_nodes`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan.FIELDS.plan_nodes) .

**JSON representation**

```
{
  "index": integer,
  "kind": enum (Kind),
  "displayName": string,
  "childLinks": [
    {
      object (ChildLink)
    }
  ],
  "shortRepresentation": {
    object (ShortRepresentation)
  },
  "metadata": {
    object
  },
  "executionStats": {
    object
  }
}
```

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Fields</th>
<th></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code>index</code></td>
<td><p><code>integer</code></p>
<p>The <code>PlanNode</code> 's index in <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#QueryPlan.FIELDS.plan_nodes"><code>node list</code></a> .</p></td>
</tr>
<tr class="even">
<td><code>kind</code></td>
<td><p><code>enum ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#Kind"><code>Kind</code></a><code> )</code></p>
<p>Used to determine the type of node. May be needed for visualizing different kinds of nodes differently. For example, If the node is a <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#Kind.ENUM_VALUES.SCALAR"><code>SCALAR</code></a> node, it will have a condensed representation which can be used to directly embed a description of the node in its parent.</p></td>
</tr>
<tr class="odd">
<td><code>displayName</code></td>
<td><p><code>string</code></p>
<p>The display name for the node.</p></td>
</tr>
<tr class="even">
<td><code>childLinks[]</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ChildLink"><code>ChildLink</code></a><code> )</code></p>
<p>List of child node <code>index</code> es and their relationship to this parent.</p></td>
</tr>
<tr class="odd">
<td><code>shortRepresentation</code></td>
<td><p><code>object ( </code><a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#ShortRepresentation"><code>ShortRepresentation</code></a><code> )</code></p>
<p>Condensed representation for <a href="https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#Kind.ENUM_VALUES.SCALAR"><code>SCALAR</code></a> nodes.</p></td>
</tr>
<tr class="even">
<td><code>metadata</code></td>
<td><p><code>object ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a><code> format)</code></p>
<p>Attributes relevant to the node contained in a group of key-value pairs. For example, a Parameter Reference node could have the following information in its metadata:</p>
<pre data-fenced=""><code>{
  &quot;parameter_reference&quot;: &quot;param1&quot;,
  &quot;parameterType&quot;: &quot;array&quot;
}</code></pre></td>
</tr>
<tr class="odd">
<td><code>executionStats</code></td>
<td><p><code>object ( </code><a href="https://protobuf.dev/reference/protobuf/google.protobuf/#struct"><code>Struct</code></a><code> format)</code></p>
<p>The execution statistics associated with the node, contained in a group of key-value pairs. Only present if the plan was returned as a result of a profile query. For example, number of executions, number of rows/time per execution etc.</p></td>
</tr>
</tbody>
</table>

## Kind

The kind of [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode) . Distinguishes between the two different kinds of nodes that can appear in a query plan.

| Enums              |                                                                                                                                                                                                                                    |
|--------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `KIND_UNSPECIFIED` | Not specified.                                                                                                                                                                                                                     |
| `RELATIONAL`       | Denotes a Relational operator node in the expression tree. Relational operators represent iterative processing of rows during query execution. For example, a `TableScan` operation that reads rows from a table.                  |
| `SCALAR`           | Denotes a Scalar node in the expression tree. Scalar nodes represent non-iterable entities in the query plan. For example, constants or arithmetic operators appearing inside predicate expressions or references to column names. |

## ChildLink

Metadata associated with a parent-child relationship appearing in a [`PlanNode`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode) .

**JSON representation**

```
{
  "childIndex": integer,
  "type": string,
  "variable": string
}
```

| Fields       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `childIndex` | `integer` The node to which the link points.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `type`       | `string` The type of the link. For example, in Hash Joins this could be used to distinguish between the build child and the probe child, or in the case of the child being an output variable, to represent the tag associated with the output variable.                                                                                                                                                                                                                                                                                                                    |
| `variable`   | `string` Only present if the child node is [`SCALAR`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#Kind.ENUM_VALUES.SCALAR) and corresponds to an output variable of the parent node. The field carries the name of the output variable. For example, a `TableScan` operator that reads rows from a table will have child links to the `SCALAR` nodes representing the output variables created for each column that is read by the operator. The corresponding `variable` fields will be set to the variable names assigned to the columns. |

## ShortRepresentation

Condensed representation of a node and its subtree. Only present for `SCALAR` [`PlanNode(s)`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#PlanNode) .

**JSON representation**

```
{
  "description": string,
  "subqueries": {
    string: integer,
    ...
  }
}
```

| Fields        |                                                                                                                                                                                                                                                                                                                                     |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `description` | `string` A string representation of the expression subtree rooted at this node.                                                                                                                                                                                                                                                     |
| `subqueries`  | `map (key: string, value: integer)` A mapping of (subquery variable name) -\> (subquery node id) for cases where the `description` string of this node references a `SCALAR` subquery contained in the expression subtree rooted at this node. The referenced `SCALAR` subquery may not necessarily be a direct child of this node. |

## QueryAdvisorResult

Output of query advisor analysis.

**JSON representation**

```
{
  "indexAdvice": [
    {
      object (IndexAdvice)
    }
  ]
}
```

| Fields          |                                                                                                                                                                                                                                                                                                                              |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `indexAdvice[]` | `object ( `[`IndexAdvice`](https://docs.cloud.google.com/spanner/docs/reference/rest/v1/ResultSetStats#IndexAdvice)` )` Optional. Index Recommendation for a query. This is an optional field and the recommendation will only be available when the recommendation guarantees significant improvement in query performance. |

## IndexAdvice

Recommendation to add new indexes to run queries more efficiently.

**JSON representation**

```
{
  "ddl": [
    string
  ],
  "improvementFactor": number
}
```

| Fields              |                                                                                                                                                                                            |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ddl[]`             | `string` Optional. DDL statements to add new indexes that will improve the query.                                                                                                          |
| `improvementFactor` | `number` Optional. Estimated latency improvement factor. For example if the query currently takes 500 ms to run and the estimated latency with new indexes is 100 ms this field will be 5. |
