# Chapter 12: Transactions, Error Handling and Dynamic SQL.
## Lesson 1: Understanding Transactions
A transaction is a logical unit of work
They are intended to be atomic. All of the work completes or none of it.

All changes in SQL server take place in the context of a transaction, in other words all operations that in any way write to the db are treated as transactions.
This includes:
* All DML (insert, update, delete) statements
* All DDL (create, drop, alter, etc)

The keywords COMMIT and ROLLBACK control the result of the transaction.

### ACID 
 * Atomicity - unit of work, all or nothing.
 * Consistency - the db is left in a consistent state (all constrains met)
 * Isolation - Independent of other changes going on at the time
 * Durability - Endures through interruption of service, committed changes are completed and uncomitted ones are rolled back.

SQL server has measures in place to ensure that the ACID properties are maintained. Commands cannot partially complete. 

Commands that voilate constraints are rolled back. 

Isolation is guarenteed through the locking mechanisms.

SQL Server maintains transactional durability through the transaction log. Every change is logged first, capturing the before and after states. On commit, if all the checks pass, then log entries are marked as applied. If a rollback is required, the db knows what to do to rollback to the start.

### Types of Transaction
 * **System** Maintain internal persistent system tables with system transactions. These are not under user control.
 * **User transactions** These are transactios created by users in the process of changing and reading data. They can be automatically, implicitly or explicitly. The `sys.dm\_tran\_active\_transactions` view tells us information about current running transactions. You can explicitly name your own transactions using the transaction commands.

#### Transaction Commands
Every transaction begins with T-SQL `BEGIN TRANSACTION` or `BEGIN TRAN` (same thing). You must end the transaction at somepoint with a `COMMIT TRANSACTION` or `COMMIT TRAN` or `COMMIT WORK` or `COMMIT`. Alternately, if rolling back replace commit in the previous statements with `ROLLBACK`.

#### Transaction Levels and States
Functions:
* **@@TRANCOUNT**
* * Level of 0 -> the code is not within a transaction
* * Level > 0 indicates an active transaction, Level > 1 indicates the nesting level.

* **XACT_STATE()**
* * 0: no active transaction
* * 1 there is an uncommited transaction that can be committed. Does not report nesting level.
* * -1: there is an uncommitted transaction that can *not* be committed due to a previous error.

#### Transaction Modes
**Autocommit**
The default mode, single statements executed im the context of their own transactions and immediately committed or rolled back on error. You don not need to issue any of the transaction commands.

**Implicit Mode**
When you issue one or more DML or DDL statements (or a select), SQL server starts the transaction but does not commit it. You must do your own COMMIT or ROLLBACK. This mode is *not* the default. You must turn it on with `SET IMPLICIT\_TRANSACTIONS ON`

@@TRANCOUNT is incremented by 1 the first time you issue a DML or DDL statement.

Implicit mode is the ANSI default (by the sounds) which would explain why Oracle does this.

The advantage of an implicit transaction are:
- You can ROLLBACK anything.
- You might be able to catch errors if you don't commit them

The disadvantages are
- More locks for ad hoc querying, you might forget to COMMIT / ROLLBACK to release them
- You need to remember to set it on for the session
- Does not play nicely with explicit transactions.

**Explicit Mode**
Occurs when you issue BEGIN TRAN statements - @@TRANCOUNT incremented by 1 each time.

#### Notes
Transactions can span batches (GO statements) but it is good practice to keep them within the same batch. 

#### Nested Transactions
Transactions within transactions
```
    BEGIN TRAN
        Do stuff 1
        BEGIN TRAN
            Do stuff 2
        COMMIT TRAN
        Do some more stuff 3
    ROLLBACK TRAN 
```
In the above case, the outermost rollback trumps the innermost commit. The inner commits have no real effect other than to decrement @@TRANCOUNT by 1.

You can only issue one ROLLBACK, no matter then nesting level.

``` 
BEGIN TRAN
    BEGIN TRAN
<---ROLLBACK TRAN
```

#### Marking a Transaction
Explicit transactions can be named with `BEGIN TRAN <name> WITH MARK`.
Names can only be 32 characters long.
Named transactions place a mark in the log, which means you can restore to that point in time easily.

Example
``` R
RESTORE DATABASE TSQ2012 FROM DISK = 'C:\SQLBackupts\TSQL2012.bak' WITH NORECOVERY;
RESTORE LOG TSQ2012 FROM DISK = 'C:\SQLBackups\TSQL2012.trn' WITH STOPATMARK = 'Tran1';
```

**Options when Restoring to Marks:**
 * You must use the transaction name if you use STOPATMARK
 * There is an optional description after WITH MARK (but it is ignored)
 * You can restore to just before the transaction with STOPBEFOREMARK

#### Additional Transaction Options
 * **Savepoints** These are locations within transactions that you can use to roll back a subset of work. ` SAVE TRANSACTION <savepoint name>`
 * **Cross-Database Transactions** SQL Server can preserve ACID across multiple databases, with limitations around the failover response of mirrored instance.
 * **Distributed Transactions** Transactions can span more than one server (over a linked server). SQL Server will elevate transactions to distributed transactions by invoking MSDTC (distributed transaction coordinator)
 * * Distributed transactions cross SQL instance boundaries, not just SQL DB boundaries.

### Basic Locking
Two types of locks:
 * Shared - read lock
 * Exclusive - write lock

Basically, only shared locks are compatible with one another.

|         |:Shared:|:Exclusive:|
|---------|--------|-----------|
|Shared   |No      |No         |
|Exculsive|No      |Yes        |

### Blocking
If two sessions request an exclusive lock over the same resouce, one is granted the resource, the other is blocked. The the blocked session must wait until the first session releases its exclusive lock. in a transaction, the locks are held to the end of the transaction.

Exclusive locks can also stop shared locks being acquired. However, with READ COMMITTED as the isolation level, shared locks are released as soon as the SELECT statement finishes (not held until the end of the transaction)

### Deadlocking
A deadlock results when two transactions are waiting on each other to finish, before continuing.
Eg. Tran1 holds lock on A, wants to acquire a lock on B. However, tran 2 wants holds lock on B and waiting for lock on A before it can complete and relaese B.

SQL server will detect this and kill one of the sessions. Return 1205 to the client.

It can be avoided by doing updates in the same order.

### Transaction Isolation Levels
* **READ COMMITTED**
* * Default, all readers only can access committed tables. 
    SELECT statements attempt to acquire shared locks, release them when they finish. Will be blocked by any exclusive locks on the resources.
* **READ UNCOMMITTED**
* * Readers can access uncomitted transaction's data, readers are not blocked by writers however may read data that is later rolled back
* **READ COMMITTED SNAPSHOT**
* * Optional way of using read committed, RSCI uses tempdb to store original versions of changed data, the readers access those until the transaction holding the exclusive lock is committedd (or rolled back)
* * Set at the database level
* * Think of it as READ COMMITTED isolation level without the reader blocking but a lot more tempdb activity.
* * Default level for Windows Azure SQL.
* **REPEATABLE READ**
* * A stricter level than read committed
* * Guarantees that whatever data is read can be reread later in the transaction (keep shared locks until the end of the transaction)
* **SNAPSHOT**
* * Uses row-versioning in tempdb, does not require shared locks but won't read dirty data.
* * Reads are not repeatable.
* **SERIALIZABLE** 
* * All reads are repeatable.
* * Strongest level, table level locking on shared locks (depending on predicate of SELECT)

**Notes**
Isolation levels are set pers session. If you do not set a different isolation level in your session, READ COMMITTED is selected as the default (OR RCSI in Azure SQL)

## Lession 2: Error Handling.
### Detecting and Raising Errors
Used to handle errors from the point of view of SQL Server (i.e. doing things it won't let you - violating constraints) or just errors from your code's point of view (like you can't X that Y if condition Z isn't met.)
 * RAISERROR (Old)
 * THROW (SQL Server 2012)

#### SQL Server Error Messages
* **Error Numbers**
* * Numbered from 1 to 49,999.
* * Custom messages from 50,000 ->
* * 50,000 reserved for custom message without a custom number.
* **Severity level**
* * Generally Severity Level >= 16 are automatically logged
* * 19 - 25 can only be specified by members of sysadmin role
* * 20 - 20 are considered fatal, terminate the connection and rollback the transactions.
* * 0 - 10 are informational only
* **State**
* * Used by Microsoft for internal purposes.
* **Error message**
* * SQL Server error messages listed in sys.messages
* * You add them with sp_addmessage

#### RAISERROR
```
 RAISERROR ( 
     { msg_id | msg_str | @local_variable } 
     { ,severity ,state }
     [ ,argument [ ,...n ] ]) [WITH option [, ...n]]
 )
```
The message can be a string, a c style printf format string, or a variable.
`RAISERROR ('Error in % stored proc', 16, 0, N'usp_InsertCategories')`

* You can use the sev levels of 0 - 9 to issue information messages (like a print)
* You can RAISERROR > 20 if you use WITH LOG, and are sysadmin role member. This will terminate the connection.
* RAISERROR with NOWAIT to send messages to the client. The message does not wait in the output buffer this way.

#### THROW
```
THROW [ {error number}, {message}, state];
```
Behaves mostly like RAISERROR with a few exceptions.
* No parentheses for params
* Can be used without params from a CATCH block
* Error number, message and state are all required.
* Message cannot be a format string (use FORMAT-MESSAGE and a variable)
* The state must be 0 - 255
* Any parameter can be a variable
* Severity is always 16
* Always terminates the batch, unless it is in a TRY block.

**Note:** The statement before a throw must terminate with a semicolon.

#### TRY\_CONVERT an TRY\_PARSE
These preempt potential errors so that your code can avoid unexpected errors.
They return null instead of raising/throwing an error. If they succeed, then the value is returned
```
SELECT TRY_CONVERT(DATETIME, '1700-01-01'); -- Returns NULL;
```
Try parse works in a similar manner, but parsing from text.

#### Handling Errors After Detection
There are two error handling methods, checking *@@ERROR* and *CATCH*

Unstructured Error Handling
* Test individual statements after execution
* IF @@ERROR = 0 -> No error ELSE @@ERROR contains the error number
* * You need to capture @@ERROR in a variable before you test for it because reading it resets it.
* Does not provide a central place to handle errors
* * Encumbent on the developer to provide structure


#### Using XACT_ABORT
- This option will (when on) will rollback any transactions when an error sev > 10 occurs.

Limitations:
- You cannot trap for the error or capture the error number
- Any error causes a rollback
- None of the remaining code is executed
- After the xact is aborted, you can only infer the line that failed based on the error message.

#### Using TRY/CATCH
Added in 2005
* Wrap the code you want to handle errors in a TRY BEGIN...END block.
* * Every try block must be followed by a catch
* * TRY CATCH must appear in same batch
* If an error is detected in the TRY, code execution jumps to the catch for handling
* * The rest of the statements within the try block are not executed
* * Control jumps to the next statement after the catch
* * If no error is detected the catch block is not executed
* No message is sent to the client if an error is detected in a try block.
* * Even RAISERROR 11-19 in a TRY does not trigger a message back.

With TRY/CATCH you don't need to trap individual statements for errors.

**Rules for Using TRY/CATCH**
 * Severity >10 and <20 transfer control to the catch
 * Errors >= 20 only transfer control if they don't close the connection
 * Some runtime errors (and compile ones) abort the batch and skip the catch
 * Errors in the catch block abort the transaction and the error is returned (unless the TRY/CATCH is within an outer TRY)
 * You can commit or rollback in CATCH
 * A trycatch does not catch fatal errors
 * you cannot use try/catch to test for object existance.
 
**Functions available within CATCH**
* ERROR_NUMBER
* ERROR_MESSAGE
* ERROR_SEVERITY
* ERROR_LINE
* ERROR_PROCEDURE
* ERROR_STATE

```
BEGIN CATCH
    SELECT ERROR_NUMBER(), ERROR_MESSAGE(), ERROR_LINE(), ERROR_SEVERITY(), ERROR_STATE()
END CATCH;
```

You can throw your own errors in a try block using THROW or RAISERROR. RAISERROR must have a sev level of 11:19 inclusive.
In the CATCH block, you can RAISERROR or you can THROW with or without parameters (like in .NET)
If you use a RAISERROR in CATCH, it must have a custom error number
THROW with parameters must be a custom error number
THROW without parameters re-raises the original error message. This terminates the batch and does not execute any remaining commands in the CATCH block.

XACT_ABORT reacts differently in a try block. It does not terminate the transaction, it transfers control to the CATCH and leaves the transaction uncommittable. You will need to roll it back.

## Lesson 3: Using Dynamic SQL
### Overview
Think of Dynamic SQL as inversion of control, or parameterising common logic.

You *need* dynamic SQL in the following cases (not neccesarily exhaustive):
* The database name in the USE statement
* Tables in 'FROM'
* Column names in SELECT, WHERE, GROUP BY, HAVING and ORDER BY
* Contents of lists in IN and PIVOT

Common use cases:
* Automation of administrative tasks
* Iterating all DBs on a server, objects in a DB, through metadata of objects
* Sprocs with many params that execute a different query based on the parameters
* Constructing parameterized ad hoc queries that can reuse previously cached execution plans.
* Constructing commands that require elements of the code based on query the underlying data eg: dynamic pivot.

### Generating SQL Strings
Delimiting is important.
In SQL 2012 you must use the single quote (apostrophe) to delimit a string. This is because of QUOTED\_IDENTIFIER. When QUOTED\_IDENTIFIER is 'ON' (the default), strings are delimited by using singlw quotes and itentifiers are delimited by double quotes.

**TIP:** You should leave quoted identifier on, because that is the ANSI standard. As well as being the SQL server default.

You then need to do 2x single-quote characters in order to escape one inside a delimited string.

So to print SELECT * FROM customers where address = N'5678 rue de l''abbaye';

`PRINT 'SELECT * FROM customers where address = N''5678 rue de l''''abbaye'';'`, or for a more simple approach use: `PRINT QUOTENAME(N'5678 rue de l''abbaye', '''') to automatically double up the quotes.

### Executing SQL Strings (EXEC)
Simplest method to execute dynamic SQL
- Can only EXEC a single batch (may have more than one command, but no 'GOs')
- You can use string literals or variables.
- - Variables can be regular or unicode
- - The variables can be (MAX) lengths

### SQL Injection
Using dynamic SQL opens you up to the possibility of injection, especially if some of the sql is generated from user input. 
Even a single quotation mark can cause a query to crash in such a way that indicates that the passed in value is being executed. 
One method to avoid it is to sanitize your dynamic sql by parameterizing it and using sp_executesql

### Executing SQL Strings (sp_executesql)
A stored procedure created as an alternative to using exec. It accepts parameters as unicode strings.
` sp_executesql [@statement = ] statement [ @params = N'parameter\_name datatype', @param1 = 'value', @param2 = 'value', ...n]

 