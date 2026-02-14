Mode in which the statement must be processed.

Enums

`  NORMAL  `

The default mode. Only the statement results are returned.

`  PLAN  `

This mode returns only the query plan, without any results or execution statistics information.

`  PROFILE  `

This mode returns the query plan, overall execution statistics, operator level execution statistics along with the results. This has a performance overhead compared to the other modes. It isn't recommended to use this mode for production traffic.

`  WITH_STATS  `

This mode returns the overall (but not operator-level) execution statistics along with the results.

`  WITH_PLAN_AND_STATS  `

This mode returns the query plan, overall (but not operator-level) execution statistics along with the results.
