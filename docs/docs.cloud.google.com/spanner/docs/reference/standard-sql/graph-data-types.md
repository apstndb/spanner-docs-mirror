Graph Query Language (GQL) supports all GoogleSQL [data types](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/data-types) , including the following GQL-specific data type:

## Graph data types list

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_element_type">Graph element type</a></td>
<td>An element in a property graph. Can be a <code dir="ltr" translate="no">GRAPH_NODE</code> or <code dir="ltr" translate="no">GRAPH_EDGE</code> .<br />
SQL type name: <code dir="ltr" translate="no">GRAPH_ELEMENT</code></td>
</tr>
<tr class="even">
<td><a href="https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-data-types#graph_path_type">Graph path type</a></td>
<td>A path in a property graph.<br />
SQL type name: <code dir="ltr" translate="no">GRAPH_PATH</code></td>
</tr>
</tbody>
</table>

## Graph element type

| Name            | Description                     |
| --------------- | ------------------------------- |
| `GRAPH_ELEMENT` | An element in a property graph. |

A variable with a `GRAPH_ELEMENT` type is produced by a graph query. The generated type has this format:

    GRAPH_ELEMENT<T>

A graph element is either a node or an edge, representing data from a matching node or edge table based on its label. Each graph element holds a set of properties that can be accessed with a case-insensitive name, similar to fields of a struct.

Graph elements with dynamic properties enabled can store properties beyond those defined in the schema. A schema change isn't needed to manage dynamic properties because the property names and values are based on the input column's values. You can access dynamic properties with their names in the same way as defined properties. For information about how to model dynamic properties, see [dynamic properties definition](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-schema-statements#dynamic_properties_definition) .

If a property isn't defined in the schema, accessing it through the [field-access-operator](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/operators#field_access_operator) returns the `JSON` type if the dynamic property exists, or `NULL` if the property doesn't exist.

**Note:** Names uniquely identify all properties in a graph element, case-insensitively. A defined property takes precedence over any dynamic property when their names conflict.

**Example**

In the following example, `n` represents a graph element in the [`FinGraph`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-schema-statements#fin_graph) property graph:

    GRAPH FinGraph
    MATCH (n:Person)
    RETURN n.name

## Graph path type

| Name         | Description                 |
| ------------ | --------------------------- |
| `GRAPH_PATH` | A path in a property graph. |

The graph path data type represents a sequence of nodes interleaved with edges and has this format:

    GRAPH_PATH<NODE_TYPE, EDGE_TYPE>

You can construct a graph path with the [`PATH`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-gql-functions#path) function or when you create a [path variable](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition) in a graph pattern.
