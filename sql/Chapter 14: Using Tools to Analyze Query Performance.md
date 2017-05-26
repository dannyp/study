# Chapter 14: Using Tools to Analyze Query Performance
## Lesson 1: Getting Started with Query Optimization
### Optimisation Problems and the Query Optimiser
Anything that isn't a very simple query can be executed in many different ways.
Different join orders and directions, when to evaluate predicates, degree of parrallelism, type of join, grouping algorithm. There's no point in spending more time searching this problem space for the best algorithm than the time saved.

**Query execution operations:**
0. T-SQL: statement to execute
1. Parsing: check the syntax, parse the tree of logical operations
2. Binding: name resolution, algebrize tree associated with objects
3. Optimization: Generation of candidate plan and selection, execution plan map logical operators to physical operators
4. Execution: Query execution, plan caching

Steps 1 - 3 are performed by the "relational engine" component, that works at a logical level. The actual execution (step 4) is performed by the "storage engine" that carries out the physical operations. Both parts exist in the same service.

In the execution phase SQL server may elect to cache the plan, which skips the optimization again. 

Cached plans can be parameterised so that one plan can be used for a wider applicability of queries. However, autoparameterisation is limited in what it can acheive, explicitly parameterising queries in your sprocs is considered better practice.

Optimisation is cost-based. A *cost* is assigned to each possible plan, with higher numbers for more complex plans (implying a slower query). The query optimiser does not guarantee that a global optimal is selected, it trades off time and plan quality within the search space.

*Cost* is calculated as a function of estimated cardinality (number of rows) and choice of algorithm. This is intended to model usage of disk I/O, CPU time, and memory. The cost of a plan = ∑(operator cost)

The cost estimates take advantage of the optimiser's *statistics* which have been gathered on all of the objects. These cover the tops of total number of rows, and distribution of key values for an index.

Using cached plans is not always the best idea, for example if a table has grown quickly, the plan may have suggested doing a scan (fast enough for small tables).

The query optimiser can guess cardinality by performing 'parameter sniffing' or trying to guess a value for a parameter and building a plan accordingly.

Plan optimisation gotchas:
* The plan search space is too big and a vastly suboptimal plan is selected due to time constraints
* Statistics not present or out of date
* A cached plan is suboptimal for a particular parameter value
* Sniffed parameter values provide a poor estimate of cardinality
* Poor estimates of physical cost of particular algorithms.
* Hardware changes can cause different plan performance characteristcts, it more CPUs -> CPU bound plans are now preferred.

### SQL Server Extended Events, SQL Trace, SQL Server Profiler
There is no such thing as zero-cost monitoring. But you want as lightweight a monitoring system as possible.
SQL Server Extended Events is very lightweight,
Enables correlation between SQL Server istances and the OS.

Extended events objects:
 * **Events** What you're monitoring
 * **Targets** consumers of events, eg write to file, store in memory, aggregate
 * **Actions** responses bound to events. Might capture stack traces, store data, aggregate events or add data to the event.
 * **Predicates** filter captured events, so that you only capture the events that you need
 * **Types** Give the captured event data meaning
 * **Maps** Internal SQL server tables that map internal numeric values to meaningful strings.

 SQL Server audit gives DBAs lightweight auditing based on extended events.

 SQL Trace is an internal SQL server mechanism for capturing events it will be deprecated in future version of SQL server.

 Traces can be created using system sprocs manually or throught the SQL Server Profiler UI. *Events* are traced.
 A source for a trace could be some logical event, like a batch or a deadlock. After the event occurs, the trace gathers the information, passes it to a queue (if it passes the filters). The event is consumed from the queue to write to a file / table / display in a UI.

SQL Profile is  UI that is based on SQL trace that can be used for ad-hoc diagnostics.

SQL Server Profiler Drawbacks
* You increase the monitoring impact over SQL trace only.
* The profiler competes for resources if running on the same OS as the instance
* If using it remotely, theres the implication of the events travelling over the network
* SQL Profiler consumes memory in line with the number of events captured.
* You might close the profiler and lost your trace.

The book says you should use SQL trace before immediately going on to talk about it (along with SQL profiler) being deprecated in future releases in favour of extended events. 

## Lesson 2: Using SET Session Options and Analyzing Query PLans
How to analyze a query with the help of SET session options and exposed query plans.

### SET Session Options
SQL stores data on *pages*. Pages are physical units on a disk inside the SQL DB. 
The size of a page is fixed to 8KB or 8,192 bytes. The data within a page can only relate to a single object (table, index, indexed view).

Logical groups if pages are called *extents*. Extents can be *mixed* if their pages belong to multiple objects or *uniform* if their pages are all the one object.

One of the aims of query optimisation is to minimize disk I/O, i.e. minimising the number of pages the SQL server has to read to execute the query.

Turning on statistics I/O gives you information about about the number of pages accessed per query: `SET STATISTICS IO`. 

Do a `DBCC DROPCLEANBUFFERS` to clear out any cached execution plans.

The output looks like this:
```
(91 row(s) affected)
Table 'Customers'. Scan count 1, logical reads 5, physical reads 1, read-ahead reads 10, lob logical reads 0, lob physical reads 0, lob read-ahead reads 0.
(830 row(s) affected)
Table 'Orders'. Scan count 1, logical reads 21, physical reads 1, read-ahead reads 19, lob logical reads 0, lob physical reads 0, lob read-ahead reads 0.
```

#### Interpretation:
* **Scan count** The number of index or table scans
* **Logical reads** Pages read from data cache - when query reads whole table this will give you an estimate of table size
* **Physical reads** Pages read from disk (non-cached pages)
* **Read-ahead reads** The number of pages SQL server reads ahead
* **Lob logical reads** Large objects read from data cache, (VARCHAR(MAX), XML, GEOGRAPHY AND GEOMETRY, etc)
* **Lob physical reads** Large objects read off disk
* **Lob read-ahead reads** Large objects read ahead

Logical reads give you a rough guide as to the effectiveness of a query.

Use the examples
```
select c.custid, c.companyname, o.orderid, o.orderdate
    from sales.customers c
        join sales.orders o on c.custid = o.custid

select c.custid, c.companyname, o.orderid, o.orderdate
    from sales.customers c
        join sales.orders o on c.custid = o.custid
    where o.custid < 5
```

The statistics IO says that query 1 requires only 2 logical reads whereas query 2 requres 60. Even though the 2nd query filters on custid, so theoretically required less rows!? Well, all table touches are counted, even if the pages are already in the cache. Cached page touches are not expensive. So in this case the statistics IO is not giving us the full explanation of what is happening.

Another option is to `SET STATISTICS TIME ON`. Which gives you an indication of the CPU time and total execution time required to perform a sql statement.

### Execution Plans
Execution plans provide the most exhaustive information about how a query is executed.

SQL server exposes both an estimated execution plan as well as the actual execution plan. Displaying just the estimated plan does not execute the query. They usually don't differ but there are cases where the estimated plan is not able to give the full picture that the actual one can.

For example:
Creating a querying a temp table, the optimizer cannot select a plan without running the query.
Similarly, optimisation of dynamic SQL is performed JIT.
Estimated plans are best utilised for long running queries in prod.

Estimation plans are available graphically, as text or as XML. Text is deprecated and being removed.

Commands:
* SET SHOWPLAN\_TEXT, SET SHOWPLAN_XML, SET SHOWPLAN\_ALL : estimated
* SET STATISTICS PROFILE, SET STATISTICS XML : actual plans
* Or, you can do it in ssms through right-click context menu or keyboard shortcuts (Ctrl + L, Ctrl + M = Estimated, Actual)

Through the execution plan you can see the physical operators. In terms of index seeks, scans, nested loop or hash joins, etc.
It is read from the right to left and top to bottom.
There are also relative costs as a percentage of total query cost.
Allows you to narrow down onto the most expensive operators in the query.

Query plans also show the data flows, the thickness of these arrows corresponds to the amount of data flowing.
Hovering over an arrow gives more details about an operator or flow.

The details show the physical and logical operators, estimated vs actual, operator cost and subtree cost. Even more details are available by pressing F4 to bring up the properties pane.

Common exec

## Lesson 3: Using Dynamic Management Objects
SQL constantly monitors itself and gathers instance health information.
This is exposed through SQL Server DMOs in the sys schema and begin with dm_.

Often there is too much activity on a system (especially one that is already under load and struggling) to effectively capture information from traces and profiles (or extended events). A lot of the information that you need is already gathered in a DMO.

Beware though, they are cleared out when an instance is restarted, so their usefulness is a linear function of uptime.

### The most important DMOs for query tuning.
* **SQL Server OS-related DMOs** Manages the OS resource info that are specific to sql server
* **Execution-related DMOs** Insight into queries that have been executed, including their text, plan, count of execs and more.
* **Index Related DMOs** These provide more useful information about index usage and also *missing* indexes.

`sys.dm\_os\_sys\_info` - Important information about the underlying OS
`sys.dm\_os\_waiting\_tasks` - Sessions taht are currently waiting on something. Join to dm\_exec\_sessions and filter is\_user\_process = 1 to filter out system sessions.
`sys.dm\_exec\_requests` - currently executing requests, including a way to get their text.
`sys.dm\_exec\_query\_stats` - information about executed queries, including CPU consuption per query, disk IO per query, and more. Also possible to then identify problematic ones and use dm\_exec\_sql\_text to retrieve the text of the query as well.
`sys.dm\_db\_missing\_index\_details` as well as columms, groups and group stats to identify indexes you don't have that you would be benefit from as well as index usage stats to identify those that you do have but aren't of any benefit.

