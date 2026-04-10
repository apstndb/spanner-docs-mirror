This page contains information about the PostgreSQL system catalog tables available in Spanner.

[PostgreSQL system catalog tables](https://www.postgresql.org/docs/current/catalogs.html) store schema data, such as information about tables and columns, and internal bookkeeping information. These system catalog tables are part of a namespace called `  pg_catalog  ` .

## Limitations and differences

This section lists the columns that are missing and other differences between the Spanner implementation of PostgreSQL system catalog tables and PostgreSQL.

### Unsupported columns

Spanner doesn't support every column in its implementation of PostgreSQL system catalog tables. The following columns aren't supported and therefore don't show up in the table.

| Catalog table name            | Missing column names                                                                                                                                                                                                                                             |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `        pg_am       `        | `        amhandler       `                                                                                                                                                                                                                                       |
| `        pg_attribute       ` | `        attacl       ` , `        attmissingval       `                                                                                                                                                                                                         |
| `        pg_class       `     | `        relacl       `                                                                                                                                                                                                                                          |
| `        pg_namespace       ` | `        nspacl       `                                                                                                                                                                                                                                          |
| `        pg_proc       `      | `        prosupport       ` , `        proacl       `                                                                                                                                                                                                            |
| `        pg_sequences       ` | `        data_type       `                                                                                                                                                                                                                                       |
| `        pg_type       `      | `        typsubscript       ` , `        typinput       ` , `        typoutput       ` , `        typreceive       ` , `        typsend       ` , `        typmodin       ` , `        typmodout       ` , `        typanalyze       ` , `        typacl       ` |

### Empty columns

Some columns in supported system catalog tables don't contain data. These columns could have content added later so avoid queries that rely on these columns being empty. For each table listed on this page, supported columns are listed, along with information about whether or not the column has content.

### Empty tables

Some tables and views have no data in them. This is so that queries can refer to them without encountering errors. This improves compatibility with tools without requiring rewrites to avoid those tables.

### Difference from PostgreSQL

In PostgreSQL, you can write to these tables directly to modify the database. In Spanner, system tables are read-only.

The Spanner implementation contains the following changes to PostgreSQL system catalog columns:

  - `  pg_node_tree  ` columns (most notably `  pg_attrdef.adbin  ` and `  pg_proc.prosqlbody  ` ) are text columns that contain the SQL expressions.

  - `  oidvector  ` columns (most notably `  pg_proc.proargtypes  ` ) are `  oid[]  ` columns.

  - `  int2vector  ` columns are `  int8[]  ` columns.

  - `  pg_class.relnatts  ` column is `  NULL  ` for sequences because you can't select them in Spanner.

  - `  pg_collation  ` only contains default and C. General collation support isn't available.

## PostgreSQL system catalog tables list

This section contains the PostgreSQL system catalog tables that Spanner supports.

### `     pg_am    `

The [`  pg_am  `](https://www.postgresql.org/docs/current/catalog-pg-am.html) table has content.

The following table shows whether columns have content.

| Column name             | Type                  | Has content |
| ----------------------- | --------------------- | ----------- |
| `        oid       `    | `        oid       `  | Y           |
| `        amname       ` | `        text       ` | Y           |
| `        amtype       ` | `        char       ` | Y           |

### `     pg_attrdef    `

The [`  pg_attrdef  `](https://www.postgresql.org/docs/current/catalog-pg-attrdef.html) table has content.

The following table shows whether columns have content.

| Column name              | Type                  | Has content |
| ------------------------ | --------------------- | ----------- |
| `        oid       `     | `        oid       `  | Y           |
| `        adrelid       ` | `        oid       `  | Y           |
| `        adnum       `   | `        int8       ` | Y           |
| `        adbin       `   | `        text       ` | Y           |

The `  adbin  ` column should contain a `  text  ` value that represents the serialized value of the default value.

### `     pg_attribute    `

The [`  pg_attribute  `](https://www.postgresql.org/docs/current/catalog-pg-attribute.html) table has content.

The following table shows whether columns have content.

| Column name                     | Type                    | Has content |
| ------------------------------- | ----------------------- | ----------- |
| `        attrelid       `       | `        oid       `    | Y           |
| `        attname       `        | `        text       `   | Y           |
| `        atttypid       `       | `        oid       `    | Y           |
| `        attstattarget       `  | `        int8       `   | N           |
| `        attlen       `         | `        int8       `   | N           |
| `        attnum       `         | `        int8       `   | Y           |
| `        attndims       `       | `        int8       `   | Y           |
| `        attcacheoff       `    | `        int8       `   | Y           |
| `        atttypmod       `      | `        int8       `   | N           |
| `        attbyval       `       | `        bool       `   | N           |
| `        attalign       `       | `        char       `   | N           |
| `        attstorage       `     | `        char       `   | N           |
| `        attcompression       ` | `        char       `   | Y           |
| `        attnotnull       `     | `        bool       `   | Y           |
| `        atthasdef       `      | `        bool       `   | Y           |
| `        atthasmissing       `  | `        bool       `   | Y           |
| `        attidentity       `    | `        char       `   | Y           |
| `        attgenerated       `   | `        char       `   | Y           |
| `        attisdropped       `   | `        bool       `   | Y           |
| `        attislocal       `     | `        bool       `   | Y           |
| `        attinhcount       `    | `        int8       `   | Y           |
| `        attcollation       `   | `        oid       `    | N           |
| `        attoptions       `     | `        text[]       ` | N           |
| `        attfdwoptions       `  | `        text[]       ` | N           |

### `     pg_class    `

The [`  pg_class  `](https://www.postgresql.org/docs/current/catalog-pg-class.html)

has content.

The following table shows whether columns have content.

| Column name                          | Type                    | Has content |
| ------------------------------------ | ----------------------- | ----------- |
| `        oid       `                 | `        oid       `    | Y           |
| `        relname       `             | `        text       `   | Y           |
| `        relnamespace       `        | `        oid       `    | Y           |
| `        reltype       `             | `        oid       `    | N           |
| `        reloftype       `           | `        oid       `    | N           |
| `        relowner       `            | `        oid       `    | N           |
| `        relam       `               | `        oid       `    | Y           |
| `        relfilenode       `         | `        oid       `    | N           |
| `        reltablespace       `       | `        oid       `    | N           |
| `        relpages       `            | `        int8       `   | N           |
| `        reltuples       `           | `        float8       ` | N           |
| `        relallvisible       `       | `        int8       `   | N           |
| `        reltoastrelid       `       | `        oid       `    | N           |
| `        relhasindex       `         | `        bool       `   | Y           |
| `        relisshared       `         | `        bool       `   | N           |
| `        relpersistence       `      | `        char       `   | Y           |
| `        relkind       `             | `        char       `   | Y           |
| `        relnatts       `            | `        int8       `   | Y           |
| `        relchecks       `           | `        int8       `   | Y           |
| `        relhasrules       `         | `        bool       `   | N           |
| `        relhastriggers       `      | `        bool       `   | N           |
| `        relhassubclass       `      | `        bool       `   | N           |
| `        relrowsecurity       `      | `        bool       `   | N           |
| `        relforcerowsecurity       ` | `        bool       `   | N           |
| `        relispopulated       `      | `        bool       `   | Y           |
| `        relreplident       `        | `        char       `   | N           |
| `        relispartition       `      | `        bool       `   | N           |
| `        relrewrite       `          | `        oid       `    | N           |
| `        relfrozenxid       `        | `        int8       `   | N           |
| `        relminmxid       `          | `        int8       `   | N           |
| `        reloptions       `          | `        text[]       ` | N           |
| `        relpartbound       `        | `        text       `   | N           |

### `     pg_collation    `

The [`  pg_collation  `](https://www.postgresql.org/docs/current/catalog-pg-collation.html) table has content.

The following table shows whether columns have content.

| Column name                          | Type                  | Has content |
| ------------------------------------ | --------------------- | ----------- |
| `        oid       `                 | `        oid       `  | Y           |
| `        collname       `            | `        text       ` | Y           |
| `        collnamespace       `       | `        oid       `  | Y           |
| `        collowner       `           | `        oid       `  | N           |
| `        collprovider       `        | `        char       ` | Y           |
| `        collisdeterministic       ` | `        bool       ` | Y           |
| `        collencoding       `        | `        int8       ` | Y           |
| `        collcollate       `         | `        text       ` | N           |
| `        collctype       `           | `        text       ` | N           |
| `        colliculocale       `       | `        text       ` | N           |
| `        collversion       `         | `        text       ` | N           |

### `     pg_constraint    `

The [`  pg_constraint  `](https://www.postgresql.org/docs/current/catalog-pg-constraint.html) table has content.

The following table shows whether columns have content.

| Column name                     | Type                    | Has content |
| ------------------------------- | ----------------------- | ----------- |
| `        oid       `            | `        oid       `    | Y           |
| `        conname       `        | `        text       `   | Y           |
| `        connamespace       `   | `        oid       `    | Y           |
| `        contype       `        | `        char       `   | Y           |
| `        condeferrable       `  | `        bool       `   | N           |
| `        condeferred       `    | `        bool       `   | N           |
| `        convalidated       `   | `        bool       `   | Y           |
| `        conrelid       `       | `        oid       `    | Y           |
| `        contypid       `       | `        oid       `    | N           |
| `        conindid       `       | `        oid       `    | N           |
| `        conparentid       `    | `        oid       `    | N           |
| `        confrelid       `      | `        oid       `    | Y           |
| `        confupdtype       `    | `        char       `   | Y           |
| `        confdeltype       `    | `        char       `   | Y           |
| `        confmatchtype       `  | `        char       `   | N           |
| `        conislocal       `     | `        bool       `   | N           |
| `        coninhcount       `    | `        int8       `   | N           |
| `        connoinherit       `   | `        bool       `   | N           |
| `        conkey       `         | `        int8[]       ` | Y           |
| `        confkey       `        | `        int8[]       ` | Y           |
| `        conpfeqop       `      | `        oid[]       `  | N           |
| `        conppeqop       `      | `        oid[]       `  | N           |
| `        conffeqop       `      | `        oid[]       `  | N           |
| `        conexclop       `      | `        oid[]       `  | N           |
| `        confdelsetcols       ` | `        int8[]       ` | N           |
| `        conbin       `         | `        text       `   | N           |

### `     pg_description    `

The [`  pg_description  `](https://www.postgresql.org/docs/current/catalog-pg-description.html) table has no content.

The following table shows whether columns have content.

| Column name                  | Type                   | Has content |
| ---------------------------- | ---------------------- | ----------- |
| `        objoid       `      | `        oid       `   | N           |
| `        classoid       `    | `        oid       `   | N           |
| `        objsubid       `    | `        int64       ` | N           |
| `        description       ` | `        text       `  | N           |

### `     pg_enum    `

The [`  pg_enum  `](https://www.postgresql.org/docs/current/catalog-pg-enum.html)

table has no content.

The following table shows whether columns have content.

| Column name                    | Type                    | Has content |
| ------------------------------ | ----------------------- | ----------- |
| `        oid       `           | `        oid       `    | N           |
| `        enumtypid       `     | `        oid       `    | N           |
| `        collnamespace       ` | `        float8       ` | N           |
| `        enumlabel       `     | `        text       `   | N           |

### `     pg_index    `

The [`  pg_index  `](https://www.postgresql.org/docs/current/catalog-pg-index.html)

table has content.

The following table shows whether columns have content.

| Column name                          | Type                    | Has content |
| ------------------------------------ | ----------------------- | ----------- |
| `        indexrelid       `          | `        oid       `    | Y           |
| `        indrelid       `            | `        oid       `    | Y           |
| `        indnatts       `            | `        int8       `   | Y           |
| `        indnkeyatts       `         | `        int8       `   | Y           |
| `        indisunique       `         | `        bool       `   | Y           |
| `        indnullsnotdistinct       ` | `        bool       `   | N           |
| `        indisprimary       `        | `        bool       `   | Y           |
| `        indisexclusion       `      | `        bool       `   | Y           |
| `        indimmediate       `        | `        bool       `   | N           |
| `        indisclustered       `      | `        bool       `   | Y           |
| `        indisvalid       `          | `        bool       `   | Y           |
| `        indcheckxmin       `        | `        bool       `   | Y           |
| `        indisready       `          | `        bool       `   | Y           |
| `        indislive       `           | `        bool       `   | Y           |
| `        indisreplident       `      | `        bool       `   | Y           |
| `        indkey       `              | `        int8[]       ` | N           |
| `        indcollation       `        | `        oid[]       `  | N           |
| `        indclass       `            | `        oid[]       `  | N           |
| `        indoption       `           | `        int8[]       ` | N           |
| `        indexprs       `            | `        text       `   | N           |
| `        indpred       `             | `        text       `   | N           |

### `     pg_namespace    `

The [`  pg_namespace  `](https://www.postgresql.org/docs/current/catalog-pg-namespace.html) table has content.

The following table shows whether columns have content.

| Column name               | Type                  | Has content |
| ------------------------- | --------------------- | ----------- |
| `        oid       `      | `        oid       `  | Y           |
| `        nspname       `  | `        text       ` | Y           |
| `        nspowner       ` | `        oid       `  | N           |

### `     pg_proc    `

The [`  pg_proc  `](https://www.postgresql.org/docs/current/catalog-pg-proc.html) table has content.

The following table shows whether columns have content.

| Column name                      | Type                    | Has content |
| -------------------------------- | ----------------------- | ----------- |
| `        oid       `             | `        oid       `    | Y           |
| `        proname       `         | `        text       `   | Y           |
| `        pronamespace       `    | `        oid       `    | Y           |
| `        proowner       `        | `        oid       `    | N           |
| `        prolang       `         | `        oid       `    | N           |
| `        procost       `         | `        float8       ` | N           |
| `        prorows       `         | `        float8       ` | N           |
| `        provariadic       `     | `        oid       `    | Y           |
| `        prokind       `         | `        char       `   | Y           |
| `        prosecdef       `       | `        bool       `   | N           |
| `        proleakproof       `    | `        bool       `   | N           |
| `        proisstrict       `     | `        bool       `   | N           |
| `        proretset       `       | `        bool       `   | Y           |
| `        provolatile       `     | `        char       `   | N           |
| `        proparallel       `     | `        char       `   | N           |
| `        pronargs       `        | `        int8       `   | Y           |
| `        pronargdefaults       ` | `        int8       `   | Y           |
| `        prorettype       `      | `        oid       `    | Y           |
| `        proargtypes       `     | `        oid[]       `  | Y           |
| `        proallargtypes       `  | `        oid[]       `  | N           |
| `        proargmodes       `     | `        char[]       ` | N           |
| `        proargdefaults       `  | `        text       `   | N           |
| `        protrftypes       `     | `        oid[]       `  | N           |
| `        prosrc       `          | `        text[]       ` | N           |
| `        probin       `          | `        text[]       ` | N           |
| `        prosqlbody       `      | `        text       `   | Y           |
| `        proconfig       `       | `        text[]       ` | N           |

### `     pg_sequence    `

The [`  pg_sequence  `](https://www.postgresql.org/docs/current/catalog-pg-sequence.html) table has content.

The following table shows whether columns have content.

| Column name                   | Type                  | Has content |
| ----------------------------- | --------------------- | ----------- |
| `        seqrelid       `     | `        oid       `  | Y           |
| `        seqtypid       `     | `        oid       `  | Y           |
| `        seqstart       `     | `        int8       ` | Y           |
| `        seqincrement       ` | `        int8       ` | N           |
| `        seqmax       `       | `        int8       ` | N           |
| `        seqmin       `       | `        int8       ` | N           |
| `        seqcache       `     | `        int8       ` | Y           |
| `        seqcycle       `     | `        bool       ` | Y           |

### `     pg_type    `

The [`  pg_type  `](https://www.postgresql.org/docs/current/catalog-pg-type.html) table has content.

The following table shows whether columns have content.

| Column name                     | Type                  | Has content |
| ------------------------------- | --------------------- | ----------- |
| `        oid       `            | `        oid       `  | Y           |
| `        typname       `        | `        text       ` | Y           |
| `        typnamespace       `   | `        oid       `  | Y           |
| `        typowner       `       | `        oid       `  | N           |
| `        typlen       `         | `        int8       ` | Y           |
| `        typbyval       `       | `        bool       ` | Y           |
| `        typtype       `        | `        char       ` | Y           |
| `        typcategory       `    | `        char       ` | Y           |
| `        typispreferred       ` | `        bool       ` | Y           |
| `        typisdefined       `   | `        bool       ` | Y           |
| `        typdelim       `       | `        char       ` | Y           |
| `        typrelid       `       | `        oid       `  | Y           |
| `        typelem       `        | `        oid       `  | Y           |
| `        typarray       `       | `        oid       `  | Y           |
| `        typalign       `       | `        char       ` | N           |
| `        typstorage       `     | `        char       ` | N           |
| `        typnotnull       `     | `        bool       ` | N           |
| `        typbasetype       `    | `        oid       `  | N           |
| `        typtypmod       `      | `        int8       ` | N           |
| `        typndims       `       | `        int8       ` | N           |
| `        typcollation       `   | `        oid       `  | N           |
| `        typdeafultbin       `  | `        text       ` | N           |
| `        typdefault       `     | `        text       ` | N           |

## What's next

  - [System views](https://docs.cloud.google.com/spanner/docs/reference/postgresql/pg-system-catalog-views)
