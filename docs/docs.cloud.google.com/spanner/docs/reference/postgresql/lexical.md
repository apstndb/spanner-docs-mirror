This page describes the lexical structure used to create PostgreSQL statements in Spanner and defines the syntax of the lexical elements, called *tokens* , of this structure.

The content on this page is based on the PostgreSQL documentation, which is available under the [PostgreSQL License](https://www.postgresql.org/about/licence/) . There are differences in behavior between the PostgreSQL capabilities supported in Spanner and their open-source PostgreSQL equivalents.

A PostgreSQL statement comprises a series of tokens. Tokens include *identifiers* , *quoted identifiers* , *keywords* , *literals* , *operators* , and *special characters* . You can separate tokens with whitespace (for example, space, tab, newline) or comments.

## Identifiers and quoted identifiers

Identifiers are names that are associated with columns, tables, and other database objects. They can be unquoted or quoted.

  - The maximum identifier length is 63 characters. For more information about identifier lengths, see [Quotas & limits](https://cloud.google.com/spanner/quotas) . The PostgreSQL parser truncates identifiers that are longer than 63 characters.
  - Unquoted identifiers must begin with a letter or an underscore character. Subsequent characters can be letters, numbers, or underscores.
  - Identifiers that start with an underscore can only be used for aliases. Schema objects such as tables, views, indexes, and columns cannot have a name that starts with an underscore.
  - Quoted identifiers must be enclosed by double quote (") characters.
      - Quoted identifiers can contain any character, such as spaces or symbols.
      - Quoted identifiers cannot be empty.
      - Quoted identifiers support the same escape sequences as [string literals](#literals-constants) .
      - A [keyword](#keywords) must be a quoted identifier if it is a standalone keyword or the first component of a path expression. It may be unquoted as the second or later component of a path expression.

**Examples**

These are valid identifiers (unquoted identifiers are converted to lower case):

``` text
Customers5
"5Customers"
dataField
_dataField1
ADGROUP
"tableName~"
"GROUP"
```

These path expressions contain valid identifiers:

``` text
foo."GROUP"
foo.GROUP
```

These are invalid identifiers:

``` text
5Customers
_dataField!
GROUP
```

`  5Customers  ` begins with a number, not a letter or underscore. `  _dataField!  ` contains the special character "\!" which is not a letter, number, or underscore. `  GROUP  ` is a keyword, and therefore cannot be used as an identifier without being enclosed by double quote characters.

### Fully qualified names

Fully qualified names (FQNs) combine the schema name and the object name to identify a database object, for example, `  sales.customers  ` . When you add an FQN in DDL, it requires quotes around each part of the name:

``` text
"foo"."Group"
```

## Case sensitivity

PostgreSQL is case sensitive. Quoted identifiers are case preserving. Unquoted identifiers are not. Unquoted identifiers are all converted to lower case before being compared.

**Examples**

``` text
CREATE TABLE Foo (a int primary key);  select * from foo;    -- works
CREATE TABLE Foo (a int primary key);  select * from "foo";  -- works
CREATE TABLE "Foo" (a int primary key);  select * from foo;  -- fails
CREATE TABLE Foo (a int primary key);  select * from "Foo";  -- fails
```

## Keywords

In the following example, the tokens `  SELECT  ` , `  UPDATE  ` , and `  VALUES  ` are examples of keywords, that is, words that have a fixed meaning in the SQL language.

``` text
SELECT * FROM MY_TABLE;
UPDATE MY_TABLE SET A = 5;
INSERT INTO MY_TABLE VALUES (3, 'hi there');
```

Keywords and identifiers have the same lexical structure, meaning that one can't know whether a token is an identifier or a keyword without knowing the language.

## Literals (constants)

There are three kinds of *implicitly-typed constants* in PostgreSQL: strings, bit strings, and numbers. Constants can also be specified with explicit types, which can enable more accurate representation and more efficient handling by the system. These alternatives are discussed in the following subsections.

### String constants

A string constant in SQL is an arbitrary sequence of characters bounded by single quotes ('), for example `  'This is a string'  ` . To include a single-quote character within a string constant, write two adjacent single quotes, for example `  'Dianne''s horse'  ` . Note that this is *not* the same as a double-quote character (").

Two string constants that are only separated by whitespace with at least one newline are concatenated and effectively treated as if the string had been written as one constant. For example:

``` text
SELECT 'foo'
'bar';
```

is equivalent to:

`  SELECT 'foobar';  `

but:

`  SELECT 'foo' 'bar';  `

is not valid syntax.

### String constants with C-style escapes

PostgreSQL also accepts *escape string constants* , which are an extension to the SQL standard. An escape string constant is specified by writing the letter E (upper or lower case) just before the opening single quote, for example `  E'foo'  ` . (When continuing an escape string constant across lines, write E only before the first opening quote.) Within an escape string, a backslash character (\\) begins a C-like *backslash escape* sequence, in which the combination of backslash and following character(s) represent a special byte value, as shown in the following table.

<table>
<thead>
<tr class="header">
<th>Backslash Escape Sequence</th>
<th>Interpretation</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code class="literal" dir="ltr" translate="no">         \b        </code></td>
<td>backspace</td>
</tr>
<tr class="even">
<td><code class="literal" dir="ltr" translate="no">         \f        </code></td>
<td>form feed</td>
</tr>
<tr class="odd">
<td><code class="literal" dir="ltr" translate="no">         \n        </code></td>
<td>newline</td>
</tr>
<tr class="even">
<td><code class="literal" dir="ltr" translate="no">         \r        </code></td>
<td>carriage return</td>
</tr>
<tr class="odd">
<td><code class="literal" dir="ltr" translate="no">         \t        </code></td>
<td>tab</td>
</tr>
<tr class="even">
<td><code class="literal" dir="ltr" translate="no">         \                     o          </code> , <code class="literal" dir="ltr" translate="no">         \                     oo          </code> , <code class="literal" dir="ltr" translate="no">         \                     ooo          </code> ( <em><code dir="ltr" translate="no">          o         </code></em> = 0–7)</td>
<td>octal byte value</td>
</tr>
<tr class="odd">
<td><code class="literal" dir="ltr" translate="no">         \x                     h          </code> , <code class="literal" dir="ltr" translate="no">         \x                     hh          </code> ( <em><code dir="ltr" translate="no">          h         </code></em> = 0–9, A–F)</td>
<td>hexadecimal byte value</td>
</tr>
<tr class="even">
<td><code class="literal" dir="ltr" translate="no">         \u                     xxxx          </code> , <code class="literal" dir="ltr" translate="no">         \U                     xxxxxxxx          </code> ( <em><code dir="ltr" translate="no">          x         </code></em> = 0–9, A–F)</td>
<td>16 or 32-bit hexadecimal Unicode character value</td>
</tr>
</tbody>
</table>

Any other character following a backslash is taken literally. Thus, to include a backslash character, write two backslashes ( `  \\  ` ). Also, a single quote can be included in an escape string by writing `  \'  ` , in addition to the normal way of writing `  ''  ` .

**Note:** Backslash escapes are recognized only in escape string constants, not in regular string constants.

Ensure that the byte sequences that you create, especially when using the octal or hexadecimal escapes, compose valid characters in the server character set encoding. A useful alternative is to use Unicode escapes or the alternative Unicode escape syntax; then the server will check that the character conversion is possible.

Note that Spanner supports only UTF-8 for strings.

The character with the code zero cannot be in a string constant.

### String Constants With Unicode Escapes

PostgreSQL also supports another type of escape syntax for strings that allows specifying arbitrary Unicode characters by code point. A Unicode escape string constant starts with U& (upper or lower case letter U followed by ampersand) immediately before the opening quote, without any spaces in between, for example U&'foo'. (Note that this creates an ambiguity with the operator &. Use spaces around the operator to avoid this problem.) Inside the quotes, Unicode characters can be specified in escaped form by writing a backslash followed by the four-digit hexadecimal code point number or alternatively a backslash followed by a plus sign followed by a six-digit hexadecimal code point number. For example, the string 'data' could be written as the following:

`  U&'d\0061t\+000061'  `

The following less trivial example writes the Russian word "slon" (elephant) in Cyrillic letters:

`  U&'\0441\043B\043E\043D'  `

If you want to use an escape character other than backslash, you can specify it using the `  UESCAPE  ` clause after the string, for example:

`  U&'d!0061t!+000061' UESCAPE '!'  `

The escape character can be any single character other than a hexadecimal digit, the plus sign, a single quote, a double quote, or a whitespace character.

To include the escape character in the string literally, write it twice.

### Dollar-Quoted String Constants

Although the standard syntax for specifying string constants is usually convenient, it can be difficult to understand when the string contains many single quotes or backslashes, because each of those must be doubled. To allow more readable queries in such situations, PostgreSQL provides another way, called "dollar quoting", to write string constants. A dollar-quoted string constant consists of a dollar sign ($), an optional "tag" of zero or more characters, another dollar sign, an arbitrary sequence of characters that makes up the string content, a dollar sign, the same tag that began this dollar quote, and a dollar sign. For example, here are two different ways to specify the string "Dianne's horse" using dollar quoting:

``` text
$$Dianne's horse$$
$SomeTag$Dianne's horse$SomeTag$
```

Notice that inside the dollar-quoted string, single quotes can be used without needing to be escaped. Indeed, no characters inside a dollar-quoted string are ever escaped: the string content is always written literally. Backslashes are not special, and neither are dollar signs, unless they are part of a sequence matching the opening tag.

It's possible to nest dollar-quoted string constants by choosing different tags at each nesting level.

The tag, if any, of a dollar-quoted string follows the same rules as an unquoted identifier, except that it cannot contain a dollar sign. Tags are case sensitive, so `  $tag$String content$tag$  ` is correct, but `  $TAG$String content$tag$  ` is not.

A dollar-quoted string that follows a keyword or identifier must be separated from it by whitespace; otherwise the dollar quoting delimiter would be taken as part of the preceding identifier.

Dollar quoting is not part of the SQL standard, but it is often a more convenient way to write complicated string literals than the standard-compliant single quote syntax. It is particularly useful when representing string constants inside other constants.

### Numeric Constants

Numeric constants are accepted in these general forms:

``` text
digits
digits.[digits][e[+-]digits]
[digits].digits[e[+-]digits]
digitse[+-]digits
```

where digits is one or more decimal digits (0 through 9). At least one digit must be before or after the decimal point, if one is used. At least one digit must follow the exponent marker (e), if one is present. There cannot be any spaces or other characters embedded in the constant. Note that any leading plus or minus sign is not actually considered part of the constant; it is an operator applied to the constant.

These are some examples of valid numeric constants:

``` text
42
3.5
4.
.001
5e2
1.925e-3
```

A numeric constant that contains neither a decimal point nor an exponent is initially presumed to be type bigint if its value fits in type bigint (64 bits); otherwise it is taken to be type numeric. Constants that contain decimal points or exponents are always initially presumed to be type numeric.

The initially assigned data type of a numeric constant is just a starting point for the type resolution algorithms. In most cases the constant will be automatically coerced to the most appropriate type depending on context. When necessary, you can force a numeric value to be interpreted as a specific data type by casting it. For example, you can force a bigint value to be treated as type numeric by writing:

``` text
NUMERIC '123'
123::numeric
```

These are actually just special cases of the general casting notations discussed in the next section.

### Constants Of Other Types

A constant of an arbitrary type can be entered using any one of the following notations:

``` text
type 'string'
'string'::type
CAST ( 'string' AS type )
```

The string constant's text is passed to the input conversion routine for the type called `  type  ` . The result is a constant of the indicated type. The explicit type cast can be omitted if there is no ambiguity as to the type the constant must be (for example, when it is assigned directly to a table column), in which case it is automatically coerced.

The string constant can be written using either regular SQL notation or dollar-quoting.

The `  ::  ` and `  CAST()  ` syntaxes can also be used to specify run-time type conversions of arbitrary expressions. To avoid syntactic ambiguity, the `  type 'string'  ` syntax can only be used to specify the type of a literal constant.

Another restriction on the `  type 'string'  ` syntax is that it does not work for array types; use `  ::  ` or `  CAST()  ` to specify the type of an array constant.

The `  CAST()  ` syntax conforms to SQL. The `  type 'string'  ` syntax is a generalization of the standard: SQL specifies this syntax only for a few data types, but PostgreSQL allows it for all types. The syntax with `  ::  ` is historical PostgreSQL usage.

## Special characters

Some characters that are not alphanumeric have a special meaning that is different from being an operator. Details on the usage can be found at the location where the respective syntax element is described. This section only exists to advise the existence and summarize the purposes of these characters.

  - A dollar sign ($) followed by digits is used to represent a positional parameter in the body of a prepared statement. In other contexts the dollar sign can be part of an identifier or a dollar-quoted string constant.

  - Parentheses (()) have their usual meaning to group expressions and enforce precedence. In some cases parentheses are required as part of the fixed syntax of a particular SQL command.

  - Brackets (\[\]) are used to select the elements of an array.

  - Commas (,) are used in some syntactical constructs to separate the elements of a list.

  - The semicolon (;) terminates a SQL command. It cannot appear anywhere within a command, except within a string constant or quoted identifier.

  - The asterisk (\*) is used in some contexts to denote all the fields of a table row or composite value. It also has a special meaning when used as the argument of an aggregate function, namely that the aggregate does not require any explicit parameter.

  - The period (.) is used in numeric constants, and to separate table, and column names.

## Comments

A comment is a sequence of characters beginning with double dashes and extending to the end of the line. For example:

`  -- This is a standard SQL comment  `

Alternatively, C-style block comments can be used:

``` text
/* multiline comment
 *   with nesting: /* nested block comment */
 */
```

where the comment begins with /\* and extends to the matching occurrence of \*/. These block comments nest, as specified in the SQL standard but unlike C, so that one can comment out larger blocks of code that might contain existing block comments.

A comment is removed from the input stream before further syntax analysis and is effectively replaced by whitespace. The exception is a comment that contains hints, which are preserved for interpretation by the query planner.
