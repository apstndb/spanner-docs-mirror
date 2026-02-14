This document provides a quick reference (also known as a *cheat sheet* ) for SQL syntax used in Spanner documentation code examples.

For a more comprehensive reference of SQL query syntax in Spanner, see [Query syntax in GoogleSQL](/spanner/docs/reference/standard-sql/query-syntax) .

## SQL syntax notation rules

The following table lists and describes the syntax notation rules that Spanner documentation commonly uses.

Notation

Example

Description

Square brackets

`  [ ]  `

Indicates that the enclosed clause or argument is optional. Don't include these brackets in your SQL query.

Parentheses

`  ( )  `

Literal parentheses. Include the parentheses in your SQL query.

Vertical bar

`  |  `

Indicates that you must choose only one option from the alternatives separated by vertical bars. Don't include bars in your SQL query.

Curly braces

`  { }  `

Indicates that the enclosed items represent a set of choices. You must choose exactly one of the options, which are separated by vertical bars (for example, in `  { a | b | c }  ` , you must choose a, b, or c). Don't include the curly braces in your SQL query.

Ellipsis

`  ...  `

Indicates that a list or a portion of the syntax has been truncated for brevity. This is distinct from a comma followed by an ellipsis, which indicates a repeating list. Don't include the ellipsis in your query.

Comma

`  ,  `

Indicates a literal comma, typically used to separate items in a list. Include commas in your query.

Comma followed by an ellipsis

`  , ...  `

Indicates that the preceding element can be repeated multiple times, with each instance separated by a comma. Don't include the ellipsis in your query. Include the commas to separate the repeated elements.

Item list

`  item [, ...]  `

Indicates that you must include at least one item. You can optionally include more items, separated by commas. Don't include the brackets or the ellipsis in your query. Include the commas to separate the additional item.

`  [item, ...]  `

Indicates that you can optionally include at least one item. You can also optionally include more items, separated by commas. Don't include the brackets or the ellipsis in your query. Include the commas to separate the additional items.

Single quotes

`  ''  `

Indicates a literal single quote mark. Used to define [string literals](/spanner/docs/reference/standard-sql/lexical#string_and_bytes_literals) in your query. Include the single quote marks in your query.

Double quotes or Backticks

`  ""  ` or `  ``  `

Indicates literal double quote marks or backticks. Used to enclose [quoted identifiers](/spanner/docs/reference/standard-sql/lexical#quoted_identifiers) in your query. Include the double quote marks or backticks in your query.

Angle brackets

`  < >  `

Literal angle brackets. Include the angle brackets in your SQL query.
