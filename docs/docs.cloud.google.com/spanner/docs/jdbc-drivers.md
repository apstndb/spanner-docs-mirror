This page describes the JDBC drivers that Spanner supports for GoogleSQL-dialect databases and PostgreSQL-dialect databases. You can use these drivers to connect to Spanner from Java applications. Spanner supports the following JDBC drivers:

  - The Spanner JDBC driver, which is an open-source JDBC driver that is written, provided, and supported by Google, similar to the [Cloud Client Libraries](/spanner/docs/reference/libraries) . This is the recommended JDBC driver for Spanner, especially for the GoogleSQL dialect. You can also use it to connect to PostgreSQL-dialect databases.
  - The PostgreSQL JDBC driver in combination with PGAdapter. This driver only supports PostgreSQL-dialect databases.

<table>
<thead>
<tr class="header">
<th></th>
<th>Spanner JDBC driver</th>
<th>PostgreSQL JDBC driver</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Download</td>
<td><a href="https://search.maven.org/artifact/com.google.cloud/google-cloud-spanner-jdbc">Maven Central</a></td>
<td><a href="https://search.maven.org/artifact/org.postgresql/postgresql">Maven Central</a></td>
</tr>
<tr class="even">
<td>Written by</td>
<td>Google</td>
<td>PostgreSQL</td>
</tr>
<tr class="odd">
<td>Support</td>
<td>Google</td>
<td>Google</td>
</tr>
<tr class="even">
<td>Open source</td>
<td>Yes; Apache license</td>
<td>Yes; BSD 2-Clause</td>
</tr>
</tbody>
</table>

**Note:** Spanner previously supported the Simba JDBC driver. If you already use the Simba driver with Spanner, you can continue to do so.

For information about the Spanner JDBC driver, see [Spanner JDBC driver FAQ](/spanner/docs/open-source-jdbc) and [How to use the Spanner JDBC driver](/spanner/docs/use-oss-jdbc) .

For information about the PostgreSQL JDBC driver, see [Connect JDBC to a PostgreSQL-dialect database](/spanner/docs/pg-jdbc-connect) .
