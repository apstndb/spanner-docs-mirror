This page explains how [fine-grained access control](/spanner/docs/fgac-about) works with Spanner sequences for GoogleSQL-dialect databases and PostgreSQL-dialect databases.

For fine-grained access control users, you can grant either one or both of the following privileges to allow access to read sequence information or generate values from the sequence.

  - Grant `  SELECT  ` on the sequence to allow read access to the parameters and current state of the sequence.
    
    ### GoogleSQL
    
    ``` text
    GRANT SELECT ON SEQUENCE SEQUENCE_NAME TO ROLE ROLE_NAME;
    ```
    
    ### PostgreSQL
    
    ``` text
    GRANT SELECT ON SEQUENCE SEQUENCE_NAME TO ROLE_NAME;
    ```

  - Grant `  UPDATE  ` on the sequence to allow calls to the sequence value generator.
    
    ### GoogleSQL
    
    ``` text
    GRANT UPDATE ON SEQUENCE SEQUENCE_NAME TO ROLE ROLE_NAME;
    ```
    
    ### PostgreSQL
    
    ``` text
    GRANT UPDATE ON SEQUENCE SEQUENCE_NAME TO ROLE_NAME;
    ```

## Required privileges for sequence operations

The following table contains details about which privileges you require when performing a specific sequence operations.

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><strong>Operation</strong></td>
<td><strong>Privilege requirements</strong></td>
</tr>
<tr class="even">
<td><p>GoogleSQL:</p>
<p><code dir="ltr" translate="no">        GET_NEXT_SEQUENCE_VALUE()       </code></p>
<p>PostgreSQL:</p>
<p><code dir="ltr" translate="no">        nextval()       </code></p></td>
<td>Requires an <code dir="ltr" translate="no">       UPDATE      </code> or <code dir="ltr" translate="no">       SELECT      </code> privilege on the sequence. Note that if you execute this function through generated columns or default values, you also need to have an <code dir="ltr" translate="no">       INSERT      </code> or <code dir="ltr" translate="no">       UPDATE      </code> privilege on the column. An <code dir="ltr" translate="no">       UPDATE      </code> privilege on a sequence doesn't automatically grant any privilege on the columns where you want to use the sequence.</td>
</tr>
<tr class="odd">
<td><p>GoogleSQL:</p>
<p><code dir="ltr" translate="no">        GET_INTERNAL_SEQUENCE_STATE()       </code></p>
<p>PostgreSQL:</p>
<p><code dir="ltr" translate="no">        spanner.get_internal_sequence_state()       </code></p></td>
<td>Requires the <code dir="ltr" translate="no">       SELECT      </code> privilege on the sequence that you request.</td>
</tr>
<tr class="even">
<td><p>GoogleSQL:</p>
<p><code dir="ltr" translate="no">        INFORMATION_SCHEMA.SEQUENCES       </code><br />
<code dir="ltr" translate="no">        INFORMATION_SCHEMA.SEQUENCE_OPTIONS       </code></p>
<p>PostgreSQL</p>
<p><code dir="ltr" translate="no">        INFORMATION_SCHEMA.SEQUENCES       </code></p></td>
<td>You can have the <code dir="ltr" translate="no">       SELECT      </code> or <code dir="ltr" translate="no">       UPDATE      </code> privilege on the sequence you want to query. You can only see the sequences that you have a privilege to view.</td>
</tr>
</tbody>
</table>

## What's next

  - Learn more about using [sequences](/spanner/docs/primary-key-default-value#bit-reversed-sequence) in Spanner.
  - Learn about `  SEQUENCE  ` for [GoogleSQL](/spanner/docs/reference/standard-sql/data-definition-language#sequence_statements) or [PostgreSQL](/spanner/docs/reference/postgresql/data-definition-language#sequence_statements) .
  - Learn about sequence functions in [GoogleSQL](/spanner/docs/reference/standard-sql/sequence_functions) or [PostgreSQL](/spanner/docs/reference/postgresql/functions-and-operators#sequence) .
  - Learn about sequences in the `  INFORMATION_SCHEMA  ` in [GoogleSQL](/spanner/docs/information-schema#sequences) or [PostgreSQL](/spanner/docs/information-schema-pg#sequences) .
