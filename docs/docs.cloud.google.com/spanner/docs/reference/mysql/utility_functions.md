Spanner supports the following MySQL utility user-defined functions. You need to implement the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](https://docs.cloud.google.com/spanner/docs/install-mysql-functions) .

## Function list

| Name                                                                                                                                   | Summary                                                              |
| -------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| [`         mysql.BIN_TO_UUID        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#bin_to_uuid)       | Converts a binary UUID representation to a STRING representation.    |
| [`         mysql.INET_ATON        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#inet_aton)           | Returns the numeric value of an IP address.                          |
| [`         mysql.INET_NTOA        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#inet_ntoa)           | Returns the IP address from a numeric value.                         |
| [`         mysql.INET6_ATON        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#inet6_aton)         | Returns the numeric value of an IPv6 address.                        |
| [`         mysql.INET6_NTOA        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#inet6_ntoa)         | Returns the IPv6 address from a numeric value.                       |
| [`         mysql.IS_IPV4        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#is_ipv4)               | Returns whether the input parameter is an IPv4 address.              |
| [`         mysql.IS_IPV4_COMPAT        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#is_ipv4_compat) | Returns whether the input parameter is an IPv4-compatible address.   |
| [`         mysql.IS_IPV4_MAPPED        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#is_ipv4_mapped) | Returns whether the input parameter is an IPv4-mapped address.       |
| [`         mysql.IS_IPV6        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#is_ipv6)               | Returns whether the input parameter is an IPv6 address.              |
| [`         mysql.IS_UUID        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#is_uuid)               | Returns whether the input parameter is a valid UUID.                 |
| [`         mysql.UUID        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#uuid)                     | Returns a Universal Unique Identifier (UUID).                        |
| [`         mysql.UUID_TO_BIN        `](https://docs.cloud.google.com/spanner/docs/reference/mysql/utility_functions#uuid_to_bin)       | Converts a string representation of UUID to a binary representation. |

## `     mysql.BIN_TO_UUID    `

    mysql.BIN_TO_UUID(bytes_expression)

**Description**

Converts a binary representation of a UUID (16 bytes) into its standard 36-character string format.

This function supports the following argument:

  - `  bytes_expression  ` : The `  BYTES  ` value representing the UUID. It must be 16 bytes long.

**Return data type**

`  STRING  `

**Differences from MySQL**

The MySQL `  BIN_TO_UUID  ` function has an optional second argument to control the byte order for time-based (Version 1) UUIDs. This function doesn't support that argument and is generally used with Version 4 UUIDs that are randomly generated.

**Limitations**

This function doesn't support the two-argument version found in MySQL for swapping time parts. The input `  binary_uuid  ` must be exactly 16 bytes; otherwise, an error occurs.

**Example**

The following example converts a binary UUID (represented by a hex string) to its string format:

    SELECT mysql.BIN_TO_UUID(FROM_HEX('00112233445566778899AABBCCDDEEFF')) as uuid_string;
    
    /*
    +--------------------------------------+
    | uuid_string                          |
    +--------------------------------------+
    | 00112233-4455-6677-8899-aabbccddeeff |
    +--------------------------------------+
    */

## `     mysql.INET_ATON    `

    mysql.INET_ATON(string_expression)

**Description**

Converts a string representation of an IPv4 address (in dot-decimal notation) into its numeric equivalent, an integer.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` representation of the IPv4 address (for example, 192.168.1.1).

**Return data type**

`  INT64  `

**Limitations**

This function parses IPv4 addresses more strictly than the MySQL version might. Invalid IP address formats return `  NULL  ` or an error.

**Example**

The following example converts the IP address 10.0.0.1 to its numeric value:

    SELECT mysql.INET_ATON('10.0.0.1') as ip_numeric_value;
    
    /*
    +------------------+
    | ip_numeric_value |
    +------------------+
    | 167772161        |
    +------------------+
    */

## `     mysql.INET_NTOA    `

    mysql.INET_NTOA(numeric_expression)

**Description**

Converts a numeric representation of an IPv4 address (an integer) back into its string representation in dot-decimal notation.

This function supports the following argument:

  - `  numeric_expression  ` : The `  INT64  ` numeric value of the IPv4 address.

**Return data type**

`  STRING  `

**Limitations**

If the input number is outside the valid range for an IPv4 address (0 to 4294967295), the function returns `  NULL  ` .

**Example**

The following example converts the numeric value 167772161 to an IP address string:

    SELECT mysql.INET_NTOA(167772161) as ip_address_string;
    
    /*
    +-------------------+
    | ip_address_string |
    +-------------------+
    | 10.0.0.1          |
    +-------------------+
    */

## `     mysql.INET6_ATON    `

    mysql.INET6_ATON(string_expression)

**Description**

Converts a string representation of an IPv6 address (or an IPv4 address) into its binary representation as `  BYTES  ` .

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` representation of the IPv6 or IPv4 address.

**Return data type**

`  BYTES  `

**Example**

The following example converts the IPv6 address 2001:db8::1 to its binary representation and displays it as a hex string:

    SELECT TO_HEX(mysql.INET6_ATON('2001:db8::1')) as ipv6_bytes_hex;
    
    /*
    +----------------------------------+
    | ipv6_bytes_hex                   |
    +----------------------------------+
    | 20010DB8000000000000000000000001 |
    +----------------------------------+
    */

## `     mysql.INET6_NTOA    `

    mysql.INET6_NTOA(bytes_expression)

**Description**

Converts a binary representation of an IPv6 or IPv4 address ( `  BYTES  ` ) back into its standard string representation.

This function supports the following argument:

  - `  bytes_expression  ` : The `  BYTES  ` representation of the IPv6 or IPv4 address.

**Return data type**

`  STRING  `

**Example**

The following example converts a binary IPv6 address (represented by a hex string) back to its string format:

    SELECT mysql.INET6_NTOA(FROM_HEX('20010DB8000000000000000000000001')) as ipv6_string;
    
    /*
    +-------------+
    | ipv6_string |
    +-------------+
    | 2001:db8::1 |
    +-------------+
    */

## `     mysql.IS_IPV4    `

    mysql.IS_IPV4(string_expression)

**Description**

Checks if a given string is a valid IPv4 address.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to check.

**Return data type**

`  BOOL  `

**Example**

The following example checks if strings are valid IPv4 addresses:

    SELECT
      mysql.IS_IPV4('192.168.1.1') as example1_is_ipv4,
      mysql.IS_IPV4('2001:db8::1') as example2_is_ipv4,
      mysql.IS_IPV4('not-an-ip') as example3_is_ipv4;
    
    /*
    +------------------+------------------+------------------+
    | example1_is_ipv4 | example2_is_ipv4 | example3_is_ipv4 |
    +------------------+------------------+------------------+
    | true             | false            | false            |
    +------------------+------------------+------------------+
    */

## `     mysql.IS_IPV4_COMPAT    `

    mysql.IS_IPV4_COMPAT(string_expression)

**Description**

Checks if a given string represents an IPv4-compatible IPv6 address. An IPv4-compatible address has the form `  ::a.b.c.d  ` .

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to check.

**Return data type**

`  BOOL  `

**Differences from MySQL**

If you provide a `  NULL  ` input, this function returns `  FALSE  ` . In MySQL 5.7, `  IS_IPV4_COMPAT(NULL)  ` returns 0 (false), and in MySQL 8.0, it returns `  NULL  ` .

**Example**

The following example checks for an IPv4-compatible IPv6 address:

    SELECT mysql.IS_IPV4_COMPAT('::192.0.2.128') as is_ipv4_compatible;
    
    /*
    +--------------------+
    | is_ipv4_compatible |
    +--------------------+
    | true               |
    +--------------------+
    */

## `     mysql.IS_IPV4_MAPPED    `

    mysql.IS_IPV4_MAPPED(string_expression)

**Description**

Checks if a given string represents an IPv4-mapped IPv6 address. An IPv4-mapped address has the form `  ::ffff:a.b.c.d  ` .

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to check.

**Return data type**

`  BOOL  `

**Example**

The following example checks for an IPv4-mapped IPv6 address:

    SELECT mysql.IS_IPV4_MAPPED('::ffff:192.0.2.128') as is_ipv4_mapped;
    
    /*
    +----------------+
    | is_ipv4_mapped |
    +----------------+
    | true           |
    +----------------+
    */

## `     mysql.IS_IPV6    `

    mysql.IS_IPV6(string_expression)

**Description**

Checks if a given string is a valid IPv6 address.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to check.

**Return data type**

`  BOOL  `

**Example**

The following example checks if strings are valid IPv6 addresses:

    SELECT
      mysql.IS_IPV6('2001:db8::1') as example1_is_ipv6,
      mysql.IS_IPV6('192.168.1.1') as example2_is_ipv6, /* This is IPv4 */
      mysql.IS_IPV6('::ffff:192.0.2.128') as example3_is_ipv6; /* IPv4-mapped IPv6 */
    
    /*
    +------------------+------------------+------------------+
    | example1_is_ipv6 | example2_is_ipv6 | example3_is_ipv6 |
    +------------------+------------------+------------------+
    | true             | false            | true             |
    +------------------+------------------+------------------+
    */

## `     mysql.IS_UUID    `

    mysql.IS_UUID(string_expression)

**Description**

Checks if a given string is a valid universally unique identifier (UUID) in the standard 8-4-4-4-12 hexadecimal format.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` to check.

**Return data type**

`  BOOL  `

**Example**

The following example checks if strings are valid UUIDs:

    SELECT
      mysql.IS_UUID('550e8400-e29b-41d4-a716-446655440000') as is_valid_uuid,
      mysql.IS_UUID('not-a-uuid') as is_invalid_uuid;
    
    /*
    +---------------+-----------------+
    | is_valid_uuid | is_invalid_uuid |
    +---------------+-----------------+
    | true          | false           |
    +---------------+-----------------+
    */

## `     mysql.UUID    `

    mysql.UUID()

**Description**

Returns a Version 4 universally unique identifier (UUID) as a string.

This function doesn't support any arguments.

**Return data type**

`  STRING  `

**Differences from MySQL**

Both this function and MySQL's `  UUID  ` function comply with [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122) . However, MySQL typically generates Version 1 UUIDs (based on current time and MAC address), while this function generates Version 4 UUIDs (based on random numbers).

**Example**

The following example generates a UUID:

    SELECT mysql.UUID() as generated_uuid;
    
    /*
    +--------------------------------------+
    | generated_uuid                       |
    +--------------------------------------+
    | 123e4567-e89b-12d3-a456-426614174000 |
    +--------------------------------------+
    */

## `     mysql.UUID_TO_BIN    `

    mysql.UUID_TO_BIN(string_expression)

**Description**

Converts a UUID string (in standard 8-4-4-4-12 format) into its 16-byte binary representation.

This function supports the following argument:

  - `  string_expression  ` : The `  STRING  ` representation of the UUID.

**Return data type**

`  BYTES  `

**Differences from MySQL**

MySQL's `  UUID_TO_BIN  ` function has an optional second argument to control byte order for Version 1 UUIDs. This function doesn't support that optional argument.

**Limitations**

This function doesn't use the optional second argument for swapping time-low and time-high parts of the UUID, as they do in standard MySQL. If the input string is not a valid UUID format, an error occurs.

**Example**

The following example converts a UUID string to its binary representation and displays it as a hex string:

    SELECT TO_HEX(mysql.UUID_TO_BIN('00112233-4455-6677-8899-aabbccddeeff')) as uuid_bytes_hex;
    
    /*
    +----------------------------------+
    | uuid_bytes_hex                   |
    +----------------------------------+
    | 00112233445566778899AABBCCDDEEFF |
    +----------------------------------+
    */
