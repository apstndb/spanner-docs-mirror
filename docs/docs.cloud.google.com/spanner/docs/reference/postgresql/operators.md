This page describes the operators supported for PostgreSQL-dialect databases in Spanner.

An operator manipulates any number of data inputs, also called operands, and returns a result.

An operator name is a sequence of characters from the following list:

  - ``  - * / < > = ~ ! @ # % ^ & | ` ?  ``

There are a few restrictions on operator names:

  - `  -  ` and `  /*  ` cannot appear anywhere in an operator name, since they will be taken as the start of a comment.

  - A multiple-character operator name cannot end in `  +  ` or `  -  ` , unless the name also contains at least one of these characters:
    
    ``  ~ ! @ # % ^ & | ` ?  ``
    
    For example, `  @-  ` is an allowed operator name, but `  \*-  ` is not. This restriction allows PostgreSQL to parse SQL-compliant queries without requiring spaces between tokens.

## Operator Precedence

The following table shows the precedence and associativity of the operators in PostgreSQL. Most operators have the same precedence and are left-associative. The precedence and associativity of the operators is hard-wired into the parser. Enclose expressions in parentheses to force a specific evaluation order.

**Operator Precedence (highest to lowest)**

<table>
<thead>
<tr class="header">
<th>Operator/Element</th>
<th>Associativity</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">        .       </code></td>
<td>left</td>
<td>table/column name separator</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        ::       </code></td>
<td>left</td>
<td>PostgreSQL -style typecast</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        [       </code> <code dir="ltr" translate="no">        ]       </code></td>
<td>left</td>
<td>array element selection</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        +       </code> <code dir="ltr" translate="no">        -       </code></td>
<td>right</td>
<td>unary plus, unary minus</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        ^       </code></td>
<td>left</td>
<td>exponentiation</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        *       </code> <code dir="ltr" translate="no">        /       </code> <code dir="ltr" translate="no">        %       </code></td>
<td>left</td>
<td>multiplication, division, modulo</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        +       </code> <code dir="ltr" translate="no">        -       </code></td>
<td>left</td>
<td>addition, subtraction</td>
</tr>
<tr class="even">
<td>(any other operator)</td>
<td>left</td>
<td>all other PostgreSQL and user-defined operators</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        BETWEEN       </code> <code dir="ltr" translate="no">        LIKE       </code> <code dir="ltr" translate="no">        IN       </code></td>
<td></td>
<td>range containment, string matching, set membership</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        &lt;       </code> <code dir="ltr" translate="no">        &gt;       </code> <code dir="ltr" translate="no">        =       </code> <code dir="ltr" translate="no">        &lt;=       </code> <code dir="ltr" translate="no">        &gt;=       </code> <code dir="ltr" translate="no">        &lt;&gt;       </code></td>
<td></td>
<td>comparison operators</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        IS       </code> <code dir="ltr" translate="no">        ISNULL       </code> <code dir="ltr" translate="no">        NOTNULL       </code></td>
<td></td>
<td><code class="literal" dir="ltr" translate="no">        IS TRUE       </code> , <code class="literal" dir="ltr" translate="no">        IS FALSE       </code> , <code class="literal" dir="ltr" translate="no">        IS NULL       </code> , <code class="literal" dir="ltr" translate="no">        IS DISTINCT FROM       </code> , and more</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        NOT       </code></td>
<td>right</td>
<td>logical negation</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">        AND       </code></td>
<td>left</td>
<td>logical conjunction</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">        OR       </code></td>
<td>left</td>
<td>logical disjunction</td>
</tr>
</tbody>
</table>

## Array operators

<table>
<thead>
<tr class="header">
<th>Operator</th>
<th>Example/Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       @&gt;      </code></td>
<td><code dir="ltr" translate="no">       array[1, 2, 3] @&gt; array[1, 2, 1] → true      </code></td>
<td>Array contains operator. Returns <code dir="ltr" translate="no">       true      </code> if the first array contains the second, that is, if every element in the second array equals some element in the first array.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       &lt;@      </code></td>
<td><code dir="ltr" translate="no">       array[1, 1, 3] &lt;@ array[1, 2, 3, 4] → true      </code></td>
<td>Array contained operator. Returns <code dir="ltr" translate="no">       true      </code> if the second array contains the first array. That is, if every element in the first array equals some element in the second array.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       &amp;&amp;      </code></td>
<td><code dir="ltr" translate="no">       array[1, 2, 3] &amp;&amp; array[1, 5] → true      </code></td>
<td>Array overlap operator. Returns <code dir="ltr" translate="no">       true      </code> if the elements in the arrays overlap, that is, if they have any element in common.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ||      </code></td>
<td><code dir="ltr" translate="no">       array[1, 2] || array[3, 4] → {1, 2, 3, 4}      </code></td>
<td>Concatenation operator. Concatenates two arrays.</td>
</tr>
</tbody>
</table>

## Date and time operators

<table>
<thead>
<tr class="header">
<th>Operator</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       date - date      </code></td>
<td><code dir="ltr" translate="no">       date '2001-10-01' - date '2001-09-28' → 3      </code></td>
<td>Subtracts dates, returning the number of days that have elapsed.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       date - integer      </code></td>
<td><code dir="ltr" translate="no">       date '2001-10-01' - 7 → 2001-09-24      </code></td>
<td>Subtracts a number of days from a date, returning the new date.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       date + integer      </code></td>
<td><code dir="ltr" translate="no">       date '2001-09-28' + 7 → 2001-10-05      </code></td>
<td>Adds a number of days to a date, returning the new date.</td>
</tr>
</tbody>
</table>

## JSONB operators

<table>
<colgroup>
<col style="width: 20%" />
<col style="width: 50%" />
<col style="width: 30%" />
</colgroup>
<thead>
<tr class="header">
<th>Operator</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">       -&gt;      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a": {"b":"bear"}}'::jsonb                 -&gt;                'a' → {"b": "bear"}       </code></p>
<p><code dir="ltr" translate="no">        '[{"a":"apple"},{"b":"bear"},{"c":"cat"}]'::jsonb                 -&gt;                2 → {"c": "cat"}       </code></p>
<p><code dir="ltr" translate="no">        '{"a": {"b":"bear"}}'::jsonb                 -&gt;                'a'                 -&gt;                'b' → bear       </code></p>
<p><code dir="ltr" translate="no">        '[{"a":"apple"},{"b":"bear"},{"c":"cat"}]'::jsonb                 -&gt;                -1 IS NULL → true       </code></p></td>
<td><p>Takes text or an integer as an argument and returns a <code dir="ltr" translate="no">        jsonb       </code> object.</p>
<p>When the argument is text, a <code dir="ltr" translate="no">        jsonb       </code> object field is extracted with the given key.</p>
<p>When the argument is an integer <em>n</em> , the <em>n</em> th element of a <code dir="ltr" translate="no">        jsonb       </code> array is returned.</p>
<p>The operator can be chained to extract nested values. See the third example provided.</p>
<p>Negative indexes are not supported. If they're used, SQL NULL is returned. See the last example provided.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       -&gt;&gt;      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a": {"b":"bear"}}'::jsonb                 -&gt;&gt;                'a' → {"b": "bear"}       </code></p>
<p><code dir="ltr" translate="no">        '[{"a":"apple"},{"b":"bear"},{"c":"cat"}]'::jsonb                 -&gt;&gt;                2 → {"c": "cat"}       </code></p>
<p><code dir="ltr" translate="no">        '[{"a":"apple"},{"b":"bear"},{"c":"cat"}]'::jsonb                 -&gt;&gt;                -1 IS NULL → true       </code></p></td>
<td><p>Takes text or an integer as an argument and returns text.</p>
<p>When the argument is text, a <code dir="ltr" translate="no">        jsonb       </code> object field is extracted with the given key.</p>
<p>When the argument is an integer <em>n</em> , the <em>n</em> th element of a <code dir="ltr" translate="no">        jsonb       </code> array is returned.</p>
<p>Negative indexes are not supported. If they're used, SQL NULL is returned. See the last example provided.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       @&gt;      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a":1, "b":2}'::jsonb                 @&gt;                '{"b":2}'::jsonb → true       </code></p></td>
<td><p>Tests whether the left JSONB value contains the right JSONB value.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       &lt;@      </code></td>
<td><p><code dir="ltr" translate="no">        '{"b":2}'::jsonb                 &lt;@                '{"a":1, "b":2}'::jsonb → true       </code></p></td>
<td><p>Tests whether the left JSONB value is contained in the right JSONB value.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ?      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a":1, "b":2}'::jsonb ? 'b' → true       </code></p>
<p><code dir="ltr" translate="no">        '["a", "b", "c"]'::jsonb ? 'b' → true       </code></p></td>
<td><p>Tests whether a text string exists as a top-level key or array element within a JSONB value.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       ?|      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a":1, "b":2, "c":3}'::jsonb                 ?|                array['b', 'd'] → true       </code></p></td>
<td><p>Tests whether any of the strings in a text array exist as top-level keys or array elements.</p></td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       ?&amp;      </code></td>
<td><p><code dir="ltr" translate="no">        '["a", "b", "c"]'::jsonb                 ?&amp;                array['a', 'b'] → true       </code></p></td>
<td><p>Tests whether all of the strings in a text array exist as top-level keys or array elements.</p></td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb || jsonb → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        '["a", "b"]'::jsonb || '["a", "d"]'::jsonb → ["a", "b", "a", "d"]       </code></p>
<p><code dir="ltr" translate="no">        '{"a": "b"}'::jsonb || '{"c": "d"}'::jsonb → {"a": "b", "c": "d"}       </code></p>
<p><code dir="ltr" translate="no">        '[1, 2]'::jsonb || '3'::jsonb → [1, 2, 3]       </code></p>
<p><code dir="ltr" translate="no">        '{"a": "b"}'::jsonb || '42'::jsonb → [{"a": "b"}, 42]       </code></p>
<p>To append an array to another array as a single entry, wrap it in an additional array layer: <code dir="ltr" translate="no">        '[1, 2]'::jsonb || jsonb_build_array('[3, 4]'::jsonb) → [1, 2, [3, 4]]       </code></p></td>
<td>Concatenates two <code dir="ltr" translate="no">       jsonb      </code> values. Concatenating two arrays generates an array containing all the elements of each input. Concatenating two objects generates an object containing the union of their keys, taking the second object's value when there are duplicate keys. All other cases are treated by converting a non-array input into a single-element array, and then processing them as two separate arrays. Does not operate recursively; only merges the top-level array or object structure.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb - text → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        '{"a": "b", "c": "d"}'::jsonb - 'a' → {"c": "d"}       </code> <code dir="ltr" translate="no">        '["a", "b", "c", "b"]'::jsonb - 'b' → ["a", "c"]       </code></p></td>
<td>Deletes a key and its value from a <code dir="ltr" translate="no">       jsonb      </code> object, or matching string values from a <code dir="ltr" translate="no">       jsonb      </code> array.</td>
</tr>
<tr class="even">
<td><code dir="ltr" translate="no">       jsonb - integer → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        '["a", "b"]'::jsonb - 1 → ["a"]       </code></p></td>
<td>Deletes the array element with the specified index. Negative integers are counted from the end. This function expects an array value.</td>
</tr>
<tr class="odd">
<td><code dir="ltr" translate="no">       jsonb #- text[] → jsonb      </code></td>
<td><p><code dir="ltr" translate="no">        '["a", {"b":1}]'::jsonb #- '{1,b}' → ["a", {}]       </code></p></td>
<td>Deletes the field or array element at the specified path, where path elements can be either field keys or array indexes.</td>
</tr>
</tbody>
</table>

## Pattern matching operators

<table>
<thead>
<tr class="header">
<th>Operator</th>
<th>Example / Notes</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><code dir="ltr" translate="no">         string              text !~               pattern              text      </code></td>
<td><code dir="ltr" translate="no">       'thomas' !~ 't.*max' → true      </code></td>
<td>Tests whether a string text does not match a regular expression. Case sensitive.</td>
</tr>
</tbody>
</table>
