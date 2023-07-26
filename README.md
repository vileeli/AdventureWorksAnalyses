# Adventure Works Sales Dashboard
## Dashboard
Dashboard done in Looker Studio can be found [here.](https://lookerstudio.google.com/reporting/f95afd27-3915-4243-94b1-1214a94ddd3a)
For analysis I used data 2003 January-June and compared it with 2004 January-June. 
Main KPIs are compared 2004 Jan-Jun to 2003 Jan-Jun. 
## Dataset
This dashboard was made using Adventure Works public dataset. 
Dataset schema can be found [here.](https://i0.wp.com/improveandrepeat.com/wp-content/uploads/2018/12/AdvWorksOLTPSchemaVisio.png?ssl=1)

## Project Task
Executive Leadership and Sales
You are asked to provide 1 dashboard or spreadsheet which you will then use for 2 presentations for each of the departments. If you wish, you can have 2 slightly modified versions of the dashboard / spreadsheet to better suit each of the two presentations.

## Preparing table
I have joined all needed tables, created some additional columns.

```
WITH
  main AS (
  SELECT orders.SalesOrderID,
         OrderDate,
         SalesPersonID,
         CONCAT(Firstname, ' ', LastName) AS Seller,
         CAST(orders.SalesPersonID AS STRING) AS SalesPerson,
         OrderQty,
         LineTotal,
         product.ProductID,
         product.Name AS Product,
         subcategory.Name AS SubCategory,
         category.Name AS Category,
         territory.CountryRegionCode AS Country,
         reason.Name AS Reason
  FROM `adwentureworks_db.salesorderheader` orders
  JOIN `adwentureworks_db.salesorderdetail` orderdetail
    ON orderdetail.SalesOrderID = orders.SalesOrderID
  JOIN `adwentureworks_db.product` product
    ON product.ProductID = orderdetail.ProductID
  JOIN `adwentureworks_db.productsubcategory` subcategory
    ON subcategory.ProductSubcategoryID = product.ProductSubcategoryID
  JOIN `adwentureworks_db.productcategory` category
    ON category.ProductCategoryID = subcategory.ProductCategoryID
  JOIN `adwentureworks_db.salesterritory` territory
    ON territory.TerritoryID = orders.TerritoryID
  LEFT JOIN `adwentureworks_db.salesorderheadersalesreason` headerreason
    ON headerreason.SalesOrderID = orders.SalesOrderID
  LEFT JOIN `adwentureworks_db.salesreason` reason
    ON reason.SalesReasonID= headerreason.SalesReasonID
  LEFT JOIN `adwentureworks_db.employee` employee
    ON orders.SalesPersonID = employee.EmployeeId
  LEFT JOIN `adwentureworks_db.contact` contacts
    ON contacts.ContactId = employee.ContactId
  WHERE (OrderDate BETWEEN '2003-01-01' AND '2003-06-30') OR (OrderDate BETWEEN '2004-01-01' AND '2004-06-30')),
 first_purchase AS (
 SELECT DISTINCT(CustomerID),
       orders.SalesOrderID,
       OrderDate,
       MIN(OrderDate) OVER (PARTITION BY CustomerID) AS first_purchase_date
 FROM `adwentureworks_db.salesorderheader` orders
 WHERE (OrderDate BETWEEN '2003-01-01' AND '2003-06-30') OR (OrderDate BETWEEN '2004-01-01' AND '2004-06-30'))
SELECT main.SalesOrderID,
       main.OrderDate,
       first_purchase_date,
       SalesPersonID,
       CASE
         WHEN Seller IS NULL THEN 'Online'
         ELSE Seller
       END AS Seller,
       OrderQty,
       LineTotal AS Revenue,
       ProductID,
       Product,
       SubCategory,
       Category,
       CustomerID,
       CASE
         WHEN DENSE_RANK() OVER (PARTITION BY CustomerID ORDER BY main.OrderDate) = 1 
         THEN 'New Customer'
         ELSE 'Returning Customer'
       END AS CustomerType,
       CASE
         WHEN Country IN ('US') THEN 'United States'
         WHEN Country IN ('DE') THEN 'Germany'
         WHEN Country IN ('AU') THEN 'Australia'
         WHEN Country IN ('FR') THEN 'France'
         WHEN Country IN ('CA') THEN 'Canada'
         WHEN Country IN ('GB') THEN 'Great Britain'
         ELSE Country
       END Country,
       Reason
FROM main
JOIN first_purchase
ON main.SalesOrderID = first_purchase.SalesOrderID
```

Final table results: 
<img width="1000" alt="Capture" src="https://github.com/TuringCollegeSubmissions/vlysen-CAR..2/assets/116706695/377ed354-3bfc-4550-a362-92b01f41c5e4">


