# Chapter 15: Implementing Indexes and Statistics
## Lesson 1: Implementing Indexes
Pages are the smallest units of readable data.
Pages are grouped into extents, made up of 8 pages. If an extent contains pages from more than 1 object, it is called a mixed extent. Otherwise, it's a uniform extent.

### Heaps and Balanced Trees
Pages are physical structures, SQL Server organises them into logical structures.
These are either *heaps* or *balanced trees*. A table organised as a balanced tree has a clustered index. It is known as a clustered table.

All indexes are balanced trees, they the leafs are either the records or a pointer to it.

#### Heaps
Tables without any indexes. Pages and extents without order.
Index Allocation Map (IAM) page keep track of which pages and extents belong to an object.
Each IAM page can be up to 4GB of indexed space, objects larger than this can be split across multiples (doubly linked lists).

The only way to find data in a heap is to scan the whole thing.
It will scan a heap in *allocation* order (physical order on disk).
Allocation order is pretty much just whereever the DB engine plonks it.

`select name, type, type_desc from sys.indexes` to query indexes and `select * from sys.dm\_db\_index\_physical\_stats()` and `exec dbo.sp\_spaceused @objname=...` 
do check what's allocated to the object 

#### Clustered Indexes
You organise a table as a balanced tree when you create a clustered index.
It resembles the inverse of a tree.
Single root, multiple leaves, zero or more intermediate levels.
Data is stored in the leaf pages.
Data is stored in logical order of the clustering key.
Clustering key can be a single column or multiple ones (composite key) up to 16 columns not exceeding 900 bytes wide.

Creating (or changing) a clustered index reorganises the same data. It doesn't create a new table or duplicate anything.

Pages above leaf level point to leaf level pages. These rows contain clustering key values in their nodes that point to the first pages where the value starts in the locally ordered leaf level. Thus if the table is small enough that only 1 page can addresss the entire dataset, then it only has the 1 root. Once it gets bigger, SQL server creates the first intermediate level page.
Pages within clusters are doubly linked lists. So look ups are O(logn) + C where C = records in page.

The clustering key doesn't need to be a unique value, SQL server will add it's own uniquifier if needed.

SQL server can *seek* for a row in a clustered index. To find a specific row, the number of pages it has to read with a clustered index is significantly smaller than if it were a heap. If you're returning the whole table, then it needs to do a scan anyway. It can also do a range scan if you've requested a chunk.

Inserting into clustered indexes is slower, especially if it has to shuffle the pages and nodes around to keep it balanced. Clustered indexes don't guarentee contiguous logical physical storage so you may still have some external fragmentation.

Internal fragmentation can be controlled with FILLFACTOR for leaf-level and with the PAD_INDEX option for the higher level pages. 

Indexes can be REBUILD or REORGANISE 'd to get rid of external fragmentataion
` ALTER INDEX ... REORGANIZE`

The shorter the clustering key, the more rows you can fit on the higher level rows => more efficient seeks (best for OLTP eg. int ID IDENTITIY(1, 1)
Data warehousing typically queries large sequential blocks of data based on a date column, so you might prefer to support a partial clustered index scan and choose your index accordingly.

#### Nonclustered Indexes
Similar structure to clustered, but the leaf is a pointers to data (index key or row locator)
Which it is depends on whether the referenced data is a balanced tree or a heap, heap - RID (8-byte pointer containing db, page, row location) else clustered index key.

Up to 999 nonclustered indexes available, 16 cols per key upto 900 bytes wide.

Seek operation on heap
- Traverse to leaf level and then retrieve row (RID) for page.
- If very selective (ie only returns 1 row) then it's fast. If you have to do it multiple times, then lose benefit.
- You can do a range scan so long as they all live in the same page.

Seek operation on a balanced tree:
- Navigate down non-clustered index until row locator, then navigate down clustered index to retreive row.
- Potentially more reads.
- But it lets you rebuild/reorg the clustered index (incl inserting rows) without fucking up the non-clustered pointer.

If a row moves in a heap, it needs to update the indexes. If it moves to another page, it leaves a forwarding pointer so that it doesn't have to update the indexes.

It's still a good practice to keep things as a clustered tree.

Clustered index should be short, because it appears in every non-clustered index leaf node (unless datawarehousing, select one that supports partial scans of relevance)

Clustering key should not change frequently (or at all) as changing it needs an update to all non-clustering indexes.

A filtered index spans a subset of column values only and therefore only applies to a subset of table rows. They are useful when particular values occur rarely compared to other more frequent ones. Create a filtered index over the rare ones only. Less expensive to maintain, fast for seeks and will end up scanning anyway for the common vals. `create index bleh() where ...`

Also enables you to do filtered uniqueness (i.e. non-nulls must be unique) filter NOT NULL on a UNIQUE INDEX.

SQL Server 2012 has a method for storing nonclustered indexes - column-store indexes. Good for datawarehousing queries.
Split the values in teh column into segments which are indexed and stored on disk / memory.
Stored compressed
Only have to fetch the single column from disk.
The rows are still reconstructed though, so they typically aren't useful for OLTP workloads.
Also, tables with columnstore indexes are read-only, you need to drop the table to update them

### Implementing Indexed Views
Might be worth optimizing queries that aggregate and join lots of data by storing the results of the view.
One option is to ETL and store the table like that.
If you create a view with that implements the required transformation logic and index it you materialise the view.
They are maintained manually, but if this is adding too much overhead, they can be dropped or disabled and re-enabled as required.

## Lesson 2: Search Arguments
### Supporting Queries with Indexes
* Use the where clause to filter rows
* Check whether an index was used using `sys.dm\_db\_index\_usage\_stats`
* Using a where does not guarantee an index will be used
* * It has to be supported by the index first.
* It might be less expensive for the db to perform a table or clustered index scan that a nonclustered index seek.
* Aggregation is doen by either streaming or hashing. Streaming is faster but it needs sorted input that it's grouping by.
* Indexes can benefit the aggregations themselves, in the case of min / max./
* If all the data is found within the index, then the query is 'covered'. These are very efficient.
* You can use the INCLUDE clause of create... index to ensure commonly retreived fields are covered. This stores these columns at the leaf level of the index and so you should not include too man.

### Search Arguments
The WHERE clause does not guarentee index usage.
The predicate must suit the use of indexes, ie. they need to be search arguments or SARGs.

Requirements for a SARG;
* Column has an index
* Appears in the query alone, not as a function parameter
* And compared to a column value that is not mutated.
* `<column> <inclusive operator> <value>`
* Column name must be alone on one side of the expression
* Inclusive operators are =, <, >, <=, >=, BETWEEN, LIKE
* LIKE is inclusive only if you don't use a wildcard at the beginning.

Example:
Replace `WHERE DATEDIFF(DAY, '20060709', orderdate) < 2` with `where dateadd(day, '20060709', 2 ) >= orderdate`



## Lesson 3: Statistics