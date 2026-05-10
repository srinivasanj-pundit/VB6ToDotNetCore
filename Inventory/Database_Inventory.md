# Database Inventory and Analysis

## Database Overview

This document provides a comprehensive inventory of the database schema, objects, relationships, and migration considerations for the VB6 application database.

## Database Specifications

### Database Engine
- **Type**: Microsoft SQL Server
- **Version**: SQL Server 2000/2005
- **Compatibility Level**: SQL Server 2000
- **Collation**: SQL_Latin1_General_CP1_CI_AS
- **Recovery Model**: Full
- **Size**: Approximately 2GB
- **Active Connections**: 50-100 concurrent users

### Database Objects Summary
```
Tables: 45
Views: 15
Stored Procedures: 78
User-Defined Functions: 12
Triggers: 8
Indexes: 67
Constraints: 89
```

## Table Inventory

### Core Business Tables

#### Customers Table
```sql
CREATE TABLE Customers (
    CustomerID nchar(5) NOT NULL PRIMARY KEY,
    CompanyName nvarchar(40) NOT NULL,
    ContactName nvarchar(30) NULL,
    ContactTitle nvarchar(30) NULL,
    Address nvarchar(60) NULL,
    City nvarchar(15) NULL,
    Region nvarchar(15) NULL,
    PostalCode nvarchar(10) NULL,
    Country nvarchar(15) NULL,
    Phone nvarchar(24) NULL,
    Fax nvarchar(24) NULL,
    Email nvarchar(50) NULL,
    CustomerType nvarchar(20) NULL,
    CreditLimit money NULL,
    IsActive bit NOT NULL DEFAULT 1,
    CreatedDate datetime NOT NULL DEFAULT GETDATE(),
    ModifiedDate datetime NULL
)
```
**Row Count**: ~5,000
**Usage**: Customer management, order processing
**Relationships**: FK to Orders, CustomerDemographics
**Indexes**: PK_CustomerID, IX_Customers_CompanyName, IX_Customers_Email

#### Orders Table
```sql
CREATE TABLE Orders (
    OrderID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    CustomerID nchar(5) NULL,
    EmployeeID int NULL,
    OrderDate datetime NULL,
    RequiredDate datetime NULL,
    ShippedDate datetime NULL,
    ShipVia int NULL,
    Freight money NULL DEFAULT 0,
    ShipName nvarchar(40) NULL,
    ShipAddress nvarchar(60) NULL,
    ShipCity nvarchar(15) NULL,
    ShipRegion nvarchar(15) NULL,
    ShipPostalCode nvarchar(10) NULL,
    ShipCountry nvarchar(15) NULL,
    OrderStatus nvarchar(20) NOT NULL DEFAULT 'New',
    TotalAmount money NULL,
    TaxAmount money NULL DEFAULT 0,
    DiscountAmount money NULL DEFAULT 0
)
```
**Row Count**: ~50,000
**Usage**: Order header information
**Relationships**: FK to Customers, Employees, Shippers
**Indexes**: PK_OrderID, IX_Orders_CustomerID, IX_Orders_OrderDate, IX_Orders_OrderStatus

#### OrderDetails Table
```sql
CREATE TABLE OrderDetails (
    OrderID int NOT NULL,
    ProductID int NOT NULL,
    UnitPrice money NOT NULL DEFAULT 0,
    Quantity smallint NOT NULL DEFAULT 0,
    Discount real NOT NULL DEFAULT 0,
    PRIMARY KEY (OrderID, ProductID)
)
```
**Row Count**: ~200,000
**Usage**: Order line items
**Relationships**: FK to Orders, Products
**Indexes**: PK_OrderDetails, IX_OrderDetails_OrderID, IX_OrderDetails_ProductID

#### Products Table
```sql
CREATE TABLE Products (
    ProductID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    ProductName nvarchar(40) NOT NULL,
    SupplierID int NULL,
    CategoryID int NULL,
    QuantityPerUnit nvarchar(20) NULL,
    UnitPrice money NULL DEFAULT 0,
    UnitsInStock smallint NULL DEFAULT 0,
    UnitsOnOrder smallint NULL DEFAULT 0,
    ReorderLevel smallint NULL DEFAULT 0,
    Discontinued bit NOT NULL DEFAULT 0,
    CreatedDate datetime NOT NULL DEFAULT GETDATE(),
    ModifiedDate datetime NULL
)
```
**Row Count**: ~2,000
**Usage**: Product catalog
**Relationships**: FK to Suppliers, Categories
**Indexes**: PK_ProductID, IX_Products_ProductName, IX_Products_CategoryID, IX_Products_Discontinued

### Supporting Tables

#### Employees Table
```sql
CREATE TABLE Employees (
    EmployeeID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    LastName nvarchar(20) NOT NULL,
    FirstName nvarchar(10) NOT NULL,
    Title nvarchar(30) NULL,
    TitleOfCourtesy nvarchar(25) NULL,
    BirthDate datetime NULL,
    HireDate datetime NULL,
    Address nvarchar(60) NULL,
    City nvarchar(15) NULL,
    Region nvarchar(15) NULL,
    PostalCode nvarchar(10) NULL,
    Country nvarchar(15) NULL,
    HomePhone nvarchar(24) NULL,
    Extension nvarchar(4) NULL,
    Photo image NULL,
    Notes ntext NULL,
    ReportsTo int NULL,
    PhotoPath nvarchar(255) NULL
)
```
**Row Count**: ~50
**Usage**: Employee information
**Relationships**: Self-referencing (ReportsTo), FK to EmployeeTerritories

#### Categories Table
```sql
CREATE TABLE Categories (
    CategoryID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    CategoryName nvarchar(15) NOT NULL,
    Description ntext NULL,
    Picture image NULL
)
```
**Row Count**: ~10
**Usage**: Product categorization
**Relationships**: FK from Products

#### Suppliers Table
```sql
CREATE TABLE Suppliers (
    SupplierID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    CompanyName nvarchar(40) NOT NULL,
    ContactName nvarchar(30) NULL,
    ContactTitle nvarchar(30) NULL,
    Address nvarchar(60) NULL,
    City nvarchar(15) NULL,
    Region nvarchar(15) NULL,
    PostalCode nvarchar(10) NULL,
    Country nvarchar(15) NULL,
    Phone nvarchar(24) NULL,
    Fax nvarchar(24) NULL,
    HomePage ntext NULL
)
```
**Row Count**: ~100
**Usage**: Supplier information
**Relationships**: FK from Products

#### Shippers Table
```sql
CREATE TABLE Shippers (
    ShipperID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    CompanyName nvarchar(40) NOT NULL,
    Phone nvarchar(24) NULL
)
```
**Row Count**: ~5
**Usage**: Shipping company information
**Relationships**: FK from Orders

### Reference Tables

#### Regions Table
```sql
CREATE TABLE Regions (
    RegionID int IDENTITY(1,1) NOT NULL PRIMARY KEY,
    RegionDescription nchar(50) NOT NULL
)
```
**Row Count**: ~20
**Usage**: Geographic regions

#### Territories Table
```sql
CREATE TABLE Territories (
    TerritoryID nvarchar(20) NOT NULL PRIMARY KEY,
    TerritoryDescription nchar(50) NOT NULL,
    RegionID int NOT NULL
)
```
**Relationships**: FK to Regions

#### EmployeeTerritories Table
```sql
CREATE TABLE EmployeeTerritories (
    EmployeeID int NOT NULL,
    TerritoryID nvarchar(20) NOT NULL,
    PRIMARY KEY (EmployeeID, TerritoryID)
)
```
**Relationships**: FK to Employees, Territories

#### CustomerDemographics Table
```sql
CREATE TABLE CustomerDemographics (
    CustomerTypeID nchar(10) NOT NULL PRIMARY KEY,
    CustomerDesc ntext NULL
)
```

## View Inventory

### Business Views

#### vw_CustomerOrders
```sql
CREATE VIEW vw_CustomerOrders AS
SELECT
    c.CustomerID,
    c.CompanyName,
    c.ContactName,
    c.City,
    c.Country,
    COUNT(o.OrderID) as OrderCount,
    SUM(o.TotalAmount) as TotalSales,
    MAX(o.OrderDate) as LastOrderDate,
    AVG(o.TotalAmount) as AvgOrderAmount
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE c.IsActive = 1
GROUP BY c.CustomerID, c.CompanyName, c.ContactName, c.City, c.Country
```

#### vw_ProductInventory
```sql
CREATE VIEW vw_ProductInventory AS
SELECT
    p.ProductID,
    p.ProductName,
    p.UnitPrice,
    p.UnitsInStock,
    p.UnitsOnOrder,
    p.ReorderLevel,
    c.CategoryName,
    s.CompanyName as SupplierName,
    CASE
        WHEN p.UnitsInStock <= p.ReorderLevel THEN 'Reorder'
        WHEN p.UnitsInStock = 0 THEN 'Out of Stock'
        ELSE 'In Stock'
    END as StockStatus
FROM Products p
LEFT JOIN Categories c ON p.CategoryID = c.CategoryID
LEFT JOIN Suppliers s ON p.SupplierID = s.SupplierID
WHERE p.Discontinued = 0
```

#### vw_OrderDetails
```sql
CREATE VIEW vw_OrderDetails AS
SELECT
    od.OrderID,
    od.ProductID,
    p.ProductName,
    od.UnitPrice,
    od.Quantity,
    od.Discount,
    (od.UnitPrice * od.Quantity * (1 - od.Discount)) as LineTotal,
    o.OrderDate,
    o.CustomerID,
    c.CompanyName
FROM OrderDetails od
JOIN Orders o ON od.OrderID = o.OrderID
JOIN Products p ON od.ProductID = p.ProductID
JOIN Customers c ON o.CustomerID = c.CustomerID
```

### Reporting Views

#### vw_MonthlySales
```sql
CREATE VIEW vw_MonthlySales AS
SELECT
    YEAR(OrderDate) as SalesYear,
    MONTH(OrderDate) as SalesMonth,
    COUNT(*) as OrderCount,
    SUM(TotalAmount) as TotalSales,
    AVG(TotalAmount) as AvgOrderAmount,
    COUNT(DISTINCT CustomerID) as UniqueCustomers
FROM Orders
WHERE OrderDate IS NOT NULL
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
```

#### vw_SalesByProduct
```sql
CREATE VIEW vw_SalesByProduct AS
SELECT
    p.ProductID,
    p.ProductName,
    c.CategoryName,
    SUM(od.Quantity) as TotalQuantity,
    SUM(od.UnitPrice * od.Quantity * (1 - od.Discount)) as TotalSales,
    COUNT(DISTINCT o.OrderID) as OrderCount,
    AVG(od.UnitPrice * (1 - od.Discount)) as AvgPrice
FROM Products p
JOIN OrderDetails od ON p.ProductID = od.ProductID
JOIN Orders o ON od.OrderID = o.OrderID
LEFT JOIN Categories c ON p.CategoryID = c.CategoryID
GROUP BY p.ProductID, p.ProductName, c.CategoryName
```

## Stored Procedure Inventory

### Order Processing Procedures

#### sp_ProcessOrder
```sql
CREATE PROCEDURE sp_ProcessOrder
    @OrderID int,
    @ProcessType nvarchar(20) = 'Complete'
AS
BEGIN
    -- Order processing logic
    -- Inventory allocation
    -- Status updates
    -- Audit trail
END
```
**Parameters**: @OrderID, @ProcessType
**Usage**: Complete order processing workflow

#### sp_AllocateInventory
```sql
CREATE PROCEDURE sp_AllocateInventory
    @OrderID int
AS
BEGIN
    -- Check inventory availability
    -- Allocate stock for order items
    -- Update UnitsInStock
    -- Handle backorders if necessary
END
```
**Parameters**: @OrderID
**Usage**: Inventory allocation for orders

#### sp_CalculateOrderTotal
```sql
CREATE PROCEDURE sp_CalculateOrderTotal
    @OrderID int,
    @TotalAmount money OUTPUT,
    @TaxAmount money OUTPUT,
    @DiscountAmount money OUTPUT
AS
BEGIN
    -- Calculate subtotal
    -- Apply discounts
    -- Calculate tax
    -- Return totals
END
```
**Parameters**: @OrderID, OUTPUT parameters
**Usage**: Order total calculations

### Reporting Procedures

#### sp_CustomerSalesReport
```sql
CREATE PROCEDURE sp_CustomerSalesReport
    @StartDate datetime,
    @EndDate datetime,
    @CustomerID nchar(5) = NULL
AS
BEGIN
    -- Customer sales analysis
    -- Date range filtering
    -- Customer-specific or all customers
END
```
**Parameters**: @StartDate, @EndDate, @CustomerID
**Usage**: Customer sales reporting

#### sp_ProductSalesReport
```sql
CREATE PROCEDURE sp_ProductSalesReport
    @StartDate datetime,
    @EndDate datetime,
    @CategoryID int = NULL
AS
BEGIN
    -- Product sales analysis
    -- Category filtering
    -- Performance metrics
END
```
**Parameters**: @StartDate, @EndDate, @CategoryID
**Usage**: Product sales reporting

### Maintenance Procedures

#### sp_CleanupOldData
```sql
CREATE PROCEDURE sp_CleanupOldData
    @RetentionDays int = 365
AS
BEGIN
    -- Archive old orders
    -- Remove temporary data
    -- Update statistics
END
```
**Parameters**: @RetentionDays
**Usage**: Data maintenance and archiving

#### sp_UpdateInventoryLevels
```sql
CREATE PROCEDURE sp_UpdateInventoryLevels
AS
BEGIN
    -- Recalculate stock levels
    -- Update reorder flags
    -- Generate reorder alerts
END
```
**Usage**: Inventory level maintenance

## User-Defined Functions

### Scalar Functions

#### fn_CalculateTax
```sql
CREATE FUNCTION fn_CalculateTax
(
    @Amount money,
    @TaxRate decimal(5,4),
    @TaxExempt bit = 0
)
RETURNS money
AS
BEGIN
    IF @TaxExempt = 1
        RETURN 0
    RETURN @Amount * @TaxRate
END
```
**Usage**: Tax calculations

#### fn_FormatPhoneNumber
```sql
CREATE FUNCTION fn_FormatPhoneNumber
(
    @Phone nvarchar(24)
)
RETURNS nvarchar(24)
AS
BEGIN
    -- Phone number formatting logic
    RETURN formatted_number
END
```
**Usage**: Phone number formatting

### Table-Valued Functions

#### fn_GetCustomerOrders
```sql
CREATE FUNCTION fn_GetCustomerOrders
(
    @CustomerID nchar(5),
    @StartDate datetime = NULL,
    @EndDate datetime = NULL
)
RETURNS TABLE
AS
RETURN
(
    SELECT * FROM Orders
    WHERE CustomerID = @CustomerID
    AND (@StartDate IS NULL OR OrderDate >= @StartDate)
    AND (@EndDate IS NULL OR OrderDate <= @EndDate)
)
```
**Usage**: Customer order retrieval

## Trigger Inventory

### Data Integrity Triggers

#### trg_OrderStatusUpdate
```sql
CREATE TRIGGER trg_OrderStatusUpdate
ON Orders
AFTER UPDATE
AS
BEGIN
    -- Status change validation
    -- Audit trail logging
    -- Business rule enforcement
END
```
**Purpose**: Order status change tracking

#### trg_InventoryUpdate
```sql
CREATE TRIGGER trg_InventoryUpdate
ON Products
AFTER UPDATE
AS
BEGIN
    -- Stock level monitoring
    -- Reorder alert generation
    -- Inventory audit trail
END
```
**Purpose**: Inventory change tracking

### Audit Triggers

#### trg_CustomerAudit
```sql
CREATE TRIGGER trg_CustomerAudit
ON Customers
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    -- Customer change logging
    -- Audit trail maintenance
END
```
**Purpose**: Customer data auditing

## Index Inventory

### Primary Key Indexes
- PK_Customers (CustomerID)
- PK_Orders (OrderID)
- PK_OrderDetails (OrderID, ProductID)
- PK_Products (ProductID)
- PK_Employees (EmployeeID)
- PK_Categories (CategoryID)
- PK_Suppliers (SupplierID)
- PK_Shippers (ShipperID)

### Foreign Key Indexes
- IX_Orders_CustomerID (Orders.CustomerID)
- IX_Orders_EmployeeID (Orders.EmployeeID)
- IX_OrderDetails_ProductID (OrderDetails.ProductID)
- IX_Products_SupplierID (Products.SupplierID)
- IX_Products_CategoryID (Products.CategoryID)

### Performance Indexes
- IX_Orders_OrderDate (Orders.OrderDate)
- IX_Orders_OrderStatus (Orders.OrderStatus)
- IX_Customers_CompanyName (Customers.CompanyName)
- IX_Customers_Email (Customers.Email)
- IX_Products_ProductName (Products.ProductName)
- IX_Products_Discontinued (Products.Discontinued)

## Constraint Inventory

### Check Constraints
```sql
CK_Orders_Freight CHECK (Freight >= 0)
CK_Products_UnitPrice CHECK (UnitPrice >= 0)
CK_OrderDetails_Quantity CHECK (Quantity > 0)
CK_OrderDetails_Discount CHECK (Discount BETWEEN 0 AND 1)
```

### Default Constraints
```sql
DF_Orders_OrderStatus DEFAULT 'New'
DF_Orders_Freight DEFAULT 0
DF_Products_Discontinued DEFAULT 0
DF_Customers_IsActive DEFAULT 1
```

### Foreign Key Constraints
```sql
FK_Orders_Customers (CustomerID → Customers.CustomerID)
FK_Orders_Employees (EmployeeID → Employees.EmployeeID)
FK_Orders_Shippers (ShipVia → Shippers.ShipperID)
FK_OrderDetails_Orders (OrderID → Orders.OrderID)
FK_OrderDetails_Products (ProductID → Products.ProductID)
FK_Products_Suppliers (SupplierID → Suppliers.SupplierID)
FK_Products_Categories (CategoryID → Categories.CategoryID)
```

## Data Volume and Growth

### Current Data Volumes
```
Customers: 5,000 records (~2MB)
Orders: 50,000 records (~15MB)
OrderDetails: 200,000 records (~25MB)
Products: 2,000 records (~1MB)
Employees: 50 records (~0.5MB)
Other Tables: ~5MB total
Total Database Size: ~50MB (plus indexes and logs)
```

### Growth Projections
```
Annual Growth Rates:
- Orders: +20% (10,000 orders/year)
- OrderDetails: +20% (40,000 lines/year)
- Customers: +10% (500 customers/year)
- Products: +5% (100 products/year)

Projected Size (5 years): ~200MB
```

## Migration Considerations

### Schema Migration Challenges

#### Data Type Considerations
```
Legacy Types → .NET Core/EF Core Types:
- nchar(5) → string (fixed length)
- nvarchar(max) → string
- money → decimal
- datetime → DateTime
- bit → bool
- image → byte[]
- ntext → string
```

#### Identity Column Handling
- IDENTITY columns → Auto-increment in EF Core
- Custom seed values preservation
- Replication considerations

#### Computed Columns
- Database computed columns → EF Core computed properties
- Formula preservation
- Performance implications

### Data Migration Strategy

#### Migration Phases
1. **Schema Migration**: Create new schema in SQL Server 2022
2. **Data Export**: Extract data from SQL Server 2000/2005
3. **Data Transformation**: Clean and transform data
4. **Data Import**: Load data into new schema
5. **Validation**: Verify data integrity
6. **Cutover**: Switch to new database

#### Data Quality Issues
- **Orphaned Records**: FK constraint violations
- **Invalid Data**: Constraint violations
- **Missing Data**: NULL value handling
- **Duplicate Data**: Uniqueness constraint violations

#### Migration Tools
- **SSIS**: SQL Server Integration Services
- **bcp**: Bulk copy program
- **Custom Scripts**: Application-specific migration logic
- **EF Core Migrations**: Schema change management

### Performance Considerations

#### Index Optimization
- Existing indexes evaluation
- New index requirements for EF Core
- Query performance monitoring
- Index maintenance strategy

#### Query Optimization
- Stored procedure conversion to LINQ/EF queries
- Parameterized query usage
- Connection pooling configuration
- Query execution plan analysis

### Security Considerations

#### Authentication Migration
- SQL Server authentication → Azure AD/Integrated
- Connection string security
- Password policy updates
- Audit trail preservation

#### Data Protection
- Sensitive data encryption
- PII data handling
- Compliance requirements (GDPR, etc.)
- Data masking for non-production

### Backup and Recovery

#### Backup Strategy
- Full backups: Weekly
- Differential backups: Daily
- Transaction log backups: Hourly
- Backup retention: 30 days
- Offsite backup storage

#### Recovery Requirements
- RTO (Recovery Time Objective): 4 hours
- RPO (Recovery Point Objective): 1 hour
- Disaster recovery plan
- Business continuity procedures

This comprehensive database inventory provides the foundation for planning the database migration strategy and ensuring data integrity throughout the .NET Core conversion process.