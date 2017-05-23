# Designing Views, Inline Functions and Synonyms
## Lession 1: Views and Inline Functions
 
 ## Views
```
  CREATE VIEW Sales.OrderTotalsByYear
    WITH SCHEMABINDING
  AS
  SELECT 
    YEAR(o.orderdate) as orderyear
    , SUM(od.qty) as qty
    FROM Sales.Orders o
        JOIN Sales.OrderDetails as od
            on od.OrderId = o.OrderId
    GROUP BY YEAR(o.orderdate)
```
Read from a view just as you would a table ` select * from sales.ordertotalbyyear`

- Only one select statement can appear in the view (unless theres a union / union all, or other set option)
- Name cannot collide with any other object.
- You cannot add an order by to a view, sets are not ordered.
- - You can do an offset fetch, with an order by. Or a top with one, but the order of selecting from the view is not guaranteed beyond that which is required for the offset fetch.
- No Parameters, no variables
- Cannot create tables, temp or otherwise
- Cannot reference temp tables. 
 
### View Options
 - **ENCRYPTION** - obfuscates the view text
 - **SCHEMABINDING** - guarentees that the underlying tables cannot be modified without dropping the view.
 - **VIEW_METADATA** - returns the metadata of the viwew instead of the base data
 - **WITH CHECK** - For views with where clauses, wont let you update through the view in a way that will make the row fall out of the view.

### Indexed Views
 Normally, no data is stored, but if you put a clustered index on the view, then the results are persisted according to the clustered index. More in Chapter 15.

### Querying
WHen you query a view, the sql optimiser will unravel the view and optimize as if it was not there. The query plan will show the underlying tables accordinglt.

### Modifying Data Through a View
 - You can INSERT, UPDATE, DELETE through a view.
 - DML Tables must reference exactly 1 table at a time
 - The view columns must reference table columns directly and not expressions or aggregations
 - The view cannot have a union, cross join, except or intersect
 - The view cannot have any grouping (distinct, group by, having)
 - The view cannot have a TOP or OFFSET FETCH if it also defines WITH CHECK.

 That said, if you create an INSTEAD OF trigger, you can intercept the update to the view and funnel it off to the underlying tables. (Chapter 13)

### Partitioned Views
 Use views to partition (very) large tables across databases and servers. 
 Only use if you cannot partition the table, you instead manually shard the table and use the view to union the tables together. If they are on the same instance it's a local partitioned view otherwise it's a distributed partitioned view.

### Inline Functions
 Only way to filter a view is with a where clause.
 You can't pass it a parameter.
 You can use an inline table valued function wrapping the view to simulate parameterising a view.

 ```
 IF OBJECT_ID(N'Sales.fn_OrderTotalsByYear', N'IF') IS NOT NULL
  DROP FUNCTION Sales.fn_OrderTotalsByYear;
 GO
 
 CREATE FUNCTION Sales.fn_OrderTotalsByYear(@orderyear int)
 RETURNS TABLE
 AS
  RETURN (
    select orderyear, qty from Sales.OrderTotalsByYear
    where @orderyear = orderyear
  );
 GO
 ```
 **OPTIONS**
  * WITH ENCRYPTION
  * WITH SCHEMABINDING

## Lesson 2: Using Synonyms
 To create a synonym assign a name and specify the name of the database object it will be assigned to,.

 e.g. ` CREATE SYNONYM dbo.Categories FOR Production.Categories `
 e.g. ` DROP SYNONYM dbo.Categories `

#### Rules:
 * Synonyms do not store data or T-SQL code
 * Their names are T-SQL identifiers
 * Uses default schema unless otherwise specified
 * Doesn't test that the object exists...
 * ...Until synonym is used.
 * You can't create a synonym of a synonym.
 * You can't alter through a synonym.
 
#### Synonyms can be created for:
 - Tables
 - Views
 - User defined functions
 - Stored procedures
 - CLR assemblies

#### Synonyms as an Abstraction layer
 - You can use them to refer to objects in another database / on a linked server.
 - Only gives you simpler queries. I don't know if i love this really though.

Synonyms are late bound and there's no WITH SCHEMABINDING  