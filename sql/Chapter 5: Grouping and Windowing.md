# Grouping and Windowing
## Lesson 1: Group BY
 - Group BY allows you to specify columns to group the output so that only 1 row is produced for each unique value.
 - The HAVING clause performs predicate based filtering on the grouped data.
 - The columns which are not being grouped by need to be aggregated
 - Available aggregations include min, max, sum, avg, count.
 - Count ignores nulls
 - The DISTINCT option is available to the aggregations.
 - The one-row'ness of each non-grouped column must be enforced by the query, not just the data. But just use the aggregations like identity functions, eg sum, min, max, avg.
 - 

### Multiple Grouping Sets.
Group in different ways within the same query

#### GROUPING SETS
 - List all the grouping sets you want to define in your output.
 - Empty = same as no group by
 - Where a row would have multiple records (i.e. it isn't part of the grouping set), null is displayed.
 
#### Cube 
 - Shortcut to grouping set that defines all possible combinations of groups within a grouping set
 - eg: `GROUP BY CUBE (X,  Y)` is equivalent to `GROUP BY GROUPING SETS ((X, Y), (X), (Y), ())`

#### Rollup
 - Cube does not enforce a heirarchy, you get every combo.
 - Rollup maintains the grouping order defined in the original set
 - eg. `GROUP BY ROLLUP (X, Y, Z)` <=> `GROUP BY GROUPING SETS (( X, Y, Z), (X, Y), X, ())`
 
Grouping sets return nulls when a column is not part of a grouping set. So what do we do if the underlying column naturally contains nulls? There's the GROUPING_ID and the GROUPING functions.

## Lesson 2: Pivoting and Unpivoting
### Pivot 
 Pivoting is a technique that groups and aggregates but also tranitions data from a state of rows to columns.

 In _all_ pivot queries you need to define 3 elements
  * What do you want to see *on rows* (on rows | grouping element)
  * What donyou want to see *on columns* (on cols | spreading element)
  * What do you want to see in the *intersection* (data, aggregation element)

  *General Form:*
  ```
    WITH PivotData AS (
        SEELCT 
            <grouping column>,
            <spreading column>,
            <aggreagation column>
            FROM <source table>
    )
        SELECT <cols, ...>
            FROM PivotData
            PIVOT (<aggregate>(aggregation column)
                FOR <spreading> IN (<list of distinct spreading values)) AS p;
  ```
  #### Why define the PivotData with only 3 columns though?
  Well, the grouping column is identified by elimination, so you will get grouped rows for everything that's not an aggregate or spreading column.

  ### Limitations of Pivot
   - The aggregation and spreading elements cannot be expressinos
   - - You'd need to do calculation in the CTE
   - You can't use COUNT(*), it has to be one of the columns
   - You can only have one aggregate function
   - You must supply a static list to the IN operator.

### Unpivot
  Consider unpivoting the inverse of pivoting. 
  Rotate from columns to rows
  Table operator that you use in the from clause.

  In _unpivot operations_ you need the following 3 elements
  * Set of source columns
  * The name you want to assign to the target values column
  * The names you want to assign to the target names column

  *General Form:*
  ```
    SELECT <column list>, <names column>, <values column>
        FROM <source table>
        UNPIVOT (<values column> FOR <names column> IN (<source columns>)) as U;
  ```

## Lesson 3: Using Window Functions
 ALso used for data analysis computations. Like grouping functions different in how you define the set of rows for the function to work with.
 With window functions you define the set of rows for the function to work with using 'over'

 There are 3 types of windowing functions, aggregate, anf offset.

 ### Window Aggregate Functions.
 - The same as GROUP aggregates, but applied to a window of rows as defined by the OVER clause.
 - They do not exclude the detail rows. You can mix detail and aggregated info.
 - OVER() empty parenthesis means the whole set
 - Use PARTITION BY to define a window partition clause to restrict the window.
 - - eg. `SUM(val) OVER(PARTITION BY OperatorID)` computes the sum for the current Operator
 - - eg. `100 * ( SUM(value) over (partition by orderid))` computes the percentage each row's value takes up out of the whole order.

 Window aggregate functions also allow _framing_. You define an order within the partition, and based on this select a moving frame within the data. 
 You can specify the following options for your window frame delimiters:
 * UNBOUNDED PRECEEDING or UNBOUNDED FOLLOWING means from the beginning or end of the partition (ie running total)
 * CURRENT ROW
 * (n rows) PRECEEDING or FOLLOWING

Using this for running computations is much more performant than tradidional (self joins, subqueries, group aggregates) techniques. 
UNBOUNDED PRECEDING is the best from an optimisation standpoint
They are only allowed in the select and order by clauses.
You need to use a table expression to predicate filter on a window function's result.
Where ROWS gives you the aggregation based on the physical rows, RANGE aggregates logically - ie it will treat rows with equal sort column values as 'one' row from the point of view of its aggregation. It's way slower and limited in its implementation in SQL server. It only supports UNBOUNDED PRECEEDING and CURRENT row for the delimiters.

### Window Ranking Function
 Ranks fows within partition based on the specified ordering. As with the other windowing functions, if you don't specify an ordering, it runs over the whole set.
 The Ordering clause is mandatory (otherwise what are they doing!?)
 Ranking functions do not support framing.
 
 _Supported Ranking Functions in T-SQL:_
 * ROW_NUMBER
 * - Computes a unique sequential integer starting at 1 within the window.
 * - Determinism dependent on the ORDER BY 
 * RANK & DENSE_RANK
 * - Assign the same ordering value to all rows that share the same value for ORDER BY
 * - Dense rank will use every integer, rank will skip where rows are repeated.
 * NTILE
 * - Grouping into predefined number of percentile bucket.

 ### Window Offset Functions
  Return an element from a single row that is in a given offset from the current row in the partition, or from the first and last row in the frame.
  
  _Supported offset functions in T-SQL_
 * LAG(col, n) and LEAD(col, n)
 * - Return the value of 'col' coming from n rows ahead or behind the current (default to 1)
 * FIRST_VALUE, LAST_VALUE
 * - Returns the value of col from the first or last value in the window.

 #### A note on performance of range.
 The default frame (when applicable) is between unbounded preceeding and current row. 
 It is receommended to avoid 'RANGE'
 You need to be explicit with 'ROWS' to account for this.
 Note that when using LAST_VALUE, you should alwyas specify UNBOUNDED FOLLOWING otherwise it will just default to the current row.

 