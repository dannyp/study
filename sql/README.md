#Study Guide
 9 / 10 people who know their stuff and fail because they don't pick the simplest correct answer.

## Types
 - Geometry vs Geography
 - Varbinary
 - DateTime2
 - 

##DML Triggers
 - Update tables to maintain integrity
 - Complex checks
 - Auditing
 - Types: instead of, after
 - SET nocount on
 - DML Triggers always part of operation (ie are rolled back)

## Views
### Restrictions
 - 1024 columns
 - Single query - no temp tables, table variables
 - Restricted data modification
 -- View can't modify data, updates through view can't update aggregated value
 - Top must include orderby

### Options
 - Schemabinding - can't modify tables underneath
 - Encryption
 - View metadata
 - Check - prevents modifications through view that would remove data from view.

## Functions
 - try\_convert, try\_parse returns null
 - parse(value AS type USING culture)
 - FORMAT(value, format, culture) - don't mermerise all fmt strings, just the major ones
 - iif(bool, true, false) "inline if"

## Ranking
 - Rank 
 - Dense rank (doesn't skip numbers)
 - Ntile: divides data into N chunks in n rows.
 - Row number

 ## Joins
  - Inner
  - Outer 'point at the table you want all the data from' eg from customers <- left join orders 
  - Full outer left and right
  - Cross

 ### Apply Operators
  - Join to a function, pass a parameter. Much better than subquery / doing it in the select.
  - Cross apply ~ 'inner join'
  - Outer apply ~ 'outer join'

