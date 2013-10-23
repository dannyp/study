##What is Scaffolding 
 Uses templates to generate boilerplate code for CRUD operations. The default templates can be customised or new
 scaffolding strategies / templates can be downloaded via nuget.
 
 Scaffolding won't build an entire application but it will do the boring file creation bits for you and then you
 can fill in the specific business logic yourself.

###Options
 _Empty Controller_
 Adds a controller with and index method and an empty strongly typed Index view.
 
 _Controller with Empty Read/Write Actions_
 Adds a controller with Index, Details, Create, Edit and Delete actions and appropriate views. The controllers don't 
 actually do anything, you will need to add your own logic to save and retreive data.
 
 _API Controller with Empty Read/Write Actions_
 ApiController derived class
 
 _Controller with Read/Write Actions Using Entity Framework_
 As above but generates code for CRUD operations using EF to store/retrive information in a DB.

## Entity Framework
 - Asp.NET MVC4 projects include a reference to the EF. EF is an ORM framework which can store objects in an RDBMS and 
 retreive the objects with LINQ
 > There is nothing forcing you to use EF in an MVC application.
 - EF is based on a *code-first* style of development. You write POCOs and EF figures out how to store them
 - There are certain conventions which define this how the DB will be structured to fit the code.
 - Eg - Table named after the Object, ID (or ObjectID) will be the PK and sets up an identity auto incrementing column
 - Unless you configure your own DB connection, EF will connect to a localDB instance of Sql Server express.
 
### DbContext Class
 - The gateway to the database.
 - One or more DbSet<Model> properties. Where Model is the type you wish to persist.
 
 
### Lazy Loading and N + 1
  - Use the Include method if you're going to be even thinking about displaying related data.
  - Lazy loading may result in a whole separate query to the DB for each item in a list. So that would be 1 query to get the list and 1 queyr for every item in a list. Yikes.
  
### Database Initialisers
  - Allows EF to recreate an existing database, either everytime the app starts or only on changes in the model.
  - You choose the strategy when you call `SetInitializer()` of EF's Database class.
  - Obviously this is only recommended for development stages where things change prapidly.
  - EF4.3 allows iterative changes to DB based on changes made to the model, so you won't always wind up losing data.
  - Database can be seedded with data by extending the default initialisers
  
## Model Binding
  - Traditionally, when data is posted back to server, you would need to pull values from the request.
  - ModelBinding is generic code which matches input names in posted data to property names of parameters (or param names themselves).
  - ModelBinder uses ValueProviders to search different areas of the request.
  
  ### Explicit Model Binding
  - Implicit model binding (above) is invoked when you have an action with a parameter.
  - ModelBinding can be explicitly invoked using TryUpdateModel() and UpdateModel calls.
  