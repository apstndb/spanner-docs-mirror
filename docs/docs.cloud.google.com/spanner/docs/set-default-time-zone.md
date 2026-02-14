Spanner provides [date](/spanner/docs/reference/standard-sql/date_functions) and [timestamp](/spanner/docs/reference/standard-sql/timestamp_functions) functions in GoogleSQL and Cloud SQL for PostgreSQL. Some functions, such as [`  TIMESTAMP  `](/spanner/docs/reference/standard-sql/timestamp_functions#timestamp) , are time zone dependent and accept an optional time zone parameter. If no time zone parameter is provided in a function, Spanner databases default to the `  America/Los_Angeles  ` time zone.

Spanner lets you change the default time zone of a database to customize this behavior.

## Limitations

  - You can only change the time zone of empty databases without any tables.
  - Providing a time zone parameter within a statement overrides the database's default time zone for that statement.
  - All timestamps in the [REST and RPC APIs](/spanner/docs/reference/standard-sql/data-types#for_rest_and_rpc_apis) must use UTC and end with an uppercase `  Z  ` .
  - Timestamps in query results are consistently presented in UTC, with `  Z  ` appended. Display time zone conversions are not performed.

## Required roles

To get the permissions that you need to set the default time zone of a database, ask your administrator to grant you the [Cloud Spanner Database Admin](/iam/docs/roles-permissions/spanner#spanner.databaseAdmin) ( `  roles/spanner.databaseAdmin  ` ) IAM role on the database. For more information about granting roles, see [Manage access to projects, folders, and organizations](/iam/docs/granting-changing-revoking-access) .

This predefined role contains the permissions required to set the default time zone of a database. To see the exact permissions that are required, expand the **Required permissions** section:

#### Required permissions

The following permissions are required to set the default time zone of a database:

  - set the default time zone of a database: `  spanner.databases.getDdl, spanner.databases.updateDdl  `

You might also be able to get these permissions with [custom roles](/iam/docs/creating-custom-roles) or other [predefined roles](/iam/docs/roles-overview#predefined) .

## Set the default time zone

To change the default time zone of your database, run the following statement:

### GoogleSQL

Use the [`  ALTER DATABASE  ` statement](/spanner/docs/reference/standard-sql/data-definition-language#alter-database) :

``` text
ALTER DATABASE DATABASE-NAME SET OPTIONS (default_time_zone = 'TIME-ZONE-NAME');
```

Replace the following:

  - DATABASE-NAME : the name of the database. For example, `  my-database  ` .
  - TIME-ZONE-NAME : the name of the time zone to set the database default to. Must be a valid entry from the [IANA Time Zone Database](https://www.iana.org/time-zones) . For example, `  Etc/UTC  ` .

### PostgreSQL

Use the [`  ALTER DATABASE  ` statement](/spanner/docs/reference/postgresql/data-definition-language#alter-database) :

``` text
ALTER DATABASE DATABASE-NAME SET spanner.default_time_zone = 'TIME-ZONE-NAME';
```

Replace the following:

  - DATABASE-NAME : the name of the database. For example, `  my-database  ` .
  - TIME-ZONE-NAME : the name of the time zone to set the database default to. Must be a valid entry from the [IANA Time Zone Database](https://www.iana.org/time-zones) . For example, `  Etc/UTC  ` .

## Examples

The following example queries show how to use the default time zone option.

### Default time zone not customized

If the default\_time\_zone option is not explicitly set in the database schema, then the value of default\_time\_zone is null and Spanner uses `  America/Los_Angeles  ` as the default time zone. `  America/Los_Angeles  ` has an offset of UTC-8 for timestamps in the following examples.

Statement:

### GoogleSQL

``` text
SELECT TIMESTAMP("2072-12-25 15:30:00") AS timestamp_str;
```

### PostgreSQL

``` text
SELECT '2072-12-25 15:30:00'::timestamptz AS timestamp_str;
```

Output:

``` text
/*----------------------*
 | timestamp_str        |
 +----------------------+
 | 2072-12-25T23:30:00Z |
 *----------------------*/
```

Statement:

### GoogleSQL

``` text
SELECT EXTRACT(HOUR FROM TIMESTAMP("2072-12-25 15:30:00Z")) AS hour;
```

### PostgreSQL

``` text
SELECT EXTRACT(HOUR FROM '2072-12-25 15:30:00Z'::timestamptz) AS hour;
```

Output:

``` text
/*------*
 | hour |
 +------+
 |  7   |
 *------*/
```

Statement:

### GoogleSQL

``` text
SELECT TIMESTAMP_TRUNC(TIMESTAMP "2072-12-25 15:30:00Z", DAY) AS date_str;
```

### PostgreSQL

``` text
SELECT DATE_TRUNC('day', TIMESTAMPTZ '2072-12-25 15:30:00Z') AS date_str;
```

Output:

``` text
/*----------------------*
 | date_str             |
 +----------------------+
 | 2072-12-25T08:00:00Z |
 *----------------------*/
```

### Default time zone option set to `     Etc/UTC    `

The following examples show how the same statements behave when the default time zone option is set to `  Etc/UTC  ` .

Statement:

### GoogleSQL

``` text
SELECT TIMESTAMP("2072-12-25 15:30:00") AS timestamp_str;
```

### PostgreSQL

``` text
SELECT '2072-12-25 15:30:00'::timestamptz AS timestamp_str;
```

Output:

``` text
/*----------------------*
 | timestamp_str        |
 +----------------------+
 | 2072-12-25T15:30:00Z |
 *----------------------*/
```

Statement:

### GoogleSQL

``` text
SELECT EXTRACT(HOUR FROM TIMESTAMP("2072-12-25 15:30:00Z")) AS hour;
```

### PostgreSQL

``` text
SELECT EXTRACT(HOUR FROM '2072-12-25 15:30:00Z'::timestamptz) AS hour;
```

Output:

``` text
/*------*
 | hour |
 +------+
 | 15   |
 *------*/
```

Statement:

### GoogleSQL

``` text
SELECT TIMESTAMP_TRUNC(TIMESTAMP "2072-12-25 15:30:00Z", DAY) AS date_str;
```

### PostgreSQL

``` text
SELECT DATE_TRUNC('day', TIMESTAMPTZ '2072-12-25 15:30:00Z') AS date_str;
```

Output:

``` text
/*----------------------*
 | date_str             |
 +----------------------+
 | 2072-12-25T00:00:00Z |
 *----------------------*/
```

### Default time zone overridden by function parameter

When a function or string literal includes a defined time zone parameter, the database's default time zone isn't applied.

### GoogleSQL

Statement:

``` text
SELECT FORMAT_TIMESTAMP("%c", TIMESTAMP "2050-12-25 15:30:55+00", "Australia/Sydney")
AS formatted;
```

Output:

``` text
/*--------------------------*
| formatted                |
+--------------------------+
| Mon Dec 26 02:30:55 2050 |
*--------------------------*/
```

Statement:

``` text
SELECT TIMESTAMP("2072-12-25 15:30:00+11:00") AS timestamp_str;
```

Output:

``` text
/*----------------------*
| timestamp_str        |
+----------------------+
| 2072-12-25T04:30:00Z |
*----------------------*/
```

### PostgreSQL

Statement:

``` text
SELECT '2072-12-25 15:30:00+11:00'::timestamptz AS timestamp_str;
```

Output:

``` text
/*----------------------*
| timestamp_str        |
+----------------------+
| 2072-12-25T04:30:00Z |
*----------------------*/
```
