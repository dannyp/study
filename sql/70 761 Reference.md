## Date and Time Functions
### Getting the Current Date and Time
**GETDATE** -> T-SQL specificic. Current DATETIME of the Server
**CURRENT\_TIMESTAMP** -> Save value but ANSI standard.
**GETUTCDATE** -> GETDATE but returning UTC
**SYSDATETIME** -> Same as CURRENT\_TIMESTAMP but datetime2
**SYSUTCDATETIME** -> Same as SYSUTCDATETIME but utc
**SYSDATETIMEOFFSET** -> Same as SYSDATETIME but returning a datetimeoffset (inludes TZ offset from UTC).

### Date and Time Parts
DATEPART lets you extract a component from an input date value (e.g. YEAR, minute, nanosecond)
e.g. `DATEPART(month, '20170212')` returns 2.
The fuctions YEAR, MONTH, DAY are just common abbreviations of the above.

DATENAME is similar to datebard, but returning character strings
e.g. `DATENAME(month, '20170212')` reuturns February (culture specific so would return febbraio in italian)

Use the DATEFROMPARTS, DATETIME2FROMPARTS, DATETIMEFROMPARTS, DATETIMEOFFSETFROMPARTS, SMALLDATETIMEFROMPATS and TIMEFROMPARTS to constructdates. eg. `DATEFROMPARTS(2017, 02, 12)`

EOMONTH computes the date at the end of the current month specified by a date input parameter. eg. `EOMONTH(CURRENT_TIMESTAMP)` returns the last day of the current month.

dateadd - adding a certain number of dateparts to a date value.
datediff - calculating the number of dateparts between two date values.
switchoffset - adjusts a date to a different timezone, relative to UTC. e.g. `SWITCHOFFSET(SYSDATETIMEOFFSET(), '-8:00')` returns the current time in the -08:00 offset (US West Coast?)
todatetimeoffset - converts a datetime which is not offset aware and return a datetimeoffset for a particular offset value. e.g. `TODATETIMEOFFSET('08:00', CURRENT_TIMESTAMP)` returns current time and adds the 08:00 offset.
* Beware the need to change datetime offset around daylight savings.

* AT TIME ZONE - switch a date to one in a particular timeszone for use in a select statement. ` SELECT CURRENT_TIMEZONE AT TIME ZONE 'Pacific Standard Time'

## Temporal Table Queries.
Temporal tables make historical information point in time queries available.
This is configured on a per-table basis by using the SYSTEM_VERSIONING option to connect a regular table to a historical table.
At the current time, SQL server only supports system-versioned temporal tables. Which means that the system determines the values for the validity period.
This is distinct from application-time period tables which are part of the sql standard.

### Creating
Requirements
* Primary key
* Two datetime2 columns with a chosen precision to store the start and end of the validity period. (UTC)
* * Closed - open interval ie > Start and <= End
* Mark the start column GENERATED ALWAYS AS ROW START
* End column with GENERATED ALWAYS AS ROW END
* Designate the oclums with the clause PERIOD FOR SYSTEM\_TIME(startcol, endcol)
* Use the table option SYSTEM\_VERSIONING ON

A linked history table which can be created for you. 
The period start and end columns can be marked as HIDDEN if you like.

Turning an existing table into a temporal table
1. Add the window start and finish columns (they will need default constraints)
2. Then alter the cable to add the WITH SYSTEM_VERSIONING = ON ()
3. (optional) Drop the default constraints.

SQL server takes care of changes in the historical table when you alter the current table.

### Modifying Data
Modify data as normal, SQL takes care of the historical bits.
The thing to be aware of is that the start/end dates of the historical windows are the start of the current transaction

### Querying
`SELECT * FROM table FOR SYSTEM\_TIME [ AS OF | BETWEEN ... AND | FROM ... TO | CONTAINED IN | ALL] [DATE | RANGE]`

You also need to put the FOR SYSTEM_TIME on any joined tables (if you want their history)

#### AS OF
` SysStartTime <= date\_time AND SysEndTime > date\_time`
Tables with rows containing the values that were actual (current) at the point in time specified.
Internally implemented as a union between current and select for the above predicate.

#### FROM ... TO ... 
` SysStartTime < end\_date\_time AND SysEndDateTime > start\_date\_time`
Returns table with all valid versions of rows valid at some point between the period.
Internally executed as a union between the temporal table and its history table.
Note the greater-than and less-than and are not greater-or-equal-to or less-than-or-equal-to.
Note also that the rows do not need to have been closed before the end or opened after.

#### BETWEEN ... AND ...
The same as the FROM - TO but with <= and >=.

#### CONTAINED IN (,)
` SysStartTime >= startdate and SysEndTime <= enddate `
All row versions that were **opened and closed** within the date range

#### ALL
Union of history and current tables, no filtering.

## Query and Output JSON
Very similar to FOR XML, FOR JSON outputs the query results as a JSON string.

Functionality
 * Use root to control returned array name.
 * AUTO = flat
 * PATH = shape control, you can dot separate names to nest objects. Nesting arrays?
 * Other options INCLUDE\_NULL\_VALUES, WITHOUT\_ARRAY\_WRAPPER

## Convert JSON to Tabular
Just like shredding XMl to tables with OPENXML

- OPENJSON(jsonexpression, pathtoroot)
- Result = table (key string, value string, type tinyint)
- Type : { 0: null, 1: string, 2: number, 3 : boolean, 4 : boolean, 5 : object}
- Flattens nested objects into json strings.

You can define a schema for JSON using the WITH clause.
```
SELECT *
    FROM OPENJSON(@json)
    WITH (
        CustomerId int '$.Customer.Id',
        CustomerName NVARCHAR(20) '$.Customer.Name',
        Orders NVARCHAR(MAX) '$.Customer.Order' AS JSON
    )
```

You can also extract scalars from JSON using JSON\_VALUE or objects/arrays from JSON using JSON\_QUERY.

e.g:
```
 Select JSON_VALUE(@json, '$.Customer.Id') as CustomerId
    , json_value(@json, '$.Customer.Name') as CustomerName
    , json_query(@json, '$.Customer.Order') as Orders;
```

JSON_MODIFY in an update statement can be used to mutate json without writing the whole string again
```
SET @json = JSON_MODIFY(@json, '$.Customer.Name', 'New name'); -- update
SET @json = JSON_MODIFY(@json, '$.Customer.Id', null) --delete id
SET @json = JSON_modify(@json, '$.Customer.Newfield', "New vaklue') -- insert
```

There is no native JSON datatype. Json is stored as an nvarchar(max).
You can check for valid json by performing `SELECT ISJSON('str');`

# Implementing Data Types
## Choosing an Appropriate Data type
* One of the most important decisions you can make.
* The relational model gives great importance to enforcing data integrity. e.g. date columns only allow valid dates.
* Types are constraints, as are NOT NULLS.
* Formatting is distinct from type.
* Type encapsulates behaviour (e.g. '+' being addition or concatenation)
* Storage size depends on type, which then impacts the level of I/O and speed as a result.
* * INTS = 4 bytes but unneccessary if you only require few rows. Similarly, if your table might one day have a billion rows - then you cand use an int for a key.
* * Datetime = 8 bytes but DATE is only 3.

#### FLOAT and REAL
These are imprecise types, they are used for floading point data. Not all values can be represented exactly. Can represent very large or very small but at the price of precision.
e.g. Don't use a float for a barcode (but this is retarded for more reasons than that)

#### Fixed vs Dynamic
char, nchar, binary or varchar, nvarchar, varbinary

With fixed types, the rows don't need to expand when the value changes. Faster for values that update frequently. Variable types save significant amounts of space if the values vary widely.

## Choosing Data Types for Keys
Key data type choice can impact speed of access and inserts.
You also need to consider whether write performance is more important that read performance.

#### Options
* **Identity Column** Autogenerated numeric keys incrementing. Any numeric type like tinyint, smallint, int, bigint, numeric, decimal
* **Sequence object** Own object in the db from which you obdate new sequence values.
* **Nosequential GUIDs** Generated global identifier. 16 bytes. Can be generated by DB or by client.
* **Sequential GUID** Only allowed in column defaults. 
* **Custom solutions** Something weird you roll your own. Don't bother.

#### Considerations
Size - Bigger = slower reads, clustered index keys are stored in nonclustered index end nodes.

Sequential vs nonsequential - sequential results in less fragmentation which is better for read performance. Inserts can also be faster if a single session is inserting to a small data file (i.e. fits on a drive)

However, for high-end storage subsystems or OLTP loads - the situation can be different... i.e. lots of new orders coming in from many different threads.

Right-most page latch contention is when you have lots of things competing for the 'newdata' page.

Only an issue when using disk-based OLTP.

Nonsequential keys cause page splits and fragmentation. Avoids the 'hot spot' in the rightmost page. Alternately page splits and fragmentation can be avoided by index rebuilds.

GUIDs are portable but large. Other options are to use a regular sequence but flip a few bits to give it a constant distribution.

## Type conversion
- You may need to explicitly cast literals
- SQL Server usually converts literals with lowest tata type precedence in an operation between two different types.
- Otherwise, CAST, CONVERT, PARSE, TRY_*

## Handling Nulls
ISNULL vs COALESCE
- isnull only allows 2 args, coalesce >= 2
- coalesce is stardard sql
- return type of isnull is type of first argument or type of second, if first has no type, if bot null untyped - int.
- coalesce returns type based on precedence or error if no types.
- isnull is nonnullable if any input is nonnullable, coalesce only if all of them are.
- coalesce might execute a subquerry more than once.
- You need to handle nulls as a special case in joins, either 
    on a.key = b.key AND (a.key is null and b.key is null), or
    on coalesce(a.key, 'n/a') = coalesce(b.key, 'n/a')
- The latter will probably not result in an optimal plan
- You'll only get 1 NULL with a DISTINCT clause
- - Relevant for set operations (INTERSECT, UNION, EXCEPT)
