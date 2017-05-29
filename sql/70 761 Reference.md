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

