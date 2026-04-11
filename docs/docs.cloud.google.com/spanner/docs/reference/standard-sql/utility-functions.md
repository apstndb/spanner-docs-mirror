GoogleSQL for Spanner supports the following utility functions.

## Function list

| Name                                                                                                                 | Summary                                                                     |
| -------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| [`GENERATE_UUID`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/utility-functions#generate_uuid) | Produces a random universally unique identifier (UUID) as a `STRING` value. |
| [`NEW_UUID`](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/utility-functions#new_uuid)           | Produces a random universally unique identifier (UUID) as a `UUID` value.   |

## `GENERATE_UUID`

    GENERATE_UUID()

**Description**

Returns a random universally unique identifier (UUID) as a `STRING` . The returned `STRING` consists of 32 hexadecimal digits in five groups separated by hyphens in the form 8-4-4-4-12. The hexadecimal digits represent 122 random bits and 6 fixed bits, in compliance with [RFC 4122 section 4.4](https://tools.ietf.org/html/rfc4122#section-4.4) . The returned `STRING` is lowercase.

**Return Data Type**

STRING

**Example**

The following query generates a random UUID.

    SELECT GENERATE_UUID() AS uuid;
    
    /*--------------------------------------+
     | uuid                                 |
     +--------------------------------------+
     | 4192bff0-e1e0-43ce-a4db-912808c32493 |
     +--------------------------------------*/

## `NEW_UUID`

    NEW_UUID()

**Description**

Returns a random universally unique identifier (UUID) as a `UUID` . The returned `UUID` consists of 32 hexadecimal digits in five groups separated by hyphens in the form 8-4-4-4-12. The hexadecimal digits represent 122 random bits and 6 fixed bits, in compliance with [RFC 4122 section 4.4](https://tools.ietf.org/html/rfc4122#section-4.4) .

GoogleSQL accepts any number of hyphens with some caveats:

1.  Use single hyphens between hexadecimal numbers, no consecutive hyphens.
2.  Don't start or end a UUID with a hyphen.

**Return Data Type**

UUID

**Example**

The following query generates a random UUID.

    SELECT NEW_UUID() AS uuid;
    
    /*--------------------------------------+
     | uuid                                 |
     +--------------------------------------+
     | 4192bff0-e1e0-43ce-a4db-912808c32493 |
     +--------------------------------------*/
