# Combining Sets
## Using JOINs
### Cross JOIN
 * The cross join is the simplest time, also known as a Cartesian product
 * Yields: a row of output for each combination of left and right.
 * m * n rows.

I've only ever found a use when computing some superset of 2 smaller ones. I.e. you've got a list of days and a list of employees and you're seeing which days they did/didn't work on. That or data-generation of a key matrix n*m.

### Inner JOIN
 Matches rows from 2 relations based on a predicate.
 Usually comparison of FK to PK equality.
 Rows from *either* side that don't find a match in the other are discarded.

 Note that when you create a PK or unique constraint, SQL server automatically creates an index, but when you create a FK, no such index is created even though it may improve performance. This is a good thing to look for when it comes to query tuning.
 
 Futher filtering can be done in the ON or WHERE clauses, logcially this has the same effect. In performance execution terms, expect the SQL query engine to work it out to be equivalent too.

 X INNER JOIN Y is also equivalent to Y INNER JOIN X.

### Outer JOIN
 Has the ability to preserve rows that do not match, either from the LHS (LEFT OUTER), the RHS (RIGHT OUTER) or both (FULL OUTER)

 In *OUTER JOINs* the ON and WHERE clauses play very different roles. Think of ON as performing the matching and WHERE performing the filtering. In this case, the WHERE predicate has the ability to discard the 'parent' side rows as well as the joined side (i.e. the left side in a LEFT JOIN and the RIGH SIDE in a right join).

## Subqueries, Table Expressions and the Apply Operator
### Subqueries
 - Subqueries can either be self contained or reference elements of the outer query.
 - Self contained subqueries can be run independently.
 - They can return tables, scalars, or multiple values.
 - - Use x = `(SELECT... )` for a scalar, or `x IN (SELECT...)` for multiple
 - - Use EXISTS to return true if the subquery produces 1 or more rows.
 - - - The select list is ignored by the optimizer.

 ### Table Expressions
 - Derived tables, CTEs, Views, Inline Table-valued functions.
 - - Derived tables and CTEs are only visible to the statement that defines them.
 - Table expressions must return relational results (no ordering, all columns uniquely named)
 - The optimizer un-nests table expressions, they have no performance implication beyond the use of tables. (Hmmm....)

 #### Derived tables
 - Subquery that returns an entire table
 ```S
    SELECT * FROM (SELECT ROW_NUMBER() OVER(ORDER BY Col1) FROM Tbl) t;
 ```
  - Column names can be given inline (`SELECT x as Column1...`) or externally (`(SELECT x.... ) as t(Column1)`)
  - Issue: nesting gets messy and is the only way to refer to 1 drived table from another query.
  - Issue: Requires duplication of code to join multiple instances of derived tables.

 #### Common Table Expressions
 - Just a derived table with a name.
    ```S
    WITH CTE_Name(Col1, Col2) AS (SELECT x, y ...) 
        SELECT * FROM CTE_Name;
    ```
 - No need to nest, just define and comma-separate.
 - Can refer to one another.
 - Recursive form available
    ```
    With RecursiveCTE as (
        Select x from table where id = 1 --base case
        union all
        select x from table as  b  
            join RecursiveCTE as r on b.newx = r.x --recursive step.
    )
    Select * from RecursiveCTE
    ```
 
 #### Views and Inline Table-Valued Functions.
 - Reusable derived tables.
 - Scoped outide of statement
 - Table valued functions are views that accept a parameter
 - Treat a view like a table in terms of joins
 - You need to join a table-valued fuunction with CROSS apply (inner join) and OUTER APPLY (left outer join.)

 ### Combining Sets
  - UNION Unifies the results of 2 sets.
  - UNION ALL same as above but includes duplicate rows.
  - INTERSECT only rows in both
  - EXCEPT only rows in the first that are not in the 2nd.