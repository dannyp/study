# Creating Tables and Enforcing Integrity
## Creating a Table
 **Options:**
  - CREATE TABLE - Explicitly define the components
  - SELECT INTO - Create a table based on the output of a query

### CREATE TABLE Syntax
```
    CREATE TABLE
        [database name].[schema name].table_name
        [as FileTable]
        ( 
            < column definitions >
            < columnset definitions >
            < table constraints >
        )
        [ ON { partition scheme [ partition_column_name] | filegroup }]
        [ TEXTIMAGE_ON { filegroup | 'default' }]
        [ FILESTREAM_ON { partition_scheme_name | filegroup | "default}]
        [ WITH (< table options >) ]
```
#### Must specify
 * Table name
 * Table columns, including:
 * - Column name
 * - Column data type

#### Able to also specify
 * **Columns:**
 * - The length of character data types
 * - Precision of numeric and (some) date types
 * - optional special cases of colums (computed, sparce, identity, rowguidcol)
 * - collation (if not the default)
 * **Constraints**
 * - Nullability
 * - Default and check constraints
 * - Optional collumn colaltions
 * - Primary keys
 * - Foreign keys
 * - Unique constraints
 * **Possible table storage directions**
 * - Filegroup (eg. ON PRIMARY being the primary filegroup)
 * - Partition schema
 * - Table compression

 It's common to define the contraints after the fact with the alter command. 

### Specifying a Database Schema
  * A schema is a named container for tables and other objects.
  * The purpose is to group the objects together, it allows tables with generic names to exist within sub areas of a database, it is also useful from the point of view of applying permissions .
  
#### Built-in Schemas
  (These cannot be dropped)
  - dbo - the default for db\_owners or db\_ddl\_admins
  - guest - a schema for guest users (rarely used)
  - INFORMATION_SCHEMA - metadata about tables.
  - sys - reserved for system objects (tables and views)
  There are also some built in schemas that match up with the built in / default roles, but are also rarely used.

  **Schemas do not nest** 
 
  You can move a table between schemas using `ALTER SCHEMA Sales TRANSFER Production.Categories` which will move the Categories table from Production to Sales. `ALTER SCHEMA Production TRANSFER Sales.Categories` will move it back.

### Naming Tables and Columns
 Important restrictions and best practices.
 **Regular Identifiers** 
  - Do not need to be surrounded with []
  - Letters (unicode)
  - Decimal numbers
  - First character must be a letter or _
  - Variables must begin with an @
  - Temp tables must begin with a #
  - Cannot be a T-SQL reserved word
  - Cannot contain spaces or special characters other than @, $, #, _
  
  Delimited identifiers do not need to conform to the above, but are required to be wrapped in [ ] 
  Use regular identifiers when possible.
  Some people like to use _ to delimit words (instead of spaces) or PascalCasingToAchieveTheSame
  Quotation marks are the ANSI standard, but in SQL Server the option SET QUOTED_IDENTIFIER can be turned off, so using quotes is risky.
  Keep object names short. That way if you include them in the names of indexes and constraints (a very common convention) then they won't risk exceeding the 128 character limit.

### Choosing column data types
 - Try to use the most efficient
 - Use NVARCHAR or VARCHAR if you expect a character string to vary in length often. 
 - If the character column is likely to be frequently updated and short, use NCHAR or CHAR
 - Use DATE, TIME and DATETIME2 over DATETIME and SMALLDATETIME
 - Use VARCHAR(MAX), NVARCHAR(MAX) and VARBINARY(MAX) over TEXT, NTEXT and IMAGE (deprecated)
 - Use ROWVERSION instead of TIMESTAMP (deprecated)
 - DECIMAL and NUMERIC are the same thing. DECIMAL is preferred because it is more descriptive.
 - ONLY use float and real when you really know what you're doing and need the precision but are familiar with rounding issues.

#### NULL and Default Values.
 NULL indicates the value is unknown.
 - If you *know* the column value may not be known, allow nulls
 - If you don't want to allow null but do want to default to a particular value, then go with NOT NULL DEFAULT(Value).

#### The Identity Propery and Sequence Numbers
 - Used to automatically generate a sequence of numbers.
 - Can only be used on 1 column per table.
 - Specify seed and increment.
 - Example: ` create table category ( categoryid int identity(1,1) not null primary key ...) `
 - MS SQL 2012 introduces Sequences - more in a later chapter (Chapter 11)

#### Computed Columns
 - Column where the value is based on an expression
 - Can be based on the value of other rows or SQL functions
 - E.g. `Fullname as Firstname + ' ' + Lastname` or ` Cost as UnitPrice * Qty `
 - Can be PERSISTED. Which means it's not computed on the fly but recalculated when the data changes and saved.
 - PERSISTED calculated columns must be deterministic, cannot reference dynamic functions like getdate() or current_timestamp. 

#### Table Compression
 - Table data can be compressed for more efficient storage (also indexes)
 - In **Enterprise Edition** table compression has 2 levels
 - * Row; more compact storage format at the row level
 - * Page; row level *plus* addition compression at the page level.
 - Example: ` create table orderdetails ( orderid int not null ... ) with (DATA_COMPRESSION = ROW)`
 - Example: ` alter table orderdetails rebuild with (data_compression = page)` 
 - Also, sp\_estimate\_data\_compression\_savings sproc will help decide whether compression is beneficial or not.

### Altering a Table
 After creation, use the ALTER TABLE command to change the tables structure or to add or remove properties.
  **Abilities**
   - Add/remove columns (new colums placed at the end - order is irrelevant in set/relation theory)
   - Change the data type of column
   - Change nullability
   - Add/remove constraints
   - - PK / FK / UQ / Check / Default.
 If you want to *change* a constraint, you need to drop and recreate it (same for computed columns)
 You cannot use ALTER TABLE to: change a column name, add/remove identity.

## Lesson 2: Enforcing Data Integrity
### Constraints
#### Primary Key Constraints
Every table should have a method of uniquely distinguishing each row. PK is the most common method of achieving this. They can be across multiple columns too.
A column that identifies every row in a meaningful way is called the natural key or business key. This could be the primary key, but developers often find it more convenient to have a unique but otherwise meaningless key called a *surrogate key*.

```
 CREATE TABLE Production.Categories (
     categoryid int not null ideantity, 
     categoryname nvarchar(15) not null,
     description nvarchar(200) not null,
     CONSTRAINT PK_Categories PRIMARY KEY(categoryid)
 )
```
Or, alternately;
` ALTER TABLE Production.Categories ADD CONSTRAINT PK_Categories PRIMARY KEY(categoryid) `

 - The primary key will be used by other tables to refer back to this one.
 - Good practise is to use the same name in both tables if possible.
 - Choose a descriptive name that flows naturally from the table name, makes life easier for everyone.
 
 **PK Requirements:**
 - The column(s) cannot allow NULL
 - Existing data must already be null
 - Only 1 PK constraint at a time on a table.

 You can list the PK constraints on tables with ` select * from sys.key_constraints where type = 'PK' `
 SQL creates a unique index to enforce the constraing whcih you can find in `sys.indexes`

#### Unique Constraints
 Similar to PK. Use for natural / Business key columns.
 Allows NULL (but you only get 1)
 Creates a nonclustered index.
 ` alter table Production.Categories add constraint UC_Categories UNIQUE(categoryname)`

#### Foreign Key constraints
 Column(s) that serve as alink to look up data in another table.
 Eg Products -> CategoryId
 Used to enforce that the CategoryId values in Products are valid entries in CategoryId
 ` alter table Production.Products WITH CHECK add constraint FK\_Products\_Categories FOREIGN KEY (categoryid) REFERENCES production.Categories(categoryid)`

 **Explanation:**
 - You declare it on the 'FOREIGN' side (i.e. the key is from a different table)
 - WITH CHECK enforces referential integrity, for new and existing data.
 - You specify a name for the key (FK_ is the convention)
 - FOREIGN KEY(categoryid) implies that the Production.Products.CategoryID column is the one we are constraining
 - REFERENCES Production.Categories.CategoryId implies that hte value must come from here. Typically it will be the primary key, but it must be unique (key or index) at the very least.

 **Rules to keep in mind**
 - Data types must match
 - 'Other' column must be unique key or index or PK
 - You can create them on computed columns.

 Creating a FK means you will probably do a JOIN on this column. You're going to have an index on one side, but the FK on it's own doesn't create one on the table with the constraint. It may help the join get resolved faster if you slap an index on that column too.

 Query sys.foreign\_keys\_table to obtain a list of FKs.

#### Check Constraints
 More general constraints beyond what is implied by the data type
 Specify an expression that must evaluate to true for the insert/update to stick.
  `ALTER TABLE Production.Products WITH CHECK ADD CONSTRAINT CHECK (unitprice >= -);`
 Good way to guarentee data quality.
 Performant when compared to triggers
 If the column allows NULLs, the constraint needs to account for that fact.
 You can't customise the error message (but honestly who is sending db errors to users)
 Cannot reference previous value of column, unlike a trigger.

 #### Default Constraints
 Most commonly defined at create time, must be uniquely named within a table
 ```
  CREATE TABLE Production.Permits (
      ...
      unitprice money not null  constraint dft_products_unitprice default(0)
      ...
  )


  