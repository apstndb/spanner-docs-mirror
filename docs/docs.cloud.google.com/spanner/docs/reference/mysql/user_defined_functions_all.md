GoogleSQL for Spanner supports the following MySQL functions. You need to install the MySQL functions in your Spanner database before you can use them. For more information on installing the functions, see [Install MySQL functions](/spanner/docs/install-mysql-functions) .

## MySQL function list

<table>
<thead>
<tr class="header">
<th>Name</th>
<th>Summary</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#bin_to_uuid"><code dir="ltr" translate="no">        mysql.BIN_TO_UUID       </code></a></td>
<td>Converts a binary UUID representation to a STRING representation.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#bit_length"><code dir="ltr" translate="no">        mysql.BIT_LENGTH       </code></a></td>
<td>Returns the length of string in bits.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#char"><code dir="ltr" translate="no">        mysql.CHAR       </code></a></td>
<td>Interprets the input parameter as an integer and returns a byte string consisting of the character given by the code value of that integer.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#concat_ws"><code dir="ltr" translate="no">        mysql.CONCAT_WS       </code></a></td>
<td>Returns a concatenated string with separators.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#date_format"><code dir="ltr" translate="no">        mysql.DATE_FORMAT       </code></a></td>
<td>Formats a date as specified.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/timestamp_functions#datediff"><code dir="ltr" translate="no">        mysql.DATEDIFF       </code></a></td>
<td>Subtracts two dates and returns the number of days between them.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#day"><code dir="ltr" translate="no">        mysql.DAY       </code></a></td>
<td>Alias for <code dir="ltr" translate="no">       DAYOFMONTH      </code> . Returns the day of the month (1-31) from a <code dir="ltr" translate="no">       TIMESTAMP      </code> value.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#dayname"><code dir="ltr" translate="no">        mysql.DAYNAME       </code></a></td>
<td>Returns the name of the weekday.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#dayofmonth"><code dir="ltr" translate="no">        mysql.DAYOFMONTH       </code></a></td>
<td>Returns the day of the month (1-31).</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#dayofweek"><code dir="ltr" translate="no">        mysql.DAYOFWEEK       </code></a></td>
<td>Returns the weekday index of the input parameter.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#dayofyear"><code dir="ltr" translate="no">        mysql.DAYOFYEAR       </code></a></td>
<td>Returns the day of the year (1-366).</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/numeric_functions#degrees"><code dir="ltr" translate="no">        mysql.DEGREES       </code></a></td>
<td>Converts radians to degrees.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#from_days"><code dir="ltr" translate="no">        mysql.FROM_DAYS       </code></a></td>
<td>Converts a day number to a date.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#from_unixtime"><code dir="ltr" translate="no">        mysql.FROM_UNIXTIME       </code></a></td>
<td>Formats a Unix timestamp as a date.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#hex"><code dir="ltr" translate="no">        mysql.HEX       </code></a></td>
<td>Returns the hexadecimal representation of a string.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#hour"><code dir="ltr" translate="no">        mysql.HOUR       </code></a></td>
<td>Returns the hour (0-23).</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#inet_aton"><code dir="ltr" translate="no">        mysql.INET_ATON       </code></a></td>
<td>Returns the numeric value of an IP address.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#inet_ntoa"><code dir="ltr" translate="no">        mysql.INET_NTOA       </code></a></td>
<td>Returns the IP address from a numeric value.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#inet6_aton"><code dir="ltr" translate="no">        mysql.INET6_ATON       </code></a></td>
<td>Returns the numeric value of an IPv6 address.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#inet6_ntoa"><code dir="ltr" translate="no">        mysql.INET6_NTOA       </code></a></td>
<td>Returns the IPv6 address from a numeric value.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#insert"><code dir="ltr" translate="no">        mysql.INSERT       </code></a></td>
<td>Inserts a substring at a specified position up to a specified number of characters.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#is_ipv4"><code dir="ltr" translate="no">        mysql.IS_IPV4       </code></a></td>
<td>Checks whether the input parameter is an IPv4 address.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#is_ipv4_compat"><code dir="ltr" translate="no">        mysql.IS_IPV4_COMPAT       </code></a></td>
<td>Returns whether the input parameter is an IPv4-compatible address.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#is_ipv4_mapped"><code dir="ltr" translate="no">        mysql.IS_IPV4_MAPPED       </code></a></td>
<td>Returns whether the input parameter is an IPv4-mapped address.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#is_ipv6"><code dir="ltr" translate="no">        mysql.IS_IPV6       </code></a></td>
<td>Returns whether the input parameter is an IPv6 address.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#is_uuid"><code dir="ltr" translate="no">        mysql.IS_UUID       </code></a></td>
<td>Returns whether the input parameter is a valid UUID. Not available in MySQL 5.7.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/json_functions#json_quote"><code dir="ltr" translate="no">        mysql.JSON_QUOTE       </code></a></td>
<td>Quotes the JSON document.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/json_functions#json_unquote"><code dir="ltr" translate="no">        mysql.JSON_UNQUOTE       </code></a></td>
<td>Unquotes the JSON value.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/timestamp_functions#localtime"><code dir="ltr" translate="no">        mysql.LOCALTIME       </code></a></td>
<td>Alias for <code dir="ltr" translate="no">       NOW      </code> . Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> for when the query statement containing this function started to run.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/timestamp_functions#localtimestamp"><code dir="ltr" translate="no">        mysql.LOCALTIMESTAMP       </code></a></td>
<td>Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> when the query statement containing this function started to run.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#locate"><code dir="ltr" translate="no">        mysql.LOCATE       </code></a></td>
<td>Returns the position of the first occurrence of a substring. The search is case insensitive.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/numeric_functions#log2"><code dir="ltr" translate="no">        mysql.LOG2       </code></a></td>
<td>Returns the base-2 logarithm of the input argument. Returns <code dir="ltr" translate="no">       NULL      </code> if the input argument is out of range.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#makedate"><code dir="ltr" translate="no">        mysql.MAKEDATE       </code></a></td>
<td>Creates a date from the year and day of year.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#microsecond"><code dir="ltr" translate="no">        mysql.MICROSECOND       </code></a></td>
<td>Returns the microseconds from the input parameter.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#mid"><code dir="ltr" translate="no">        mysql.MID       </code></a></td>
<td>Alias for <a href="/spanner/docs/reference/standard-sql/string_functions#substring"><code dir="ltr" translate="no">        SUBSTRING       </code></a> .</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#minute"><code dir="ltr" translate="no">        mysql.MINUTE       </code></a></td>
<td>Returns the minute from the input parameter.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#month"><code dir="ltr" translate="no">        mysql.MONTH       </code></a></td>
<td>Returns the month from the date passed.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#monthname"><code dir="ltr" translate="no">        mysql.MONTHNAME       </code></a></td>
<td>Returns the name of the month.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/timestamp_functions#now"><code dir="ltr" translate="no">        mysql.NOW       </code></a></td>
<td>Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> when the query statement containing this function started to run.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#oct"><code dir="ltr" translate="no">        mysql.OCT       </code></a></td>
<td>Returns a string containing an octal representation of a number.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#ord"><code dir="ltr" translate="no">        mysql.ORD       </code></a></td>
<td>Returns the character code for the leftmost character of the input parameter.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#period_add"><code dir="ltr" translate="no">        mysql.PERIOD_ADD       </code></a></td>
<td>Adds a period to a year-month.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#period_diff"><code dir="ltr" translate="no">        mysql.PERIOD_DIFF       </code></a></td>
<td>Returns the number of months between periods.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/numeric_functions#pi"><code dir="ltr" translate="no">        mysql.PI       </code></a></td>
<td>Returns the value of pi.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#position"><code dir="ltr" translate="no">        mysql.POSITION       </code></a></td>
<td>Alias for LOCATE. Returns the starting position of the first occurrence of the substring.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#quarter"><code dir="ltr" translate="no">        mysql.QUARTER       </code></a></td>
<td>Returns the quarter from a date input parameter.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#quote"><code dir="ltr" translate="no">        mysql.QUOTE       </code></a></td>
<td>Escapes the input parameter for use in a SQL statement.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/numeric_functions#radians"><code dir="ltr" translate="no">        mysql.RADIANS       </code></a></td>
<td>Converts degrees to radians.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#regexp_like"><code dir="ltr" translate="no">        mysql.REGEXP_LIKE       </code></a></td>
<td>Returns <code dir="ltr" translate="no">       TRUE      </code> if the string matches a regular expression. Not available in MySQL 5.7.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#regexp_substr"><code dir="ltr" translate="no">        mysql.REGEXP_SUBSTR       </code></a></td>
<td>Returns a substring matching a regular expression. Not available in MySQL 5.7.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#second"><code dir="ltr" translate="no">        mysql.SECOND       </code></a></td>
<td>Returns the second (0-59).</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/encryption_compression_functions#sha2"><code dir="ltr" translate="no">        mysql.SHA2       </code></a></td>
<td>Calculates a SHA-2 checksum.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#space"><code dir="ltr" translate="no">        mysql.SPACE       </code></a></td>
<td>Returns a string of the specified number of spaces.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#str_to_date"><code dir="ltr" translate="no">        mysql.STR_TO_DATE       </code></a></td>
<td>Converts a string to a date.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/string_functions#strcmp"><code dir="ltr" translate="no">        mysql.STRCMP       </code></a></td>
<td>Compares two strings.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#substring_index"><code dir="ltr" translate="no">        mysql.SUBSTRING_INDEX       </code></a></td>
<td>Returns a substring from a string before the specified number of occurrences of the delimiter. Performs a case-sensitive match when searching for the delimiter.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#sysdate"><code dir="ltr" translate="no">        mysql.SYSDATE       </code></a></td>
<td>Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> when the query statement containing this function started to run.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#time"><code dir="ltr" translate="no">        mysql.TIME       </code></a></td>
<td>Extracts the time portion of the expression passed.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#to_days"><code dir="ltr" translate="no">        mysql.TO_DAYS       </code></a></td>
<td>Returns the date input parameter converted to days.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#to_seconds"><code dir="ltr" translate="no">        mysql.TO_SECONDS       </code></a></td>
<td>Returns the date or datetime input parameter converted to seconds since Year 0.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/numeric_functions#truncate"><code dir="ltr" translate="no">        mysql.TRUNCATE       </code></a></td>
<td>Truncates a number to a specified number of decimal places.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/string_functions#unhex"><code dir="ltr" translate="no">        mysql.UNHEX       </code></a></td>
<td>Returns a string containing the hex representation of a number.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#unix_timestamp"><code dir="ltr" translate="no">        mysql.UNIX_TIMESTAMP       </code></a></td>
<td>Returns a Unix timestamp.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#utc_date"><code dir="ltr" translate="no">        mysql.UTC_DATE       </code></a></td>
<td>Returns the current UTC date.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#utc_timestamp"><code dir="ltr" translate="no">        mysql.UTC_TIMESTAMP       </code></a></td>
<td>Returns the <code dir="ltr" translate="no">       TIMESTAMP      </code> when the query statement containing this function started to run.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/utility_functions#uuid"><code dir="ltr" translate="no">        mysql.UUID       </code></a></td>
<td>Returns a Universal Unique Identifier (UUID).</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/utility_functions#uuid_to_bin"><code dir="ltr" translate="no">        mysql.UUID_TO_BIN       </code></a></td>
<td>Converts a string representation of a UUID to a binary representation.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#week"><code dir="ltr" translate="no">        mysql.WEEK       </code></a></td>
<td>Returns the week number.</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#weekday"><code dir="ltr" translate="no">        mysql.WEEKDAY       </code></a></td>
<td>Returns the weekday index.</td>
</tr>
<tr class="even">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#weekofyear"><code dir="ltr" translate="no">        mysql.WEEKOFYEAR       </code></a></td>
<td>Returns the calendar week of the date (1-53).</td>
</tr>
<tr class="odd">
<td><a href="/spanner/docs/reference/mysql/date_time_functions#year"><code dir="ltr" translate="no">        mysql.YEAR       </code></a></td>
<td>Returns the year.</td>
</tr>
</tbody>
</table>
