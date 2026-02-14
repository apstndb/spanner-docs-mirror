Data validation is the process of comparing data from both the source and the destination database tables to ensure they match.

The [Data Validation Tool (DVT)](https://pypi.org/project/google-pso-data-validator/#:%7E:text=The%20Data%20Validation%20Tool%20is,with%20multi%2Dleveled%20validation%20functions) is an open source tool that can connect to data stores and perform checks between your source database and Spanner. We recommend using it to perform basic validations as a part of your migration, such as the following:

  - Check that all tables were created and that all schema mappings are correct.
  - Match the total number of rows for each table.
  - Extract random rows to verify consistency.
  - Validate your columns, for example, use `  count  ` , `  sum  ` , `  avg  ` , `  min  ` , `  max  ` , or `  group by  ` .
  - Compare any cyclic redundancy checks or hash functions at the row level.

To perform more specific validations, build custom checks during migration.
