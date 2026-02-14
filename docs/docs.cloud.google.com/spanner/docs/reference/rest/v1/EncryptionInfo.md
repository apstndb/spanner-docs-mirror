  - [JSON representation](#SCHEMA_REPRESENTATION)
  - [Type](#Type)

Encryption information for a Cloud Spanner database or backup.

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
  &quot;encryptionType&quot;: enum (Type),
  &quot;encryptionStatus&quot;: {
    object (Status)
  },
  &quot;kmsKeyVersion&quot;: string
}</code></pre></td>
</tr>
</tbody>
</table>

Fields

`  encryptionType  `

`  enum ( Type  ` )

Output only. The type of encryption.

`  encryptionStatus  `

`  object ( Status  ` )

Output only. If present, the status of a recent encrypt/decrypt call on underlying data for this database or backup. Regardless of status, data is always encrypted at rest.

`  kmsKeyVersion  `

`  string  `

Output only. A Cloud KMS key version that is being used to protect the database or backup.

## Type

Possible encryption types.

Enums

`  TYPE_UNSPECIFIED  `

Encryption type was not specified, though data at rest remains encrypted.

`  GOOGLE_DEFAULT_ENCRYPTION  `

The data is encrypted at rest with a key that is fully managed by Google. No key version or status will be populated. This is the default state.

`  CUSTOMER_MANAGED_ENCRYPTION  `

The data is encrypted at rest with a key that is managed by the customer. The active version of the key. `  kmsKeyVersion  ` will be populated, and `  encryptionStatus  ` may be populated.
