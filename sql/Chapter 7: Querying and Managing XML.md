# Querying with XML
## Lesson 1: Returning Results as XML with FOR XML
 * XML must have a single root node
 * It is made up of a heirarchy of elements with attributes

The T-SQL select statement can be used to create XML using the FOR XML clause in a SELECT.
You should always use an ORDER BY to give the XML document consistent formatting
The FOR XML comes after the ORDER BY.
SQL Server uses the order of the columns to determine the nesting structure of the XML.
In RAW and AUTO mode you can also return the XSD for the XML document. If you want just the XSD, then rig the query to return an empty set.

#### FOR XML RAW
 - The XML created is quite close to the tabluar presentation of the data.
 - XML document has `<row>` elements for each line with attributes for each column
 - You can enhance it be renaming the row element and adding a root.

#### FOR XML AUTO
 - Gives you nice xml documents with nested elements
 - Simple to use
 - You can use the ELEMENTS word to produce element centric XML
 - A namespace can be specified and aliased to tables in the query
 
#### FOR XML PATH (and EXPLICIT)
 - Manually define the xml returned.
 - EXPLICIT is only included for backwards compatibility. It uses T-SQL specific syntax whereas PATH is SQL standard
 - In PATH mode, column names and aliases are XPath expressions.
 - - eg Description as 'Invoice/Description' or '@Description' for an attribute.
 - If you want to create XML with nested elements for child tables you need to:
 - * You need to use a subquery that returns a FOR XML ___, TYPE in the projection of the outer query.

 ### Shredding XML to Tables.
  Now that we've looked at table to XML, we can also do the opposite.
  The OPENXML function provides a rowset over in-memory XML documents.
  OPENXML uses a DOM document, so you need to prepare your XML document beforehand.
   - sys.sp_xml_preparedocument stored proc
   - then sys.sp_xml_removedocument to shred the DOM.

   OPENXML requires the following parameters
   * An XML DOM document handle (int)
   * XPath expression to find nodes that map to your rowset (rowpattern)
   * Description of the rowset returned.
   * Mapping between xml nodes and rowset columns - 

    The description of the rowset is the WITH clause of the OPENXML statement, it is structured like a CREATE TABLE statement.
    The mapping between the xml nodes and rowset columns is the optional 'flags' paremeter.
    It's basically a bitmap where 1 - Attribute mapping, 10 - Element mapping, 11 - Both (but behaviour is undefined), 1011 (logical OR of 1000 and 10 and 1). 

## Lesson 2: Querying XML Data with XQuery
 XQuery is a standard language for browsing XML and returning XML.
 It is much richer than XPath. It allows for looping and shaping the returned XML as well as just browsing it.
 Some XQuery features are not available in the SQL server engine.

 Case sensitive (like XML)
 XQuery returns sequences that include *atomic* or *complex* values.

 If you have an @x as XML, you can use @x.query([xquery]) to extract xml.

 Every identifier in XQuery is called a *qualified name* or **QName**. A QName is made up of a local name and an optional prefix. 

 The following standard prefixes are defined in SQL server:
  * xs - The namespace for an XML schema.
  * xsi - XML schema insance namespace, associates XML Schemas with instance documents
  * xdt - The namespace for XPath and XQuery
  * fn - The functions namespace
  * sqltypes - Mappings for SQL Server types
  * xml - Default XML namespace

 These standard preixes can be used in queries without needing to be defined. Our own datatypes are defined in the prolog at the beggining of the XQuery.

### XQuery Data Types
 There are about 50 types. Most of which you will never use.
 Some of the important **node** types are attribute, comment, element, namespace, text,processing-instruction and document-node.
 Some of the important **atomic** types are xs:QName, xs:date, xs:time, xs:datetime, xs:float,xs:double, xs:decimal, xs:integer.

#### XQuery Functions
 Similarly, there are dozens of functions in XQuery,
 * **Numeric functions:** ceiling(), floor(), round()
 * **String functions:** concat(), contains(), substring(), string-length(), lower-case() and upper-case()
 * **Boolean functions:** not(), true(), false()
 * **Nodes functions:** local-name() and namespace-uri()
 * **Aggregate functions:** count(), min(), max(), avg(), sum().
 * **Data accessor functions:** data(), string()
 * **SQL Server extensions functions:** sql:column(), sql:variable()

 #### Navigation
  Basic navigation can be acheived through XPath expressions, though there are many more approaches possible within XQuery.
  XPath allows us to specify a path in absolute terms or relative to the current node.

  Paths consist of a series of steps, e.g: ` Node-name / child::element-name[@attribute-name=value]`

  Steps are separated by slashes.
   - **Axis** specifies the direction of travel, (*child::* in the example above)
   - **Node test** Specifies the nodes to select, when there are multiple options (*element-name*)
   - **Predicate** Narrows down the nodes selected by by node test (*[@attribute-name=vale]*)

   @ is a shorthand for the attribute:: axis.
   Supported axes in SQL Server: child::, descendant::, self::, descendend-or-self:: (//), attribute:: (@), parent::(..)
   Wildcards can be used in node tests, e.g. '*' = any pricipal node. You can also narrow down to a particular namespace eg ` ns:* ` or a particular element in any namespace `*:element`.

   There are also node kind tests:
    * comment()
    * node()
    * processing-instructions()
    * text()

### Predicates
 Basic predicates include numeric and boolean predicates.
 Numeric predicates select node by position eg `/x/y[1]` -> selects the first y under x. You could also use them over a whole path eg `(/x/y)[1]` -> selects first occurence path /x/y.
 Boolean predicates select all nodes where it evaluates to true.
 Comparison operators work over both atomic values and sequences. A comparison involving a sequence evaluates to true if any part of the left matches any part of the right.

 For example:
 ```
    (1, 2, 3) = 1 -> true
    (1, 2, 3) = (5,6) -> false
    (1, 2, 3) != 1 -> true, becase 2 and 3 != 1 so the whole thing evaluates to true.
 ```
 The value comparison operators work in a more familar way, but they only work on single value arrays.
 
 XQuery also supports if then else expressions (they work more like a CASE statement in TSQL than a tradidional if/then/else)

### FLWOR Expressions
 - for, let, where, order by, return
 - The real power of XQuery.
 - Works over XML or any sequence.

#### The parts of a FLWOR Statement
 * **For:** - Bind iterator variables to input sequence.
 * **Let:** - Assign value to variable for a specific iteration.
 * **Where:** - Filter the iteration
 * **Order by:** - Control the order that elements are processed
 * **Return:** - Evaluated once per iteration, format the resulting xml here.

 **Notes:**
 - The type returned to the orderby clause must be compatible with the gt operator (so it can be sorted!) which means it needs to be atomic, so we specify the index in the orderdate sequence.
 - XQuery uses the braces to delimit what needs to be evaluated.

## Lesson 3: Using the XML Data Type
 XML is a standard for exchange of information.
 Sometimes it is too much work to decompose the XML and store it relationally.

### When to Use the XML Data Type
 Sometimes schemas can be volatile, requiring different types of related data records that have the potential for a high level of heterogeneitity.
  - e.g. different DDL extended event types, medical clinical systems, sparse data.
 
 It's not that you couldn't model these systems in a normal relational model, it might just lead you to a model with lots of different tables which becomes difficult to manage.

 Also useful for inherently heirarchical data or inherently ordered data.

### XML Data Type Methods
 - query()
 - value() 
 - - Accepts an XQuery expression and returns a scalar (must specify position)
 - exist()
 - - Returns 1, 0, NULL based on whether the XQuery returns a non-empty, empty or NULL result
 - modify()
 - - Invoke in an update statememnt to in-place modify a large XML data value without having to replace the whole thing.
 - nodes()
 - - Same purpose as OPENXL rowset function. But much faster.
 - - Returns a row for each node from the starting point of the XQuery expression.
 - - You can APPLY from one table to a nodes function, as it needs to be executed row by row.


 ### Using the XML Data Type for Dynamic Schema.
  - You can store data as XML in a column with the XML type
  - By default any xml can be stored, however we are able to create a schema which will be used to validate data being stored in the column
  - One schema is only so useful (ie - if the data has a consistent structure, why bother with xml in the first place), so we are able to create schema collections.
  - The easiest way to create schemas and therefore schema collections is to create the objects as tables, query them with FOR XML AUTO / RAW and generate the schemas for them.
  - You then save the result as text and use it to create the schema collection.
  - It is also advisable to add a check constraint for data going in to match namespaces to the predefined schemas (so you don't bind one type to the schema of another type).
 
  ### XML Indexes
   XML type is very large (up to 2GB). 
    * **Primary XML index** First index created, shreds the data into a persisted representation. Approx same number of rows as nodes. Even by itself, will speed up searches using exist().
    * **Value** Useful if queries are value based and path not known
    * **Path** Use if the queries specify path expressions, better for speeding up exist() than the primary index. Also speeds up value()
    * **Property** Useful for queries that retrieve one or more values from individual indexes using the value() method.
    The underlying table also has to have a clustered primary key.

