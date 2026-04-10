GoogleSQL for Spanner supports statistical aggregate functions. To learn about the syntax for aggregate function calls, see [Aggregate function calls](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate-function-calls) .

## Function list

| Name                                                                                                                                            | Summary                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| [`         STDDEV        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev)           | An alias of the `        STDDEV_SAMP       ` function.           |
| [`         STDDEV_SAMP        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev_samp) | Computes the sample (unbiased) standard deviation of the values. |
| [`         VAR_SAMP        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#var_samp)       | Computes the sample (unbiased) variance of the values.           |
| [`         VARIANCE        `](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#variance)       | An alias of `        VAR_SAMP       ` .                          |

## `     STDDEV    `

    STDDEV(
      [ DISTINCT ]
      expression
      [ HAVING { MAX | MIN } having_expression ]
    )

**Description**

An alias of [STDDEV\_SAMP](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#stddev_samp) .

## `     STDDEV_SAMP    `

    STDDEV_SAMP(
      [ DISTINCT ]
      expression
      [ HAVING { MAX | MIN } having_expression ]
    )

**Description**

Returns the sample (unbiased) standard deviation of the values. The return result is between `  0  ` and `  +Inf  ` .

All numeric types are supported. If the input is `  NUMERIC  ` then the internal aggregation is stable with the final output converted to a `  FLOAT64  ` . Otherwise the input is converted to a `  FLOAT64  ` before aggregation, resulting in a potentially unstable result.

This function ignores any `  NULL  ` inputs. If there are fewer than two non- `  NULL  ` inputs, this function returns `  NULL  ` .

`  NaN  ` is produced if:

  - Any input value is `  NaN  `
  - Any input value is positive infinity or negative infinity.

To learn more about the optional aggregate clauses that you can pass into this function, see [Aggregate function calls](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate-function-calls) .

**Return Data Type**

`  FLOAT64  `

**Examples**

    SELECT STDDEV_SAMP(x) AS results FROM UNNEST([10, 14, 18]) AS x
    
    /*---------+
     | results |
     +---------+
     | 4       |
     +---------*/

    SELECT STDDEV_SAMP(x) AS results FROM UNNEST([10, 14, NULL]) AS x
    
    /*--------------------+
     | results            |
     +--------------------+
     | 2.8284271247461903 |
     +--------------------*/

    SELECT STDDEV_SAMP(x) AS results FROM UNNEST([10, NULL]) AS x
    
    /*---------+
     | results |
     +---------+
     | NULL    |
     +---------*/

    SELECT STDDEV_SAMP(x) AS results FROM UNNEST([NULL]) AS x
    
    /*---------+
     | results |
     +---------+
     | NULL    |
     +---------*/

    SELECT STDDEV_SAMP(x) AS results FROM UNNEST([10, 14, CAST('Infinity' as FLOAT64)]) AS x
    
    /*---------+
     | results |
     +---------+
     | NaN     |
     +---------*/

## `     VAR_SAMP    `

    VAR_SAMP(
      [ DISTINCT ]
      expression
      [ HAVING { MAX | MIN } having_expression ]
    )

**Description**

Returns the sample (unbiased) variance of the values. The return result is between `  0  ` and `  +Inf  ` .

All numeric types are supported. If the input is `  NUMERIC  ` then the internal aggregation is stable with the final output converted to a `  FLOAT64  ` . Otherwise the input is converted to a `  FLOAT64  ` before aggregation, resulting in a potentially unstable result.

This function ignores any `  NULL  ` inputs. If there are fewer than two non- `  NULL  ` inputs, this function returns `  NULL  ` .

`  NaN  ` is produced if:

  - Any input value is `  NaN  `
  - Any input value is positive infinity or negative infinity.

To learn more about the optional aggregate clauses that you can pass into this function, see [Aggregate function calls](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/aggregate-function-calls) .

**Return Data Type**

`  FLOAT64  `

**Examples**

    SELECT VAR_SAMP(x) AS results FROM UNNEST([10, 14, 18]) AS x
    
    /*---------+
     | results |
     +---------+
     | 16      |
     +---------*/

    SELECT VAR_SAMP(x) AS results FROM UNNEST([10, 14, NULL]) AS x
    
    /*---------+
     | results |
     +---------+
     | 8       |
     +---------*/

    SELECT VAR_SAMP(x) AS results FROM UNNEST([10, NULL]) AS x
    
    /*---------+
     | results |
     +---------+
     | NULL    |
     +---------*/

    SELECT VAR_SAMP(x) AS results FROM UNNEST([NULL]) AS x
    
    /*---------+
     | results |
     +---------+
     | NULL    |
     +---------*/

    SELECT VAR_SAMP(x) AS results FROM UNNEST([10, 14, CAST('Infinity' as FLOAT64)]) AS x
    
    /*---------+
     | results |
     +---------+
     | NaN     |
     +---------*/

## `     VARIANCE    `

    VARIANCE(
      [ DISTINCT ]
      expression
      [ HAVING { MAX | MIN } having_expression ]
    )

**Description**

An alias of [VAR\_SAMP](https://docs.cloud.google.com/spanner/docs/reference/standard-sql/statistical_aggregate_functions#var_samp) .
