#Views
The ambasador between the user and the controller.

 ##Overview
  - Contained in the `~/views` folder
  - Convention: beneath /views is a folder for each controller and view files (.cshtml if razor) for each action.
  - Returning the result of View() from controller causes convention to be followed
  - Can be overridden with View("name") or View("~/path/to/view.cshtml")
  - You must include the full path + extension when using the tilde syntax.
 
 ##ViewData and Viewbag
  - ViewData is a ViewDataDictionary. 
  - This is how data can be passed to a view.
  - ViewBag is a dynanmic wrapper around viewdata. Available in MVC 3+ because of the dynamic keyword in C#4
  - `ViewBag.CurrentTime = DateTime.Now` is akin to saying `ViewData["CurrentTime"]...`
 
 ###Notes:
  - In the case of ViewBag the key must be a valid C# identifier. Whereas in viewdata it can be any string.
  - Dynamic values cannot be passed in as parameters to extension methods. Complier must know type of parameter at compile time.
  - eg. this will fail `@Html.TextBox("Name", ViewBag.Name)` -> either use `ViewData["Name"]` or `(string)ViewBag.Name`
  
 ##Strongly Typed views.
  - Using ViewBag you need to cast or if you use `dynamic` lose intellisense.
  - There is an overload of View() in controller that allows you to specify a Model.
  - This sets the value of ViewData.Model
  - You can specify the type of the model using the @model keyword in the view eg. `@model MvcApplication.Models.PageModel`
  - You can use a @using statement to avoid the fully qualified type, or put the namespace in Web.config System.Web.Webpages.Razor > Pages > Namespaces
  
 ##Razor
  - Razor will automatically work out when to transition between HTML and code.
  - @ Character transitions from HTML to code. The end of a valid identifier transitions back
  - In ambiguous cases, you can explicitly wrap code in @(blah).
  - Alternately you can escape @ characters by doing @@Blah for example - twitter usernames.
  - Razor expressions are automatically HTML encoded (to avoid XSS)
  - In cases where you need to show the markup, use Html.Raw  
  - Alternately use code blocks @ { ... } for well, blocks of code.
  
 ### Warning:
  This is not enough to protect you from XSS attacks when dynamically setting values in JS. You should in that case ust @Ajax.JavaScripStringEncode
  
 ## Layouts
  - They are master pages.
  - Use combination of `@RenderBody()` and `@RenderSection("Footer")` to insert content.