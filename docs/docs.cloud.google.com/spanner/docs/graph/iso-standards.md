**Note:** This feature is available with the Spanner Enterprise edition and Enterprise Plus edition. For more information, see the [Spanner editions overview](/spanner/docs/editions-overview) .

The page describes how Spanner Graph supports the ISO international standard query language for graph databases.

Spanner Graph is based on two ISO standards:

  - [ISO/IEC 9075-16:2023 - Information technology — Database languages SQL Property Graph Queries (SQL/PGQ)](https://www.iso.org/standard/79473.html) , Edition 1, 2023
  - [ISO/IEC 39075:2024 - Information technology — Database languages — GQL](https://www.iso.org/standard/76120.html) , Edition 1, 2024

The following tables describe the high-level relationship between SQL/PGQ, GQL, and how Spanner Graph supports these standards.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Standard</th>
<th style="text-align: left;"></th>
<th style="text-align: left;">SQL/PGQ</th>
<th style="text-align: left;">GQL</th>
<th style="text-align: left;">Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Query</td>
<td style="text-align: left;">Graph pattern matching capabilities</td>
<td style="text-align: left;">Shares the core Graph Pattern Matching Language (GPML) functionalities with GQL.</td>
<td style="text-align: left;">Shares the core GPML functionalities with SQL/PGQ.</td>
<td style="text-align: left;">Both standards are supported. For more information, see Spanner Graph <a href="/spanner/docs/reference/standard-sql/graph-patterns">GQL patterns</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Query</td>
<td style="text-align: left;">Other query language features (for example, <code dir="ltr" translate="no">       LIMIT      </code> , <code dir="ltr" translate="no">       ORDER      </code> , aggregation)</td>
<td style="text-align: left;">SQL-based.</td>
<td style="text-align: left;">Similar to SQL, but the GQL query features are linearly composable graph query statements.</td>
<td style="text-align: left;">Both standards are supported. For more information, see Spanner Graph <a href="/spanner/docs/reference/standard-sql/graph-query-statements">GQL query statements</a> and <a href="/spanner/docs/reference/standard-sql/query-syntax">Query syntax in GoogleSQL</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Query</td>
<td style="text-align: left;">Graph and table interoperability</td>
<td style="text-align: left;">Supported.</td>
<td style="text-align: left;">Not supported.</td>
<td style="text-align: left;">Both standards are supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-sql-queries#graph_table_operator"><code dir="ltr" translate="no">        GRAPH_TABLE       </code> operator</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Types</td>
<td style="text-align: left;"></td>
<td style="text-align: left;">Data types, functions and expressions in SQL/PGQ and GQL are similar.</td>
<td style="text-align: left;">Data types, functions and expressions in SQL/PGQ and GQL are similar.</td>
<td style="text-align: left;">Supports most data types and expressions in SQL/PGQ and GQL. For more information, see <a href="/spanner/docs/reference/standard-sql/data-types">Data types in GoogleSQL</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">DML</td>
<td style="text-align: left;"></td>
<td style="text-align: left;">SQL/PGQ inherits DML from SQL.</td>
<td style="text-align: left;">Graph-based DML is supported.</td>
<td style="text-align: left;">Supports SQL table-based DML. For more information, see the <a href="/spanner/docs/reference/standard-sql/dml-syntax">GoogleSQL data manipulation language</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Schema</td>
<td style="text-align: left;"></td>
<td style="text-align: left;">Supports using <code dir="ltr" translate="no">       CREATE PROPERTY GRAPH      </code> from tables.</td>
<td style="text-align: left;">Supports using <code dir="ltr" translate="no">       CREATE PROPERTY GRAPH      </code> with open types and closed types.</td>
<td style="text-align: left;">Supports the SQL/PGQ method. For more information, see the <a href="/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph"><code dir="ltr" translate="no">        CREATE PROPERTY GRAPH       </code></a> definition.</td>
</tr>
</tbody>
</table>

## SQL/PGQ support

**Note:** G000 is the feature ID from the ISO SQL/PGQ standard.

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Standard</th>
<th style="text-align: left;">SQL/PGQ feature ID</th>
<th style="text-align: left;">Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Query (Graph and table interoperability)</td>
<td style="text-align: left;">Feature G900: <code dir="ltr" translate="no">       GRAPH_TABLE      </code></td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/graph/queries-overview#query-graphs-and-tables-together"><code dir="ltr" translate="no">        GRAPH_TABLE       </code></a> operator.</td>
</tr>
<tr class="even">
<td style="text-align: left;">Schema</td>
<td style="text-align: left;">Feature G924: Explicit key clause for element tables. This implies a claim of conformance to Feature G920: DDL-based SQL-property graphs.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph"><code dir="ltr" translate="no">        CREATE_PROPERTY_GRAPH       </code> statement</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Schema</td>
<td style="text-align: left;">Feature G925: Explicit label and properties clause for element tables. This implies a claim of conformance to Feature G920: DDL-based SQL-property graphs.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph"><code dir="ltr" translate="no">        CREATE_PROPERTY_GRAPH       </code> statement</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G001: Repeatable-elements match mode.</td>
<td style="text-align: left;">Supported. Repeatable elements match mode is the default semantic. Explicit repeatable elements match mode clause syntax is not supported.</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G008: Graph pattern <code dir="ltr" translate="no">       WHERE      </code> clause. This implies a claim of conformance to Feature G000: Graph pattern.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition">Graph pattern</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G034: Path concatenation.</td>
<td style="text-align: left;">Supported. For more information, see see <a href="/spanner/docs/reference/standard-sql/graph-patterns#graph_pattern_definition">Graph pattern</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G040: Vertex pattern.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-patterns#element_pattern_definition">Element pattern</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G042: Basic full edge patterns.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-patterns#element_pattern_definition">Element pattern</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G070: Label expression: label disjunction.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition">Label expression</a> .</td>
</tr>
<tr class="even">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G073: Label expression: individual label name.</td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/graph-patterns#label_expression_definition">Label expression</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Query (GPML)</td>
<td style="text-align: left;">Feature G090: Property reference.</td>
<td style="text-align: left;">Supported.</td>
</tr>
</tbody>
</table>

## GQL support

**Note:** GG00 is the feature ID from the ISO GQL standard.

<table>
<colgroup>
<col style="width: 33%" />
<col style="width: 33%" />
<col style="width: 33%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Standard</th>
<th style="text-align: left;">GQL feature ID</th>
<th style="text-align: left;">Spanner Graph</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Schema</td>
<td style="text-align: left;">Feature GG02: Graph with a closed graph type. Conformance with at least one of GG20, GG21, GG22, or GG23:<br />

<ul>
<li>Feature GG20: Explicit element type names.</li>
</ul>
<ul>
<li>Feature GG21: Explicit element type key label sets.</li>
</ul>
<ul>
<li>Feature GG22: Element type key label set inference.</li>
</ul>
<ul>
<li>Feature GG22: Element type key label set inference.</li>
</ul>
<ul>
<li>Feature GG23 Optional element type key label sets.</li>
</ul></td>
<td style="text-align: left;">Supported. GQL support can be chosen from GG01: Graph with an open type or GG02.<br />
Spanner Graph doesn't support the exact same <code dir="ltr" translate="no">       CREATE_GRAPH_TYPE      </code> statement as GQL. However, the <a href="/spanner/docs/reference/standard-sql/graph-schema-statements#gql_create_graph"><code dir="ltr" translate="no">        CREATE_PROPERTY_GRAPH       </code> statement</a> supported by Spanner Graph is closely related to GG02 (with similar support for GG20, GG21, GG22, and GG23).</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lexical structure</td>
<td style="text-align: left;">"A claim of conformance to a specific version of TheUnicode® Standard and the synchronous versions of Unicode Technical Standard #10, Unicode Standard Annex #15, and Unicode Standard Annex #31. The claimed version of The Unicode® Standard shall not be less than 13.0.0."</td>
<td style="text-align: left;">Spanner Graph GQL shares the exact <a href="/spanner/docs/reference/standard-sql/lexical">lexical structure</a> with GoogleSQL. For reference to unicode escape values, see <a href="/spanner/docs/reference/standard-sql/lexical#escape_sequences">Escape sequences for string and bytes literals</a> .</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Data types</td>
<td style="text-align: left;">"A claim of conformance to the set of all value types that are supported as the types of property values. At minimum, this set shall include:<br />

<ul>
<li>The character string type specified by <code dir="ltr" translate="no">         STRING        </code> or <code dir="ltr" translate="no">         VARCHAR        </code> .</li>
</ul>
<ul>
<li>The boolean type specified by <code dir="ltr" translate="no">         BOOLEAN        </code> or <code dir="ltr" translate="no">         BOOL        </code> .</li>
</ul>
<ul>
<li>The signed regular integer type specified by <code dir="ltr" translate="no">         SIGNED INTEGER        </code> , <code dir="ltr" translate="no">         INTEGER        </code> , or <code dir="ltr" translate="no">         INT        </code> .</li>
</ul>
<ul>
<li>The approximate numeric type specified by <code dir="ltr" translate="no">         FLOAT        </code> ."</li>
</ul></td>
<td style="text-align: left;">Supported. For more information, see <a href="/spanner/docs/reference/standard-sql/data-types">a full list of data types</a> supported by Spanner Graph GQL.</td>
</tr>
</tbody>
</table>

## Additional features

The features listed in the previous sections are the minimal conformance features of the standards. Spanner Graph supports additional features in the ISO standards. To learn more, see [Spanner Graph schema overview](/spanner/docs/graph/schema-overview) and [GQL overview](/spanner/docs/reference/standard-sql/graph-intro) .
