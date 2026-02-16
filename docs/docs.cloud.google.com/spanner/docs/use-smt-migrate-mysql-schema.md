This document describes how to use the Spanner migration tool (SMT) to migrate your MySQL schema to Spanner. SMT can read your MySQL schema and convert it to Spanner schema, remove duplicate indexes, and offer schema optimization suggestions.

## Before you begin

Ensure you've installed and set up SMT. For more information, see [Set up Spanner migration tool](/spanner/docs/set-up-spanner-migration-tool) .

## Configure schema

Once you've connected your source database and Spanner with SMT, the tool reads your MySQL schema and converts it to Spanner schema. The tool doesn't convert stored procedures or triggers.

To view a report on the schema conversion, go to the **Configure Schema** page on the web UI and click **View assessment** .

This report provides an overall assessment of schema conversion and also provides detailed table-level and column-level conversion information, suggestions and warnings, and lists schema elements that couldn't be converted.

## Modify schema

On the **Configure Schema** page, you can view your source MySQL database schema and the draft of the Spanner schema. You can also modify the converted schema to fit your organization's schema requirements.

You can select tables or indexes, view and manage their schema.

### Tables

Select a table you want to modify on the **Spanner draft** tab on the web UI. You can drop or restore tables on this tab. For each table you select, you can view the following list of tabs:

  - Columns
  - Primary key
  - Foreign key
  - Check constraints
  - SQL

#### Column

The **Column** tab provides information about the columns in the selected table. You can edit the columns in the following ways:

  - Modify a column name
  - Delete a column
  - Change the column's data type
  - Add auto-generated IDs
  - Modify the default value
  - Modify the null property

Besides editing existing columns in the Spanner draft, you can also add new columns to the selected table.

#### Primary key

You can view and edit the table's primary key from the **Primary key** tab in the following ways:

  - Add or remove a column from a primary key
  - Change the order of columns in a primary key

You can also use auto-generated columns for primary keys. You can choose one of the following to create auto-generated columns.

  - **[UUID function](/spanner/docs/reference/standard-sql/utility-functions#generate_uuid)** : generate a UUID v4 as part of the table's primary key `  default  ` expression.
  - **[Bit-reverse function](/spanner/docs/reference/standard-sql/bit_functions#bit_reverse)** : map existing integer keys as a bit-reversed sequence.

#### Foreign key

You can view and edit the table's foreign key from the **Foreign key** tab in the following ways:

  - Change the foreign key constraint name
  - Drop the foreign key if you want to use an interleaved table instead
  - Convert an interleaved table back to a foreign key

#### Check constraints

You can view and edit the table's check constraints using the **Check constraints** tab in the following ways:

  - Change the check constraint name or condition
  - Remove the check constraint

#### SQL

You can view the Spanner data definition language in the GoogleSQL dialect in the **SQL** tab.

### Indexes

Select an index you want to modify on the **Spanner draft** tab on the web UI. You can edit the index in the following ways:

  - Drop or restore the index
  - Add a secondary index
  - View the Spanner DDL in the **SQL** tab.

## Prepare migration

Once you complete configuring your schema, you can download the schema as a text file and use the DDL to create the schema in your target Spanner instance.

To download the schema, go to the **Prepare Migration** page on the web UI and do the following:

1.  In the **Migration Mode** drop-down, select **Schema** .
2.  Click **Download** to download the schema DDL as a text file.
