#Forms and HTML Helpers
1. Understanding forms
2. Making HTML helpers work for you
3. Editing and inputting helpers
4. Displaying and rendering helpers

## When to GET or POST
 - GET requests represent read only operations which you don't mind if they are repeated
 - POST used to submit transactions or other data-modifying actions
 -- Browsers will sometimes help users from double-posting.
 
## HTML Helpers
 - Methods which are invoced on the `Html` property of a view.
 - Most helpers output HTML markup. eg. `@Html.BeginForm("Action", "Controller". FormMethod.Get).....`
 - They encode the output to help avoid XSS
 - Able to override default attributes (use @class to set class and '_' in place of '-', such that data_dash translate to data-dash in html)
 - The majority of helpers are definted as Extension methods.
 -- Make sure `System.Web.Mvc.Html` is in scope
 -- You can build your own helpers methods simply by writing new extension methods.
 - There are *Strongly Typed Helpers* which you pass a lambda expression to to specify the model property for rendering
 -- Intellisense, compile time warnings, easier refactoring.
 -- eg `@Html.DisplayFor(model => model.Name)`
 - Helpers take advantage of model metadata (eg. displayname, data type, validation)
 - Templated helpers - `Html.EditorFor` and `Html.DisplayFor` will choose the applicable template to render. You can override the defaults too.
 - Helpers render form fields such they will be Model bound correctly. Including populating model state with errors to then be displayed.
 
 ### Input Helpers
  - Html.Hidden
  - Html.TextBox
  - Html.DropDownList
  - Html.Password
  - Html.RadioButton
  - Html.CheckBox
  
 ### Rendering Helpers
  - Html.ActionLink
  - Html.RouteLink
  - Html.Partial : renders partial into a string. Preferred for convenience.
  - Html.RenderPartial : writes directly to the response output stream. May offer better performance.
  - Html.Action : (see above for difference)
  - Html.RenderAction (see above for difference)
  
  There is also an attribute on Controller actions [IsChildAction] which will prevent it being called from a URL and only allow it to be called from the 
  RenderAction or Action methods.
  
 ### Url Helpers
  - Url.Action
  - Url.Content
  - Url.RouteUrl
 