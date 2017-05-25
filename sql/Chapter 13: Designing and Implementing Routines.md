# Chapter 13: Designing and Implementing Routines
## Lesson 1: Stored Procedures (TODO)

## Lesson 2: Implementing Triggers
A special kind of stored procedure associated with selected DML events on a table or view. Cannot be explicitly executed, they are fired automatically when the associated DML event occurs.

### DML Triggers
#### AFTER
This trigger fires after the event finishes and can only be defined on permanent tables.

#### INSTEAD OF
This trigger replaces the event and can be defined on permanent tables or views.

#### General Info
DML triggers can be used for auditing, enforcing complex integrity rules.
The execute as part of the same transaction as the DML event that triggered them. This means that a ROLLBACK TRAN from within the trigger will roll back the whole transaction as well as whatever the DML event did.
If you want to rollback just the DML event's changes, you can do a RAISERROR or a THROW.
The normal exit from a trigger is a RETURN as in a sproc.
In the trigger TSQL code you can access tables named inserted and deleted, they function the same as the OUTPUT clause.
In the case of an INSTEAD OF trigger, the inserted and deleted tables contain the rows that would be affected by the DML statement.

#### AFTER triggers
Only available on tables, the trigger code executes after the DML statement has passed all constraints. If a constraint is violated, the trigger is not executed.

**Exam Tip**
Use `@@ROWCOUNT = 0` as the *first* line in the trigger, if it is 0 then you don't need to do anything in the trigger as inserted and deleted will be empty. It has to be the first line, any other line will reset the value of @@ROWCOUNT

Returning result sets from triggers is not advisable, and can be disallowed with the sp_configure option called 'Disallow Result Sets from Triggers'. It's also deprecated and dropped from SQL versions post 2012.

#### Nesting Triggers
You can update a table from a trigger. The updated table may in turn have it's own trigger defined. You can nest them to a depth of 32 and if you're reaching this limit take a good hard look at yourself.

They can be sidabled with an exec sp_configure 'nested triggers', 0; RECONFIGURE;

#### INSTEAD OF Triggers
This executes the batch of sql code instead of the DML event that caused it.

Although insteadOf triggers can be created against both tables and views they are commonly used with views.

This is done to get around some of the restrictions on updating base tables through views (e.g. can only update 1 base table at a time, aggregations that prevent a direct update). 

### DML Trigger functions
There are two functions available:
* **UPDATE()**
* * Determine which columns were used in an INSERT or UPDATE statement. e.g. `IF UPDATE(qty) PRINT 'Qty updated';`. Returns true even if the column is unmodified.

* **COLUMNS_UPDATED()** 
* * You can use this function if you know the sequence number of the column in the table. You use a bitwise & to see which columns were updated (presumanbly COLSEQNO & COLUMNS_UPDATED() == 1 -> updated?)


## Lesson 3: User Defined Functions
T-SQL or CLR routines that accept parameters and return *either* scalars or tables.

Their purpose is to encapsulate code.
Like a sproc they can accept params. Unlike a sproc they are embedded in T-SQL statements and not EXECUTEd.

UDFs cannot perform DDL or DML (insert, update, delete) on a permanent table.

Table-valued UDFs are accessed in the FROM clause

### Scalar UDFs
Return a single value.
Can appear anywere in a query that any other scalar returning expression can appear.
Example
```
CREATE FUNCTION dbo.FunctionName (@param int)
RETURNS int
AS 
BEGINS
    RETURN 1 + @param;
END;
GO
```

### Table UDFs
Inline table valued UDFs don't have any statements other than the return() statement itself.
Treated by the optimizer the same as a view.
You can INSERT UPDATE DELETE on them just like a view.
Think of them as parameterized views.

```
CREATE FUNCTION dbo.FunctionName(@param int)
RETURNS TABLE AS RETURN(
    SELECT * FROM dbo.Table where @param = column;
);
SELECT * FROM dbo.FunctionName(3);
```

Multistatement UDFs have to define the return value as a table variable and insert their data into the variable before eventually returning it.

```
CREATE FUNCTION dbo.FunctionName(@param int)
RETURNS @returnvalue ( col1 int, col2 int)
AS
BEGIN
 insert into @returnvalue values ( 1 , 3)
 return;
END
```

### Limitations of UDFs
Cannot:
* Apply DDL or DML
* Change DB or SQL instance state
* Create or Access temp tables
* Coll sproc
* Produce side effects (e.g. RAND and NEWID are not allowed)

### Options of UDFs
WITH:
* **ENCRYPTION** Obfuscation of source code
* **SCHEMABINDING** Binds schemachanges of referenced objects
* **RETURNS NULL ON NULL INPUT** Any null params return null (scalar only) without executing
* **CALLED ON NULL INPUT** Default, implies that it will execute on any null
* **EXECUTE AS** Under various contexts

### UDF Performance Considerations
Manner of use of a fuction can have various performance implications.
Scalar UDFs are often called RBAR (row by agonising row)
E.g. Scalar UDF in the SELECT executes once for every row in output. Scalar UDF in where clause executes once for every row in table.
Use of a scalar udf prevents the query from being parallelized. This seems like an annoying implication because why else would they not be able to have side effects!? 