# Components

This document identifies and catalogs all components found in the VB6 application source code. Components are organized by type for clarity in the migration process.

## Forms
### Main Forms
- **frmMain**: Main application window
- **frmLogin**: User authentication form
- **frmDashboard**: Application dashboard

### Data Entry Forms
- **frmCustomerEntry**: Customer information input
- **frmOrderEntry**: Order processing form
- **frmProductEntry**: Product management form
- **frmInventoryEntry**: Inventory tracking form

### Reporting Forms
- **frmReportViewer**: Generic report display
- **frmSalesReport**: Sales analytics
- **frmInventoryReport**: Stock reports

### Utility Forms
- **frmSettings**: Application configuration
- **frmAbout**: Application information
- **frmHelp**: User assistance

## Modules
### Business Logic Modules
- **modBusinessLogic**: Core business rules and calculations
- **modValidation**: Data validation functions
- **modSecurity**: Authentication and authorization

### Data Access Modules
- **modDatabase**: Database connection and queries
- **modADOHelper**: ADO utility functions
- **modDataConversion**: Data type conversions

### Utility Modules
- **modCommon**: Shared utility functions
- **modConstants**: Application constants
- **modErrorHandler**: Error handling routines

## Classes
### Data Classes
- **clsCustomer**: Customer entity
- **clsOrder**: Order entity
- **clsProduct**: Product entity
- **clsInventory**: Inventory entity

### Business Classes
- **clsOrderProcessor**: Order processing logic
- **clsReportGenerator**: Report generation
- **clsEmailSender**: Email functionality

### Utility Classes
- **clsLogger**: Application logging
- **clsConfigManager**: Configuration management

## ActiveX Controls
### UI Controls
- **MSFlexGrid**: Data grid display
- **Calendar**: Date picker
- **ProgressBar**: Progress indication
- **StatusBar**: Status display

### Third-party Controls
- **FarPoint Spread**: Advanced spreadsheet control
- **ComponentOne Controls**: Various UI components
- **Crystal Reports Viewer**: Report display

## COM Objects
### Microsoft COM Objects
- **ADODB.Connection**: Database connectivity
- **ADODB.Recordset**: Data manipulation
- **CDO.Message**: Email sending
- **Scripting.FileSystemObject**: File operations

### Custom COM Objects
- **CustomDataAccess**: Proprietary data access layer
- **ReportEngine**: Custom reporting component

## APIs
### Windows APIs
- **user32.dll**: User interface functions
- **kernel32.dll**: System functions
- **gdi32.dll**: Graphics functions

### Database APIs
- **ODBC API**: Database connectivity
- **OLE DB**: Data access abstraction

### External APIs
- **Office Automation**: Excel/Word integration
- **Web Services**: SOAP-based services (if any)

## DLLs and Libraries
- **msvbvm60.dll**: VB6 runtime
- **oleaut32.dll**: OLE automation
- **comdlg32.dll**: Common dialogs
- **Custom libraries**: Business-specific DLLs

## Database Objects
### Tables
- Customers, Orders, Products, Inventory, Users

### Views
- ActiveOrders, ProductSummary, SalesByMonth

### Stored Procedures
- sp_GetCustomer, sp_ProcessOrder, sp_GenerateReport

## Configuration Files
- **app.config**: Application settings
- **connection.config**: Database connections
- **user.config**: User preferences