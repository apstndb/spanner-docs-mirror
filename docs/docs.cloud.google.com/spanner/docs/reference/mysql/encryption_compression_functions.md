Spanner supports the following MySQL encryption and compression functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

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
<td><a href="#sha2"><code dir="ltr" translate="no">        mysql.SHA2       </code></a></td>
<td>Calculates a SHA-2 checksum.</td>
</tr>
</tbody>
</table>

## `     mysql.SHA2    `

``` text
mysql.SHA2(bytes_expression, hash_length)
```

**Description**

Calculates an SHA-2 checksum for the input `  BYTES  ` .

This function supports the following arguments:

  - `  bytes_expression  ` : The `  BYTES  ` value for which to calculate the checksum.
  - `  hash_length  ` : The chosen bit length of the hash. Supported values are 256 and 512.

**Return data type**

`  STRING  `

**Differences from MySQL**

The MySQL version of `  SHA2()  ` also supports SHA-224 and SHA-384 hash lengths, which this function does not. Additionally, this function returns the checksum as a hexadecimal `  STRING  ` , while the MySQL version returns binary data.

**Limitations**

This function only supports hash lengths of 256 and 512 bits. The output is always a hexadecimal string, not binary data. If you provide an unsupported `  hash_length  ` , the function returns an error.

**Example**

The following example calculates the SHA-256 checksum for a `  BYTES  ` value:

``` text
SELECT mysql.SHA2(B'GoogleCloud', 256) as sha256_checksum;

/*
+------------------------------------------------------------------+
| sha256_checksum                                                  |
+------------------------------------------------------------------+
| 2002A3F1F350598F5FAF799AD9C9936908230796712E24682016E5257A4D2000 |
+------------------------------------------------------------------+
*/
```
