# Study Guide
 9 / 10 people who know their stuff and fail because they don't pick the simplest correct answer.
 *Question format*: You have... You need.... What should you do....
 All answers exist (check this)
 Look for users that are obviously wrong.
 
### Exam Tips
 - [ ] Set date, set goals along the way
 - [ ] Keep an eye on time, don't panic
 - [ ] Eliminate wrong answers, guess if not sure (no points deducted)

## Types
 - [ ] Geometry vs Geography
 - [ ] Varbinary
 - [ ] DateTime2

## DML Triggers
 - [ ] Update tables to maintain integrity
 - [ ] Complex checks
 - [ ] Auditing
 - [ ] Types: instead of, after
 - [ ] SET nocount on
 - [ ] DML Triggers always part of operation (ie are rolled back)

## Views
### Restrictions
 - [ ] 1024 columns
 - [ ] Single query - [ ] no temp tables, table variables
 - [ ] Restricted data modification
 -- [ ] View can't modify data, updates through view can't update aggregated value
 - [ ] Top must include orderby

### Options
 - [ ] Schemabinding - [ ] can't modify tables underneath
 - [ ] Encryption
 - [ ] View metadata
 - [ ] Check - [ ] prevents modifications through view that would remove data from view.

## Functions
 - [ ] try\_convert, try\_parse returns null
 - [ ] parse(value AS type USING culture)
 - [ ] FORMAT(value, format, culture) - [ ] don't mermerise all fmt strings, just the major ones
 - [ ] iif(bool, true, false) "inline if"

## Ranking
 - [ ] Rank 
 - [ ] Dense rank (doesn't skip numbers)
 - [ ] Ntile: divides data into N chunks in n rows.
 - [ ] Row number

 ## Joins
  - [ ] Inner
  - [ ] Outer 'point at the table you want all the data from' eg from customers <- [ ] left join orders 
  - [ ] Full outer left and right
  - [ ] Cross

 ### Apply Operators
  - [ ] Join to a function, pass a parameter. Much better than subquery / doing it in the select.
  - [ ] Cross apply ~ 'inner join'
  - [ ] Outer apply ~ 'outer join'

 ## CTE Common Table Expressions
  - [ ] 'Inline views'
  - [ ] Lets the compiler work out the best way to structure query
  - [ ] Better structure too.

 ## Handling Nulls
  - [ ] NULL != NULL
  - [ ] Must use is not null
  - [ ] CASE WHEN IS NOT
  - [ ] COALESCE

 ## Pivot
  - [ ] Not dynamic, you need to tell it which columns to create.
  SELECT * FROM SalesData
    PIVOT (Sum(SalesAmount) FOR SalesYear IN ([2001], [2002])) AS Pvt

 ## Rollup and Cube
  - [ ] 
    SELECT Region, Country, Category, Sum(TotalSold) as TotalSales FROM Sales.SalesInformation
        -- [ ] then
        GROUP BY ROLLUP (Region, Country, Category) -- [ ] Aggregate for Region, then Country, Then Category (with nulls to denote "All")
        -- [ ] or
        GROUP BY CUBE ( Region, Country, Category) -- [ ] Aggregate for all combinations of these cols.
        -- [ ] or 
        GROUP BY GROUPING SETS ( ( Region, Country, Category ), (Region, County)) -- [ ] restrict the output sets. 
  - [ ] Use grouping ID to tell when a column is a null or is just being excuded from the group by for a row.

 ## FOR XML
 ### RAW
  - [ ] Gives you the word 'row' for every row and puts values as attributes
  - [ ] You can rename rows with parameters to RAW, eg. RAW('Contact') 
  - [ ] You can give the root element a name with ROOT('Root')
  - [ ] Use 'Elements' to give values their own node each

 ### Auto
  - [ ] Same as raw but use the name of the table as the wrapper
 
 ### Explitit
  - [ ] You specify for each column whether it is an element or an attribute and where it appears in the output.
  - [ ] E.g. , ProductID as [Product!1!ID] <Product ID=''>.
  - [ ] Use !ELEMENT to specify element.

 ### Path
  - [ ] Used to control heirarchy

 ### Heirarchies
  - [ ] The order of tables in joins will inform where they appear in the heirarchy unless controlled by path
  - [ ] However this can lead to duplication of wrapper nodes
  - [ ] You need to use an inner SELECT in the projection of the Wrapper "FOR XML"

 ## Fetch and Offset
  - [ ] OFFSET BY X ROWS 
  - [ ] FETCH NEXT Y ROWS ONLY;

 ## Stored procedure
  - [ ] Encryption **
  - [ ] Recompile
  - [ ] execute as :
   -- [ ] Owner
   -- [ ] Self = whomever executes the ALTER
   -- [ ] Caller = caller of the sproc
   -- [ ] 'user'

 ## Update Statement
  - [ ] When doing JOINs - [ ] you have to list it twice, both in the update and the join

 ## Merge
    MERGE Target AS T
    USING Soure AS S
    WHEN NOT MATCHED BY TARGET -- [ ] Insert
    WHEN MATCHED -- [ ] UPDATE
    WHEN NOT MATCHED ON SOURCE -- [ ] Delete
    OUTPUT $action  -- [ ] what'd we do to the Target

 ## JOIN TYPES
    - [ ] NESTED LOOP
    -- [ ] Small joins
    - [ ] MERGE
    -- [ ] Large, sorted data
    - [ ] HASH
    -- [ ] Large, unsorted data
    If you're looking at the execution plan and wondering where the problem is - [ ] its the hash join.
 
 ## Transactions
  - [ ] BEGIN, COMMIT, ROLLBACK, 
  - [ ] SAVE (a point you can ROLLBACK to)
  - [ ] You want to know the ISOLATION levels
  -- [ ] READ COMMITTED - [ ] no dirty reads, locks on select (the default).
  -- [ ] READ UNCOMMITTED - [ ] dirty data, no locks. You'll even see stuff that gets rolled back.
  -- [ ] REPEATABLE READ - [ ] Always see the data you've already seen. Select blocks updates but not inserts. Row level locks.
  -- [ ] SERIALIZABLE - [ ] Guarentees that read data is locked, prevents new data coming in. Blocks inserts as well as updates. If no index, will cause table lock.
  -- [ ] READ COMMITED SNAPSHOT (If snapshot isolation is allowed at db level) query sandboxes - [ ] tempdb overhead.

 ## Try / Catch
   - [ ] No finally
   - [ ] Nothing allowed between end try and begin catch
   
 ## Cursors
   - [ ] Row by agonising row
   - [ ] Steps
   -- [ ] declare, open, fetch, create, use, close, deallocate.

 ## Hints
 ### Query Hints
   - [ ] Don't look at every query hint
   - [ ] OPTION (OPTIMIZE..)
 ### Table Hints
   - [ ] WITH (TABLOCK)


