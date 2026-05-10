# Component Inventory

## Application Components Catalog

This document provides a comprehensive inventory of all components, modules, forms, and external dependencies identified in the VB6 application.

## Forms Inventory

### Main Application Forms

#### frmMain (Main Application Window)
```
Form Type: MDI Parent Form
File: frmMain.frm
Lines of Code: 1,250
Controls: 45
Description: Main application container with menu system and MDI interface

Key Components:
- Menu Bar: File, Edit, View, Tools, Help
- Toolbar: Common actions (New, Save, Delete, Print)
- Status Bar: Application status and user information
- MDI Client Area: Container for child forms

Dependencies:
- modSecurity.bas (access control)
- modCommon.bas (utility functions)
- All child forms

Event Handlers:
- Form_Load: Initialize application, check user permissions
- Form_Unload: Cleanup resources, save user preferences
- mnuFileExit_Click: Application exit with confirmation
- Toolbar button clicks: Delegate to appropriate child forms
```

#### frmCustomerEntry (Customer Management)
```
Form Type: MDI Child Form
File: frmCustomerEntry.frm
Lines of Code: 890
Controls: 32
Description: Customer information entry and maintenance form

Key Components:
- Customer search and filter controls
- Customer detail input fields (name, address, contact info)
- Credit limit and payment terms
- Customer type selection
- Validation and error display

Business Logic:
- Customer validation rules
- Duplicate detection
- Credit limit calculations
- Address validation

Database Operations:
- SELECT: Customer search and retrieval
- INSERT: New customer creation
- UPDATE: Customer information updates
- DELETE: Customer deactivation (soft delete)
```

#### frmOrderEntry (Order Processing)
```
Form Type: MDI Child Form
File: frmOrderEntry.frm
Lines of Code: 1,120
Controls: 38
Description: Order creation and management form

Key Components:
- Customer selection and lookup
- Product selection with inventory check
- Order item management (add, edit, remove)
- Pricing calculations and discounts
- Shipping information
- Order status tracking

Business Logic:
- Order total calculations
- Inventory availability checks
- Credit limit validation
- Tax calculations
- Shipping cost estimation

Database Operations:
- SELECT: Customer and product data retrieval
- INSERT: Order and order detail creation
- UPDATE: Order modifications and status updates
- Complex transactions for inventory allocation
```

#### frmReports (Reporting Interface)
```
Form Type: MDI Child Form
File: frmReports.frm
Lines of Code: 650
Controls: 15
Description: Report generation and viewing interface

Key Components:
- Report type selection
- Parameter input controls
- Report preview area
- Export options (PDF, Excel, Print)
- Report scheduling

Integration:
- Crystal Reports 8.5 integration
- Database query execution
- File system operations for export

Reports Generated:
- Customer lists and summaries
- Order reports and analytics
- Inventory status reports
- Sales performance reports
- Financial reports
```

### Dialog Forms

#### frmLogin (Authentication)
```
Form Type: Modal Dialog
File: frmLogin.frm
Lines of Code: 180
Controls: 8
Description: User authentication dialog

Key Components:
- Username input
- Password input (masked)
- Login button
- Cancel button
- Remember me option

Security Features:
- Password masking
- Failed attempt tracking
- Account lockout after 3 failures
- Session timeout handling
```

#### frmCustomerSearch (Customer Lookup)
```
Form Type: Modal Dialog
File: frmCustomerSearch.frm
Lines of Code: 220
Controls: 12
Description: Customer search and selection dialog

Key Components:
- Search criteria inputs (name, phone, email)
- Results grid display
- Select and Cancel buttons
- Advanced search options

Functionality:
- Real-time search as user types
- Multiple selection modes
- Recent customer history
- Customer detail preview
```

#### frmProductSearch (Product Lookup)
```
Form Type: Modal Dialog
File: frmProductSearch.frm
Lines of Code: 195
Controls: 10
Description: Product search and selection dialog

Key Components:
- Product name/category search
- Inventory status filtering
- Price range filtering
- Results display with stock levels
- Select and Cancel buttons

Business Logic:
- Stock availability checking
- Pricing rule application
- Product status validation
```

## Module Inventory

### Business Logic Modules

#### modBusinessLogic.bas (Core Business Rules)
```
Lines of Code: 8,500
Functions: 85
Description: Central business logic module containing core application rules

Key Functions:
- CalculateOrderTotal: Order pricing and discount calculations
- ValidateCustomer: Customer data validation
- ProcessOrder: Complete order processing workflow
- UpdateInventory: Stock level management
- CalculateTax: Tax computation by jurisdiction
- ApplyDiscounts: Discount rule processing
- ValidateCreditLimit: Credit availability checking

Dependencies:
- modDatabase.bas (data access)
- modValidation.bas (input validation)
- modCommon.bas (utilities)
```

#### modValidation.bas (Input Validation)
```
Lines of Code: 3,100
Functions: 60
Description: Input validation and business rule enforcement

Key Functions:
- ValidateEmail: Email format validation
- ValidatePhone: Phone number format validation
- ValidatePostalCode: Postal code validation by country
- ValidateCreditCard: Payment card validation
- ValidateAddress: Address format validation
- ValidateDate: Date format and range validation
- ValidateNumeric: Numeric input validation with ranges

Validation Rules:
- Required field checking
- Data type validation
- Format pattern matching
- Business rule validation
- Cross-field validation
```

#### modDatabase.bas (Data Access)
```
Lines of Code: 4,200
Functions: 45
Description: Database connection and query execution module

Key Functions:
- OpenDatabase: Database connection management
- ExecuteQuery: SQL query execution
- ExecuteStoredProcedure: Stored procedure calls
- BeginTransaction: Transaction management
- CommitTransaction: Transaction completion
- RollbackTransaction: Transaction reversal
- GetConnectionString: Connection string management

Database Operations:
- Connection pooling
- Error handling and retry logic
- Parameterized query execution
- Result set processing
- Connection cleanup
```

### Utility Modules

#### modCommon.bas (Common Utilities)
```
Lines of Code: 2,850
Functions: 40
Description: Common utility functions used across the application

Key Functions:
- FormatCurrency: Currency formatting
- FormatDate: Date formatting
- FormatPhone: Phone number formatting
- EncryptData: Basic data encryption
- DecryptData: Data decryption
- LogError: Error logging
- SendEmail: Email sending functionality
- ExportToExcel: Excel export utilities

Utilities:
- String manipulation functions
- Date/time utilities
- File system operations
- Registry access functions
- System information retrieval
```

#### modSecurity.bas (Security Module)
```
Lines of Code: 1,200
Functions: 25
Description: Application security and access control

Key Functions:
- AuthenticateUser: User authentication
- AuthorizeAction: Permission checking
- HashPassword: Password hashing
- ValidateSession: Session validation
- LogSecurityEvent: Security event logging
- EncryptSensitiveData: Sensitive data encryption
- CheckUserRole: Role-based access control

Security Features:
- User authentication
- Role-based permissions
- Session management
- Audit trail logging
- Data encryption
- Password policies
```

## Class Inventory

### Business Entity Classes

#### clsCustomer (Customer Entity)
```
Lines of Code: 2,100
Properties: 18
Methods: 15
Description: Customer business entity class

Properties:
- CustomerID, CompanyName, ContactName, ContactTitle
- Address, City, Region, PostalCode, Country
- Phone, Fax, Email, CustomerType, CreditLimit
- IsActive, CreatedDate, ModifiedDate

Methods:
- Load: Load customer from database
- Save: Save customer to database
- Validate: Validate customer data
- Delete: Deactivate customer
- GetOrders: Retrieve customer orders
- UpdateCreditLimit: Modify credit limit
- CheckCreditAvailability: Check available credit
```

#### clsOrder (Order Entity)
```
Lines of Code: 2,800
Properties: 22
Methods: 20
Description: Order business entity class

Properties:
- OrderID, CustomerID, EmployeeID, OrderDate
- RequiredDate, ShippedDate, ShipVia, Freight
- ShipName, ShipAddress, ShipCity, ShipRegion
- ShipPostalCode, ShipCountry, OrderStatus
- TotalAmount, TaxAmount, DiscountAmount

Methods:
- Load: Load order from database
- Save: Save order to database
- AddItem: Add order item
- RemoveItem: Remove order item
- CalculateTotal: Calculate order total
- ProcessPayment: Process order payment
- ShipOrder: Mark order as shipped
- CancelOrder: Cancel order
```

#### clsInventory (Inventory Entity)
```
Lines of Code: 1,960
Properties: 12
Methods: 18
Description: Inventory management class

Properties:
- ProductID, ProductName, SupplierID, CategoryID
- QuantityPerUnit, UnitPrice, UnitsInStock, UnitsOnOrder
- ReorderLevel, Discontinued, CreatedDate, ModifiedDate

Methods:
- Load: Load product from database
- Save: Save product to database
- AdjustStock: Modify stock levels
- CheckAvailability: Check stock availability
- PlaceReorder: Initiate reorder process
- UpdatePrice: Modify product price
- Discontinue: Mark product as discontinued
- GetStockHistory: Retrieve stock change history
```

## External Dependencies

### Third-Party Components

#### Crystal Reports 8.5
```
Purpose: Report generation and viewing
Files: craxdrt.dll, crviewer.dll
Integration Points:
- frmReports (report viewer)
- modReports.bas (report generation)
Usage: All reporting functionality
Migration Notes: Replace with .NET reporting tools (RDLC)
```

#### Office Automation (Office 2003/2007)
```
Purpose: Excel/Word document generation
Files: Excel.exe, Winword.exe
Integration Points:
- modExport.bas (document export)
- frmReports (report export)
Usage: Report export to Excel/Word formats
Migration Notes: Replace with Office Interop or OpenXML
```

### ActiveX Controls

#### Calendar Control (MSCAL.OCX)
```
Purpose: Date selection interface
Usage: Date input fields across forms
Migration Notes: Replace with .NET DateTimePicker or calendar control
```

#### DataGrid Control (DBGrid32.OCX)
```
Purpose: Data display and editing
Usage: Data grids in order forms and reports
Migration Notes: Replace with .NET DataGridView
```

#### Chart Control (MSChart20.OCX)
```
Purpose: Data visualization
Usage: Charts in reports and analytics
Migration Notes: Replace with .NET charting controls
```

#### Masked Edit Control (MSMask32.OCX)
```
Purpose: Formatted input fields
Usage: Phone numbers, postal codes, dates
Migration Notes: Replace with .NET MaskedTextBox
```

### Windows API Calls

#### User32.dll Functions
```
- FindWindow: Window manipulation
- SendMessage: Inter-process communication
- SetWindowPos: Window positioning
- GetWindowText: Window text retrieval
Usage: Custom UI behaviors and integrations
Migration Notes: Replace with .NET window management
```

#### Kernel32.dll Functions
```
- GetPrivateProfileString: INI file reading
- WritePrivateProfileString: INI file writing
- GetComputerName: System information
- GetUserName: User information
Usage: Configuration management and system info
Migration Notes: Replace with .NET configuration and environment APIs
```

#### COM Components

#### Custom Business DLLs
```
Files: BusinessRules.dll, ValidationEngine.dll
Purpose: Extended business logic and validation
Integration Points: modBusinessLogic.bas, modValidation.bas
Usage: Complex business rule processing
Migration Notes: Convert to .NET assemblies
```

## Database Objects

### Tables (45 total)

#### Core Business Tables
```
Customers: Customer information
Orders: Order headers
OrderDetails: Order line items
Products: Product catalog
Categories: Product categories
Suppliers: Supplier information
Employees: Employee information
Shippers: Shipping companies
```

#### Supporting Tables
```
Regions: Geographic regions
Territories: Sales territories
EmployeeTerritories: Employee-territory assignments
CustomerDemographics: Customer classification
```

### Stored Procedures (78 total)

#### Order Processing SPs
```
sp_ProcessOrder: Complete order processing
sp_AllocateInventory: Inventory allocation
sp_CalculateOrderTotal: Order total calculation
sp_ValidateCreditLimit: Credit checking
sp_UpdateOrderStatus: Status updates
```

#### Reporting SPs
```
sp_CustomerSalesReport: Customer sales analysis
sp_ProductSalesReport: Product performance
sp_InventoryStatusReport: Stock levels
sp_SalesByPeriodReport: Period sales analysis
```

#### Maintenance SPs
```
sp_CleanupOldData: Data archiving
sp_UpdateInventoryLevels: Stock updates
sp_ProcessReturns: Return processing
sp_GenerateInvoices: Invoice creation
```

### Views (15 total)

#### Business Views
```
vw_CustomerOrders: Customer order summary
vw_ProductInventory: Product stock status
vw_OrderDetails: Order item details
vw_SalesByCustomer: Customer sales analysis
vw_SalesByProduct: Product sales analysis
```

#### Reporting Views
```
vw_MonthlySales: Monthly sales summary
vw_CustomerSummary: Customer activity summary
vw_InventorySummary: Inventory status overview
vw_OrderStatusSummary: Order status breakdown
```

## Configuration Files

### Application Configuration
```
App.ini: Application settings
- Database connection strings
- Report file paths
- Email server settings
- Default preferences

User.ini: User preferences
- Window positions
- Default search criteria
- Display preferences
- Recent file lists
```

### Database Configuration
```
Database connection parameters
- Server name and instance
- Database name
- Authentication method
- Connection pooling settings
- Timeout values
```

## File System Dependencies

### Report Templates
```
Location: .\Reports\
Files: CustomerReport.rpt, OrderReport.rpt, InventoryReport.rpt
Purpose: Crystal Reports templates
Migration Notes: Convert to RDLC format
```

### Export Templates
```
Location: .\Templates\
Files: InvoiceTemplate.doc, QuoteTemplate.doc, ReportTemplate.xls
Purpose: Office document templates
Migration Notes: Convert to OOXML format
```

### Log Files
```
Location: .\Logs\
Files: Application.log, Error.log, Security.log
Purpose: Application logging
Migration Notes: Implement structured logging
```

## Migration Impact Assessment

### Component Complexity Ranking

#### High Complexity Components
- modBusinessLogic.bas: Complex business rules
- frmOrderEntry: Complex UI workflow
- clsOrder: Complex business entity
- Order processing stored procedures

#### Medium Complexity Components
- modDatabase.bas: Data access patterns
- frmCustomerEntry: Form validation logic
- Crystal Reports integration
- Office automation

#### Low Complexity Components
- modCommon.bas: Utility functions
- Dialog forms: Simple UI components
- Basic validation functions
- Simple database queries

### Dependency Mapping

#### Critical Dependencies
```
frmMain → All child forms
modBusinessLogic → modDatabase, modValidation
frmOrderEntry → clsOrder, modBusinessLogic
Crystal Reports → All reporting functionality
```

#### Optional Dependencies
```
Office Automation → Export features
Custom ActiveX → Enhanced UI features
Windows API → System integration features
```

This comprehensive inventory provides the foundation for planning the migration effort and identifying components that require special attention during the conversion process.