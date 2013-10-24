Validation should happen in the browser for conveneience and *must* happen on the server side for security.

## Validation Annotations
#### Required
  Server and clientside validation logic 
  
#### StringLength
  Used to ensure strings fit in the database by limiting the max number of characters. Can also provide a minimum.

#### RegularExpression
  Validates content agains a regex

#### Range
  Number falls between upper and lower limits.

## Other Validation Attributes
#### Remote
  Client side only validation with a server callback. Specify an action which returns JSON true/false for valid/invalid.

#### Compare
  Two properties on a model have the same value. Eg password or email check.

  Error messages can be customised with parameters to the attributes. Similarly they can be localised by specifying resource names.

### Validation and Model Binding
  Validation occurs when the model binding is happening. Errors are placed into the modelstate
  
 