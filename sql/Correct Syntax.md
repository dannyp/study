# Correct Syntax for Various Types of Query
## MERGE
```
MERGE Products as TARGET
USING NewProducts AS SOURCE
 ON TARGET.ProductID = SOURCE.ProductID
WHEN NOT MATCHED BY SOURCE THEN DELETE
WHEN NOT MATCHED BY TARGET THEN 
    INSERT(Name, Price) VALUES (SOURCE.Name, SOURCE.Price)
WHEN MATCHED 
    AND (TARGET.Name <> SOURCE.Name)
    AND (TARGET.Price <> SOURCE.Price)
    UPDATE SET 
        TARGET.Price = Source.Price, 
        TARGET.Name = SOURCE.Name
OUTPUT 
    @action, deleted.ProductID, inserted.ProductId
;
```

## Indexed View
```
CREATE VIEW Sales.OrderView
WITH SCHEMABINDING
AS
SELECT ....
CREATE UNIQUE CLUSTERED INDEX CIDX_OrderView ON Sales.OrderView(CustId, OrderID)
CREATE NONCLUSTERED INDEX IX_OrderViewCustomer ON Sales.OrderView(CustId, customername)
;
```

## Table Valued Function
```
CREATE FUNCTION dbo.ufnGetContactInformation(@ContactID int)
RETURNS @retContactInformation TABLE (
    ContactID int PRIMARY KEY NOT NULL,
    FirstName nvarchar(50) NULL,
    LastName nvarchar(50) NULL
)
AS
BEGIN
    SELECT @ContactID = ...,
        @FirstName = ...
        @LastName = ...
        FROM Person.Person
        WHERE BusinessEntityID = @ContactId
    RETURN;
END;
```

## UNION
 * Name columns
 * Check for allows repeats or requires distinct
```
SELECT FirstName + ' ' + LastName as FullName 
    FROM Customers
UNION [ALL]
Select FullName from MoreData
```

## .WRITE(expr, offset, length)
```
UPDATE Customer
    SET CustomerName.WRITE('_obsolete', 3, 9) -- Trim existing to 3 chars and append '_obsolete'
    , SET CustomerName.WRITE('_obsolete', 0, null) -- Replace name with _obsolete
    , SET CustomerName.WRITE('_obsolete', null, 0) -- appends _obsolete to existing customernane
```

## Add a Computed Column that calls a funtion
```
ALTER TABLE Sales.Customer
    ADD TotalSales AS dbo.udfCalcTotalSales(custid) PERSIST
```
