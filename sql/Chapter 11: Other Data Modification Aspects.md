# Chapter 11: Other Data Modification Aspects
## Lesson 1: Sequence Object and Identity columns
Identity column properties and the sequence object are both features that generate sequences automatically. Often for use as a surrogate key.
IDENTITY is as old as the hills, has some limitations
Sequence object introduced in SQL server 2012 to overcome the limitations.

### Using IDENTITY
```
 CREATE TABLE dbo.MyOrders (
     OrderID int not null IDENTITY(1, 1) CONSTRAINT PK_MyOrders PRIMARY KEY
 )
```
You don't need to specify insert values for the Identity column, but you can if you want after setting `SET IDENTITY_INSERT <table> ON`;

**Querying the last generated identity value**
* SCOPE_IDENTITY function - last generated in current scope in current session
* @@IDENTITY@@ - last generated identity in current session, regardless of scope
* IDENT_CURRENT - function accepts table and returns last identity generated for the table.

IDENTITY property does not guarentee uniqueness - you may've inserted a future value. You need to guarentee it yourself with PRIMARY KEY, UNIQUE constraints and / or RESEED.

If you reach the max value in a type, IDENTITY will start to fail, until you reseed it.

### Using Sequences
Introduced in 2012
It's own independent object in the DB
**Sequences avoid the following limitations of IDENTITY**
* Identity is tied to a column in a table, you can't add/remove it without adding/removing the column
* Table specific, not global.
* Only generated concurrently with the operation, only available after the fact.
* No cyclying
* Column cannot be updated
* Reset by TRUNCATE

` CREATE SEQUENCE <schema>.<object>;`
**Available Options**
* INCREMENT BY (default 1)
* MINVALUE (default min of type)
* MAXCALUE (default max of type)
* CYCLE | NO CYCLE (default NO CYCLE)
* START WITH (default minvalue)
* CACHE n | NO CACHE (default CACHE 50) 

Example - equivalent to INT IDENTITY(1,1)
```
 CREATE SEQUENCE Sales.SeqOrderIds as INT MINVALUE 1 CYCLE;
    SELECT NEXT VALUE FOR Sales.SeqOrderIds; -- 1
    SELECT NEXT VALUE FOR Sales.SeqOrderIds; -- 2
    SELECT NEXT VALUE FOR Sales.SeqOrderIds; -- 3

 ALTER SEQUENCE Sales.SeqOrderIds RESTART WITH 1;
```

To use a sequence in place of an IDENTITY, you can specify the NEXT VALUE FOR in the default constraint of your otherwise identity PK column. Only assigns if not specified (no need to set insert identity)

Sequences can be cached in periods, which improves disk access performance of inserts. You might lose a range of values if the db shuts down suddenly.

Sequences don't guarentee a lack of gaps. I.e. FOR NEXT VALUE is not undone if a transaction is rolled back.

Use sp\_sequence\_get\_range to advance the sequence by a bulk amount, which effectively reserves a range for you own use.

## Lesson 2: Merging
MERGE from source table / table expression into a target table
```
 MERGE INTO <target> as tgt
    USING <source> as src
    ON <predicate>
    WHEN MATCHED [and <other predicate>]
        THEN (update and/or delete)
    WHEN NOT MATCHED [BY TARGET] [AND <more predicate>]
        THEN INSERT
    WHEN NOT MATCHED BY SOURCE [AND <even more predicates>]
        THEN (update and/or delete)

```
Notes:
- The using is treated like a from in a select. You can use joins or table expressions. Or even an openrowset.
- the ON {predicate} section defines how rows are matched between source and target. Not quite a filter like in a JOIN.
- WHEN MATCHED defines the action to take when source is matched. Because the target exists, insert is not allowed here. You can have multiple when matched functions with different predicates.
- WHEN NOT MATCHED defines the action to take when a row on source is not matched by a row on target. You can only perform INSERTS here because the target row does not exist.
- WHEN NOT MATCHED BY SOURCE target row exists but has no match on source, cannot insert can update or delete. can have multiple copies of functions.
- You must specify at least one clause, but don't need all 3. One is adequate.

Need to explicitly avoid unneccessary reads, so use the extra predicate in WHEN MATCHED to check for all columns equality.

**Avoiding Merge Conflicts**
Assume key K does not exist in target.
Two processes attempt to merge K at the same time, you might have two merges trying to insert the same key, causing a FK violation.
You should use the SERIALIZABLE hint.

## Lesson 3: Using the OUTPUT Option
THe OUTPUT option is available on data modificattion statements, it returns information about the modified rows. Example uses are auditing and archiving.

Similar to the SELECT part of a query insofar as you can specify expressions and alias them. You need to prefix the columns with either *inserted*, or *deleted*. In the case of an UPDATE the inserted are the values after the update and the deleted are the values after the update.

You can have the OUTPUT clause return a result set to the caller or pipe it INTO a table (which can have it's own OUTPUT clause to return to the user).

The target of the INTO clause cannot have triggers or participate in an FK relationship.

### Update with OUTPUT
No special tricks except as highlighted about that the inserted and deleted values correspond to before and after the merge.

### Merge with OUTPUT
You can use OUTPUT with MERGE taking into account these special considerations
- One merge statement can perform different actions, you can use the $action function to determine for a given output row, what the merge action was ('INSERT', 'UPDATE', 'DELETE')
- You can also access your source alias in a MERGE's OUTPUT.

### Composable DML
You can define what looks like a derived table based on the OUTPUT clause and use an outer INSERT SELECT to capture that output which the use of a WHERE clause to filter. You can't have GROUP BY, HAVING etc or anything besides WHERE.

