This page lists the PostgreSQL system catalog views that Spanner supports.

### pg\_available\_extension\_versions

The [`  pg_available_extension_versions  `](https://www.postgresql.org/docs/current/view-pg-available-extension-versions.html) view has no content.

The following table shows whether columns have content.

| Column name                  | Type                    | Has content |
| ---------------------------- | ----------------------- | ----------- |
| `        name       `        | `        text       `   | N           |
| `        version       `     | `        text       `   | N           |
| `        installed       `   | `        bool       `   | N           |
| `        superuser       `   | `        bool       `   | N           |
| `        trusted       `     | `        bool       `   | N           |
| `        relocatable       ` | `        bool       `   | N           |
| `        schema       `      | `        text       `   | N           |
| `        requires       `    | `        text[]       ` | N           |
| `        comment       `     | `        text       `   | N           |

### pg\_available\_extensions

The [`  pg_available_extensions  `](https://www.postgresql.org/docs/current/view-pg-available-extensions.html) view has no content.

The following table shows whether columns have content.

| Column name                        | Type                  | Has content |
| ---------------------------------- | --------------------- | ----------- |
| `        name       `              | `        text       ` | N           |
| `        default_version       `   | `        text       ` | N           |
| `        installed_version       ` | `        text       ` | N           |
| `        comment       `           | `        text       ` | N           |

### pg\_config

The [`  pg_config  `](https://www.postgresql.org/docs/current/view-pg-config.html)

view has no content.

The following table shows whether columns have content.

| Column name              | Type                  | Has content |
| ------------------------ | --------------------- | ----------- |
| `        name       `    | `        text       ` | N           |
| `        setting       ` | `        text       ` | N           |

### pg\_cursors

The [`  pg_cursors  `](https://www.postgresql.org/docs/current/view-pg-cursors.html)

view has no content.

The following table shows whether columns have content.

| Column name                    | Type                         | Has content |
| ------------------------------ | ---------------------------- | ----------- |
| `        name       `          | `        text       `        | N           |
| `        statement       `     | `        text       `        | N           |
| `        is_holdable       `   | `        bool       `        | N           |
| `        is_binary       `     | `        bool       `        | N           |
| `        is_scrollable       ` | `        bool       `        | N           |
| `        creation_time       ` | `        timestamptz       ` | N           |

### pg\_file\_settings

The [`  pg_file_settings  `](https://www.postgresql.org/docs/current/view-pg-file-settings.html) view has no content.

The following table shows whether columns have content.

| Column name                 | Type                  | Has content |
| --------------------------- | --------------------- | ----------- |
| `        sourcefile       ` | `        text       ` | N           |
| `        sourceline       ` | `        int8       ` | N           |
| `        seqno       `      | `        int8       ` | N           |
| `        name       `       | `        text       ` | N           |
| `        setting       `    | `        text       ` | N           |
| `        applied       `    | `        bool       ` | N           |
| `        error       `      | `        text       ` | N           |

### pg\_hba\_file\_rules

The [`  pg_hba_file_rules  `](https://www.postgresql.org/docs/current/view-pg-hba-file-rules.html) view has no content.

The following table shows whether columns have content.

| Column name                  | Type                    | Has content |
| ---------------------------- | ----------------------- | ----------- |
| `        rule_number       ` | `        int8       `   | N           |
| `        file_name       `   | `        text       `   | N           |
| `        line_number       ` | `        int8       `   | N           |
| `        type       `        | `        text       `   | N           |
| `        database       `    | `        text[]       ` | N           |
| `        user_name       `   | `        text[]       ` | N           |
| `        address       `     | `        text       `   | N           |
| `        netmask       `     | `        text       `   | N           |
| `        auth_method       ` | `        text       `   | N           |
| `        options       `     | `        text[]       ` | N           |
| `        error       `       | `        text       `   | N           |

### pg\_indexes

The [`  pg_indexes  `](https://www.postgresql.org/docs/current/view-pg-indexes.html)

view has content.

The following table shows whether columns have content.

| Column name                 | Type                  | Has content |
| --------------------------- | --------------------- | ----------- |
| `        schemaname       ` | `        text       ` | Y           |
| `        tablename       `  | `        text       ` | Y           |
| `        indexname       `  | `        text       ` | Y           |
| `        tablespace       ` | `        text       ` | N           |
| `        indexdef       `   | `        text       ` | N           |

### pg\_matviews

The [`  pg_matviews  `](https://www.postgresql.org/docs/current/view-pg-matviews.html)

view has no content.

The following table shows whether columns have content.

| Column name                   | Type                  | Has content |
| ----------------------------- | --------------------- | ----------- |
| `        schemaname       `   | `        text       ` | N           |
| `        matviewname       `  | `        text       ` | N           |
| `        matviewowner       ` | `        text       ` | N           |
| `        tablespace       `   | `        text       ` | N           |
| `        hasindexes       `   | `        bool       ` | N           |
| `        ispopulated       `  | `        bool       ` | N           |
| `        definition       `   | `        text       ` | N           |

### pg\_policies

The [`  pg_policies  `](https://www.postgresql.org/docs/current/view-pg-policies.html)

view has no content.

The following table shows whether columns have content.

| Column name                 | Type                    | Has content |
| --------------------------- | ----------------------- | ----------- |
| `        schemaname       ` | `        text       `   | N           |
| `        tablename       `  | `        text       `   | N           |
| `        policyname       ` | `        text       `   | N           |
| `        permissive       ` | `        text       `   | N           |
| `        roles       `      | `        text[]       ` | N           |
| `        cmd       `        | `        text       `   | N           |
| `        qual       `       | `        text       `   | N           |
| `        with_check       ` | `        text       `   | N           |

### pg\_prepared\_xacts

The [`  pg_prepared_xacts  `](https://www.postgresql.org/docs/current/view-pg-prepared-xacts.html) view has no content.

The following table shows whether columns have content.

| Column name                  | Type                         | Has content |
| ---------------------------- | ---------------------------- | ----------- |
| `        transaction       ` | `        int8       `        | N           |
| `        gid       `         | `        text       `        | N           |
| `        prepared       `    | `        timestamptz       ` | N           |
| `        owner       `       | `        text       `        | N           |
| `        database       `    | `        text       `        | N           |

### pg\_publication\_tables

The [`  pg_publication_tables  `](https://www.postgresql.org/docs/current/view-pg-publication-tables.html) view has no content.

The following table shows whether columns have content.

| Column name                 | Type                    | Has content |
| --------------------------- | ----------------------- | ----------- |
| `        pubname       `    | `        text       `   | N           |
| `        schemaname       ` | `        text       `   | N           |
| `        tablename       `  | `        text       `   | N           |
| `        attnames       `   | `        text[]       ` | N           |
| `        rowfilter       `  | `        text       `   | N           |

### pg\_roles

The [`  pg_roles  `](https://www.postgresql.org/docs/current/view-pg-roles.html) view has no content.

The following table shows whether columns have content.

| Column name                        | Type                         | Has content |
| ---------------------------------- | ---------------------------- | ----------- |
| `        rolname       `           | `        text       `        | N           |
| `        rolsuper       `          | `        bool       `        | N           |
| `        rolinherit       `        | `        bool       `        | N           |
| `        rolcreaterole       `     | `        bool       `        | N           |
| `        rolcreatedb       `       | `        bool       `        | N           |
| `        rolcanlogin       `       | `        bool       `        | N           |
| `        rolreplication       `    | `        bool       `        | N           |
| `        rolconnlimit       `      | `        int8       `        | N           |
| `        rolpassword       `       | `        text       `        | N           |
| `        rolvaliduntil       `     | `        timestamptz       ` | N           |
| `        rolbypassrls bool       ` | `        bool       `        | N           |
| `        rolconfig       `         | `        text[]       `      | N           |
| `        oid       `               | `        oid       `         | N           |

### pg\_rules

The [`  pg_rules  `](https://www.postgresql.org/docs/current/view-pg-rules.html) view has no content.

The following table shows whether columns have content.

| Column name                 | Type                  | Has content |
| --------------------------- | --------------------- | ----------- |
| `        schemaname       ` | `        text       ` | N           |
| `        tablename       `  | `        text       ` | N           |
| `        rulename       `   | `        text       ` | N           |
| `        definition       ` | `        text       ` | N           |

### pg\_sequences

The [`  pg_sequences  `](https://www.postgresql.org/docs/current/view-pg-sequences.html)

view has content.

The following table shows whether columns have content.

| Column name                    | Type                  | Has content |
| ------------------------------ | --------------------- | ----------- |
| `        schemaname       `    | `        text       ` | Y           |
| `        sequencename       `  | `        text       ` | Y           |
| `        sequenceowner       ` | `        text       ` | N           |
| `        start_value       `   | `        int8       ` | Y           |
| `        min_value       `     | `        int8       ` | N           |
| `        max_value       `     | `        int8       ` | N           |
| `        increment_by       `  | `        int8       ` | N           |
| `        cycle       `         | `        bool       ` | Y           |
| `        cache_size       `    | `        int8       ` | Y           |
| `        last_value       `    | `        int8       ` | N           |

### pg\_settings

The [`  pg_settings  `](https://www.postgresql.org/docs/current/view-pg-settings.html)

view has content.

The following table shows whether columns have content.

| Column name                      | Type                    | Has content |
| -------------------------------- | ----------------------- | ----------- |
| `        name       `            | `        text       `   | Y           |
| `        setting       `         | `        text       `   | Y           |
| `        unit       `            | `        text       `   | N           |
| `        category       `        | `        text       `   | Y           |
| `        short_desc       `      | `        text       `   | Y           |
| `        extra_desc       `      | `        text       `   | N           |
| `        context       `         | `        text       `   | Y           |
| `        vartype       `         | `        text       `   | Y           |
| `        source       `          | `        text       `   | Y           |
| `        min_val       `         | `        text       `   | Y           |
| `        max_val       `         | `        text       `   | Y           |
| `        enumvals       `        | `        text[]       ` | N           |
| `        boot_val       `        | `        text       `   | Y           |
| `        reset_val       `       | `        text       `   | Y           |
| `        sourcefile       `      | `        text       `   | N           |
| `        sourceline       `      | `        int8       `   | N           |
| `        pending_restart       ` | `        bool       `   | Y           |

### pg\_shmem\_allocations

The [`  pg_shmem_allocations  `](https://www.postgresql.org/docs/current/view-pg-shmem-allocations.html) view has no content.

The following table shows whether columns have content.

| Column name                     | Type                  | Has content |
| ------------------------------- | --------------------- | ----------- |
| `        name       `           | `        text       ` | N           |
| `        off       `            | `        int8       ` | N           |
| `        size       `           | `        int8       ` | N           |
| `        allocated_size       ` | `        int8       ` | N           |

### pg\_tables

The [`  pg_tables  `](https://www.postgresql.org/docs/current/view-pg-tables.html)

view has content.

The following table shows whether columns have content.

| Column name                  | Type                  | Has content |
| ---------------------------- | --------------------- | ----------- |
| `        schemaname       `  | `        text       ` | Y           |
| `        tablename       `   | `        text       ` | Y           |
| `        tableowner       `  | `        text       ` | N           |
| `        tablespace       `  | `        text       ` | N           |
| `        hasindexes       `  | `        bool       ` | Y           |
| `        hasrules       `    | `        bool       ` | N           |
| `        hastriggers       ` | `        bool       ` | N           |
| `        rowsecurity       ` | `        bool       ` | N           |

### pg\_views

The [`  pg_views  `](https://www.postgresql.org/docs/current/view-pg-views.html) view has content.

The following table shows whether columns have content.

| Column name                 | Type                  | Has content |
| --------------------------- | --------------------- | ----------- |
| `        schemaname       ` | `        text       ` | Y           |
| `        viewname       `   | `        text       ` | Y           |
| `        viewowner       `  | `        text       ` | N           |
| `        definition       ` | `        text       ` | Y           |

## What's next

  - [System catalogs](https://docs.cloud.google.com/spanner/docs/reference/postgresql/pg-system-catalog-tables)
