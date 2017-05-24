# Chapter 10: Modifying Data
## Lesson 1: Inserting Data
### INSERT VALUES
Use the insert values statement to insert one or more rows into a target table.
eg. `INSERT INTO Sales.MyOrders(custid, empid, orderdate, shipcountry, frieght) VALUES (2, 19, '20120620', N'USA', 30.00)`

- It is not strictly neccessary to specify the target columns directly, but it is considered a best practice.
- You don't need to specify identity columns. You need to set IDENTITY_INSERT ON if you do.
- If no value is specified SQL server will use a default (or NULL)
- You can specify multiple sets of values.
- You can also insert from a SELECT statement in place of the values clause.
- - In this case it may be beneficial to avoiding a fully logged operation.
- - Investigate using a simple model, or the tablock hints.

### INSERT EXEC
- Insert result set from a dynamic batch or sproc
- Can insert multiple result sets, so long as they are all compatible with the destination table.

### INSERT INTO
- Two parts, the query (SELECT...) and the table (INTO...)
- Creates the destination based on the column names, types, nullability and identity properties of the source.
- Indexes, partitions, constraints, permissions, triggers.
- If you don't want the target to have an identity, manipulate the source (i.e. ID + 0), use an isnull() to make the destination NOT NULL
- If you want to change the type, cast it. Will always allow nulls though.

## Lesson 2: Updating data
T-SQL supports the standard UPDATE statement
```
  UPDATE <table>
    set <column> = <expression>, ...
    WHERE <predicates>
```
**Note:** If you do not supply a predicate, all rows in the table will be updated.

It is possible to perform updates based on joins (non standard T-SQL).

In this case the format is
```
    update <table alias>
        set <col> = <expr>
        FROM <table> <table alias>
            <join clauses>
        WHERE <predicates>
```
Updates with JOINs can be nondeterministic when multiple source rows match one target row. You won't get a warning. It just selects 'one', not sure which.
Instead you can user MERGE (Chapter 11)

### Update and Table Expressions
- It's possible to update through CTEs and derived tables.
- This is useful because you normally would run a SELECT first before running an UPDATE
- Instead you can write a CTE, test the select then just update straight into the CTE (or inline subquery)

### UPDATE Based on a Variable
- Updates the underlying table and collects the new value.
```
 UPDATE Sales.MyOrderDetails
    SET @newdiscount = discount += 0.05
    WHERE orderid = 10250 and productid = 51;
```

Updates execute all at one, so if you use Col1 in the assignment of Col2, as well as updating Col1, you get the pre-update value of Col1.

## Lesson 3: Deleting Data
### DELETE
DELETE statement removes rows from a table. 
```
DELETE FROM <table> WHERE <predicate>
```

If  you don't use a predicate, all rows get deleted.
DELETE statements are fully logged, can cause the transaction log increase dramatically in side. The can also result in lock escalation from row to table locks and blocking as a result.

Is is common to avoid this by deleting in parts, using TOP and wrapping the deletes in a loop.
```
WHILE 1=1
BEGIN
DELETE TOP 1000 FROM Products;
IF @@rowcount < 1000 BREAK;
END;
```

### TRUNCATE
Deletes all rows without an optional filter.
**Differences with DELETE**
* Truncates writes less to the log. Way faster for large datasets.
* Truncate resets the identity property
* Delete supported when there is a foreign key, truncate is not.
* Can't truncate a table used in an indexed view, can delete.
* Truncate requires ALTER permissions. DELETE requires DELETE permissions.
