# Lesson 1: Filtering Data 
## Predicates, Three-Valued Logic and Search Arguments
 - A predicate is a logical expression that can evaluate to true of false.
 - When NULLs are present in the data, things get trickier. 
 -* Logical expressions where NULLs are present can have a 3rd value 'unknown'
 - WHERE returns values that evaluate to *true* only.
 - Even when doing <> queries.
 - Need to use IS and IS NOT when filter for NULLs explicitly.

 ### Performance
  - Filtering reduces rows in the query (networl traffic).
  - Option of using indexes, more efficient than a full scan.
  - Needs to be in the form of a *Search Argument* (SARG) to efficiently use indexes. See Ch. 15
  -- I.e. needs to be in the form Column Operator Value or Value operator column.
  -- Manipulating the column side of the predicate stops this.

  Eg. The following is *not* a SARG
  ```S
    select orderid from sales.orders 
    where coalesce(shipdate, '19000101') = coalesce(@shipdt, '19000101')
  ```
  Change the where clause to be:
  ```S
    select orderid from sales.orders 
    where shipdate = @shipdt OR (shipdate is null AND @shpdt is null)
  ```

  ### Combining Operators
   Use:
   1. NOT
   2. AND
   3. OR

   The above is also the order of precedence. I.e. `where col1 = 'w' and col2 = '3' or col1 = 'a' and col2 = '6'` is equivalent to `where (col1 = 'w' and col2 = '3') or (col1 = 'a' and col2 = '6')`. 
   * You need to use the parenthesis to control the order of operations if you need a different order.
   * THe SQL engine will try to short circuit however you shouldn't use this as an assumption to avoid invalid conversions.

   ## Filtering Character Data.
   * Literals have implicit types, you need to be careful that you're not getting the SQL engine to convert the column side of the predicate expression.
   
   ## Filtering Time and Date Data.
   * Dates are very locale sensitive.
   * Dependent on the language of the logon.
   * The `YYYY-MM-DD` and `YYYYMMDD` formats are considered language neutral.
   * Without a time part of a datetime, midnight is assumed. Use convert or cast.
   * * Affects indexes. You could use between or >= , < to achieve the same result in SARG format.

# Lesson 2: Sorting Data
## Understanding when Sort Order is Guaranteed
Sometimes you will consistently see that data is being returned in a particular order such as PK or clustered index order.

In reality, unless explicitly specified by the ORDER BY clause of the query, the sort order is not guaranteed.

Ordering is only deterministic so long as the column that you're ordering on has unique values.

Some Rules:
 * You can order by ordinal column values (not reccommended)
 * The ordered by column does not need to appear in the output
 * If DISTINCT is specified, then the ordered by column does neet to appear
 * Evaluate after the select, so you can use calculated or aliased columns.
 * NULLS come before non-NULLS under ASCENDING order.
 * T-SQL does not support NULLS FIRST / NULLS LAST.

# Lesson 3: Filtering with TOP and OFFSET FETCH.
## TOP (N) [PERCENT]
Filters the output to only return the first N rows, as specified by the argument to TOP. E.g. `SELECT TOP(10) ...`. 
The sort order is specified by the ORDER BY clause to the data.
Use a float for _N_ and the *PERCENT* keyword to return a percentage of the original dataset (if N has a fractional part to it, the PERCENT will perform a ceiling)

TOP() is not limited to a constant, you can pass it a parameters.

Only deterministic when ORDER BY is used.

Use WITH TIES option to show extra records which cannot be split (ie 2 equal dates)

WITH TIES will make the query deterministic, or you can use a unique ordering to break ties.

## OFFSET-FETCH
 * TOP() is not SQL Standard, OFFSET-FETCH is.
 * Can be used for pagination
 * Requires ORDER BY (appears after order by)
 * FETCH requires OFFSET (even if OFFSET is 0)
 * No percent or WITH TIES options.