This page points out differences in behavior between the PostgreSQL capabilities supported in Spanner and their open source PostgreSQL equivalents.

**Caution:** The following specific behaviors may change to better align with open source PostgreSQL.

## Float8 underflow

In some cases, open source PostgreSQL returns an error such as `  ERROR: value out of range: underflow  ` if the result of floating point math would be truncated to 0. Numbers too close to zero that are not representable as distinct from zero cause an underflow error.

The PostgreSQL interface doesn't return an error in these cases; it returns the truncated 0 value instead.

## concat function

In the PostgreSQL interface, the `  concat()  ` function returns NULL if any argument is NULL. However, in open source PostgreSQL, this function ignores NULLs and returns a concatenation of all non-NULL arguments, or an empty string if all arguments are NULL. For example:

``` text
-- Returns `abcdef` in open source PostgreSQL.
-- Returns NULL in the PostgreSQL interface.
select concat('abc', NULL, 'def');
```

The PostgreSQL interface and open source PostgreSQL have identical behavior for the `  ||  ` operator with scalar (non-array) operands.

## array\_cat() function

In the PostgreSQL interface, the `  array_cat()  ` function returns NULL if any argument is NULL. However, in open source PostgreSQL, this function ignores NULLs and returns a concatenation of all non-NULL array arguments, or a NULL array if all arguments are NULL. The same is true for the `  ||  ` operator with array operands. For example:

``` text
-- Returns `{abc, def}` in open source PostgreSQL.
-- Returns NULL in the PostgreSQL interface.
select '{abc}'::text[] || NULL::text[] || '{def}'::text[];
```

## array\_to\_string function

In the PostgreSQL interface, the `  array_to_string(array text_array, delimiter text [, null_string text ])  ` function returns NULL if the `  null_string  ` argument is NULL. However, in open source PostgreSQL, this function ignores the NULL array elements and returns a concatenation of all non-NULL elements as a string representation if the `  null_string  ` argument is NULL. For example:

``` text
-- Returns `a,b,c` in open source PostgreSQL.
-- Returns NULL in the PostgreSQL interface.
select array_to_string('{a,b,NULL,c}'::text[], ',', NULL);
```

## Object names

For more information about naming issues, see [Name](/spanner/docs/reference/postgresql/data-definition-language#names) .
