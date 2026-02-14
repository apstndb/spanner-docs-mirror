Spanner supports the following JSON MySQL functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

## Function list

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="#json_quote"><code dir="ltr" translate="no">        mysql.JSON_QUOTE       </code></a></td>
<td>Quotes a string as a JSON string literal.</td>
</tr>
<tr class="even">
<td><a href="#json_unquote"><code dir="ltr" translate="no">        mysql.JSON_UNQUOTE       </code></a></td>
<td>Unquotes a JSON string literal.</td>
</tr>
</tbody>
</table>

## `     mysql.JSON_QUOTE    `

``` text
mysql.JSON_QUOTE(string_expression)
```

**Description**

Quotes a string as a JSON string literal. This function escapes special characters as required by the JSON specification and encloses the result in double quotes.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` value to quote.

**Return data type**

`  STRING  `

**Example**

The following example quotes an input string to make it a valid JSON string literal:

``` text
  SELECT mysql.JSON_QUOTE('test') as json_quoted;

/*
+------------------------------------------------------------------------------+
| json_quoted                                                                  |
+------------------------------------------------------------------------------+
| "test"                                                                       |
+------------------------------------------------------------------------------+
*/
```

## `     mysql.JSON_UNQUOTE    `

``` text
mysql.JSON_UNQUOTE(json_string_expression)
```

**Description**

Unquotes a JSON string literal, returning the original string value. This involves interpreting escape sequences within the input JSON string.

This function supports the following argument:

  - `  json_string_expression  ` : The `  STRING  ` value to unquote. This string should be a valid JSON string literal, meaning it is typically enclosed in double quotes and has internal special characters escaped.

**Return data type**

`  STRING  `

**Limitations**

If the input string is not a valid JSON string literal (for example, it is not enclosed in double quotes or contains invalid escape sequences), this function might return `  NULL  ` or an empty string, depending on the specific input and the underlying `  JSON_VALUE  ` behavior.

**Example**

The following example unquotes a JSON string literal:

``` text
SELECT mysql.JSON_UNQUOTE('\"test\"') as json_unquoted;

/*
+----------------------------------------------------------------------------+
| json_unquoted                                                              |
+----------------------------------------------------------------------------+
| test                                                                       |
+----------------------------------------------------------------------------+
*/
```
