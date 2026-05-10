# Component Mapping Guide

## VB6 to .NET Core Component Mapping

This document provides detailed mapping between VB6 application components and their .NET Core equivalents, including conversion patterns, complexity assessment, and implementation guidelines.

## Architecture Mapping

### High-Level Architecture Comparison

#### VB6 Architecture
```
VB6 Application
├── User Interface Layer
│   ├── Forms (MDI interface)
│   ├── Controls (VB6 native)
│   └── Event handlers
├── Business Logic Layer
│   ├── Modules (.bas files)
│   ├── Classes (.cls files)
│   └── Global functions
├── Data Access Layer
│   ├── ADO connections
│   ├── SQL queries
│   └── Stored procedures
└── External Interfaces
    ├── COM components
    ├── Windows API calls
    └── File system operations
```

#### .NET Core Architecture (Target)
```
.NET Core Application
├── Presentation Layer
│   ├── WPF Views (XAML)
│   ├── ViewModels (MVVM)
│   └── Controllers (Web API)
├── Application Layer
│   ├── Services (business logic)
│   ├── Commands/Queries (CQRS)
│   └── Event handlers
├── Domain Layer
│   ├── Entities (EF Core)
│   ├── Value Objects
│   └── Domain Services
└── Infrastructure Layer
    ├── EF Core Context
    ├── Repositories
    ├── External APIs
    └── File system services
```

## Module-to-Class Mapping

### Business Logic Modules

#### modBusinessLogic.bas → Business Services
```
VB6 Module Structure:
Public Function CalculateOrderTotal(orderId As Long) As Currency
Public Function ValidateCustomer(customer As clsCustomer) As Boolean
Public Sub ProcessOrder(orderId As Long)
Public Function ApplyDiscount(order As clsOrder) As Double

↓

C# Service Classes:
public class OrderService : IOrderService
{
    public decimal CalculateOrderTotal(int orderId)
    public ValidationResult ValidateCustomer(Customer customer)
    public async Task ProcessOrderAsync(int orderId)
    public decimal ApplyDiscount(Order order)
}

public class CustomerService : ICustomerService
{
    public ValidationResult ValidateCustomer(Customer customer)
    public async Task<Customer> CreateCustomerAsync(Customer customer)
}
```

**Mapping Rules**:
- Public functions → Public methods
- Parameters → Method parameters with type conversion
- Return values → Return types with type mapping
- Error handling → Exception throwing

#### modValidation.bas → Validation Classes
```
VB6 Validation Functions:
Public Function ValidateEmail(email As String) As Boolean
Public Function ValidatePhone(phone As String) As Boolean
Public Function ValidatePostalCode(code As String, country As String) As Boolean

↓

C# Validation Classes:
public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(c => c.Email)
            .NotEmpty()
            .EmailAddress();

        RuleFor(c => c.Phone)
            .Matches(@"^\+?[\d\s\-\(\)]+$")
            .When(c => !string.IsNullOrEmpty(c.Phone));
    }
}

public static class ValidationHelper
{
    public static bool IsValidEmail(string email)
    public static bool IsValidPhone(string phone)
    public static bool IsValidPostalCode(string code, string country)
}
```

**Mapping Rules**:
- Validation functions → FluentValidation rules
- Helper functions → Static utility methods
- Business rules → Validation rule classes

### Data Access Modules

#### modDatabase.bas → Infrastructure Layer
```
VB6 Database Module:
Public Function OpenConnection() As ADODB.Connection
Public Function ExecuteQuery(sql As String) As ADODB.Recordset
Public Sub ExecuteNonQuery(sql As String)
Public Function GetScalar(sql As String) As Variant

↓

C# Infrastructure Layer:
public class ApplicationDbContext : DbContext
{
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
}

public interface IDatabaseService
{
    Task<int> ExecuteNonQueryAsync(string sql, object parameters = null);
    Task<T> GetScalarAsync<T>(string sql, object parameters = null);
}

public class DatabaseService : IDatabaseService
{
    private readonly ApplicationDbContext _context;

    public async Task<int> ExecuteNonQueryAsync(string sql, object parameters = null)
    {
        return await _context.Database.ExecuteSqlRawAsync(sql, parameters);
    }
}
```

**Mapping Rules**:
- Connection management → DbContext
- Query execution → EF Core methods
- Transaction management → EF Core transactions
- Error handling → Exception handling

#### Repository Pattern Implementation
```
VB6 Data Access → Repository Pattern:
' Direct SQL in forms/modules
Dim rs As ADODB.Recordset
rs.Open "SELECT * FROM Customers", conn

↓

C# Repository Pattern:
public interface ICustomerRepository
{
    Task<Customer> GetByIdAsync(int id);
    Task<IEnumerable<Customer>> GetAllAsync();
    Task<Customer> AddAsync(Customer customer);
    Task UpdateAsync(Customer customer);
    Task DeleteAsync(int id);
}

public class CustomerRepository : ICustomerRepository
{
    private readonly ApplicationDbContext _context;

    public async Task<Customer> GetByIdAsync(int id)
    {
        return await _context.Customers.FindAsync(id);
    }
}
```

## Form-to-View Mapping

### Main Application Window

#### frmMain.frm → MainWindow.xaml
```
VB6 Main Form:
- MDI parent container
- Menu bar (File, Edit, View, Tools, Help)
- Toolbar with common actions
- Status bar with user info
- Child form management

↓

WPF Main Window:
<Window x:Class="VB6ToDotNetCore.UI.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        Title="VB6 to .NET Core Application" Height="600" Width="800">

    <Window.DataContext>
        <local:MainViewModel/>
    </Window.DataContext>

    <DockPanel>
        <Menu DockPanel.Dock="Top">
            <MenuItem Header="File">
                <MenuItem Header="New Customer" Command="{Binding NewCustomerCommand}"/>
                <MenuItem Header="New Order" Command="{Binding NewOrderCommand}"/>
                <Separator/>
                <MenuItem Header="Exit" Command="{Binding ExitCommand}"/>
            </MenuItem>
            <!-- Other menu items -->
        </Menu>

        <ToolBar DockPanel.Dock="Top">
            <Button Command="{Binding NewCustomerCommand}" Content="New Customer"/>
            <Button Command="{Binding NewOrderCommand}" Content="New Order"/>
            <Separator/>
            <Button Command="{Binding SaveCommand}" Content="Save"/>
        </ToolBar>

        <StatusBar DockPanel.Dock="Bottom">
            <TextBlock Text="{Binding StatusMessage}"/>
            <Separator/>
            <TextBlock Text="{Binding CurrentUser}"/>
        </StatusBar>

        <ContentControl Content="{Binding CurrentView}"/>
    </DockPanel>
</Window>
```

**Mapping Rules**:
- MDI container → ContentControl with data binding
- Menu items → Menu with commands
- Toolbar → ToolBar with buttons
- Status bar → StatusBar with bindings
- Child forms → UserControls with ViewModels

### Data Entry Forms

#### frmCustomerEntry.frm → CustomerView.xaml
```
VB6 Customer Form:
- TextBox: txtCustomerName, txtAddress, txtCity, etc.
- ComboBox: cboCountry, cboCustomerType
- CheckBox: chkIsActive
- CommandButton: btnSave, btnCancel, btnSearch
- DataGrid: grdSearchResults

↓

WPF Customer View:
<UserControl x:Class="VB6ToDotNetCore.UI.CustomerView">
    <UserControl.DataContext>
        <local:CustomerViewModel/>
    </UserControl.DataContext>

    <Grid>
        <Grid.ColumnDefinitions>
            <ColumnDefinition Width="Auto"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>

        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- Customer Name -->
        <Label Grid.Row="0" Grid.Column="0" Content="Customer Name:"/>
        <TextBox Grid.Row="0" Grid.Column="1" Text="{Binding Customer.Name, UpdateSourceTrigger=PropertyChanged}"/>

        <!-- Address -->
        <Label Grid.Row="1" Grid.Column="0" Content="Address:"/>
        <TextBox Grid.Row="1" Grid.Column="1" Text="{Binding Customer.Address, UpdateSourceTrigger=PropertyChanged}"/>

        <!-- City -->
        <Label Grid.Row="2" Grid.Column="0" Content="City:"/>
        <TextBox Grid.Row="2" Grid.Column="1" Text="{Binding Customer.City, UpdateSourceTrigger=PropertyChanged}"/>

        <!-- Search Results -->
        <DataGrid Grid.Row="3" Grid.Column="0" Grid.ColumnSpan="2"
                  ItemsSource="{Binding SearchResults}"
                  SelectedItem="{Binding SelectedCustomer}"
                  AutoGenerateColumns="False">
            <DataGrid.Columns>
                <DataGridTextColumn Header="Name" Binding="{Binding Name}"/>
                <DataGridTextColumn Header="City" Binding="{Binding City}"/>
                <DataGridTextColumn Header="Phone" Binding="{Binding Phone}"/>
            </DataGrid.Columns>
        </DataGrid>

        <!-- Buttons -->
        <StackPanel Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Orientation="Horizontal" HorizontalAlignment="Right">
            <Button Content="Save" Command="{Binding SaveCommand}" Margin="5"/>
            <Button Content="Cancel" Command="{Binding CancelCommand}" Margin="5"/>
            <Button Content="Search" Command="{Binding SearchCommand}" Margin="5"/>
        </StackPanel>
    </Grid>
</UserControl>
```

**Mapping Rules**:
- TextBox → TextBox with binding
- ComboBox → ComboBox with ItemsSource
- CheckBox → CheckBox with binding
- CommandButton → Button with Command
- DataGrid → DataGrid with ItemsSource
- Form layout → Grid layout

## Class Mapping

### Business Entity Classes

#### clsCustomer.cls → Customer Entity
```
VB6 Class:
Public CustomerID As Long
Public CompanyName As String
Public ContactName As String
Public Address As String
Public City As String
Public Phone As String
Public Email As String

Public Sub Save()
    ' Save logic
End Sub

↓

C# Entity:
public class Customer
{
    public int CustomerID { get; set; }
    public string CompanyName { get; set; }
    public string ContactName { get; set; }
    public string Address { get; set; }
    public string City { get; set; }
    public string Phone { get; set; }
    public string Email { get; set; }
    public bool IsActive { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime ModifiedDate { get; set; }

    // Navigation properties
    public ICollection<Order> Orders { get; set; }
}

EF Core Configuration:
public class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> builder)
    {
        builder.HasKey(c => c.CustomerID);
        builder.Property(c => c.CompanyName).HasMaxLength(100).IsRequired();
        builder.Property(c => c.Email).HasMaxLength(50);
        builder.HasMany(c => c.Orders).WithOne(o => o.Customer);
    }
}
```

**Mapping Rules**:
- Public fields → Properties with getters/setters
- Data types → .NET types
- Methods → Instance methods
- Relationships → Navigation properties

## Control Mapping

### VB6 Controls to WPF Controls

| VB6 Control | WPF Equivalent | Mapping Notes |
|-------------|----------------|----------------|
| TextBox | TextBox | Direct mapping, text binding |
| Label | Label | Direct mapping, content binding |
| CommandButton | Button | Command binding instead of Click event |
| ComboBox | ComboBox | ItemsSource and SelectedItem binding |
| ListBox | ListBox | ItemsSource and SelectedItem binding |
| CheckBox | CheckBox | IsChecked binding |
| OptionButton | RadioButton | GroupName and IsChecked binding |
| DataGrid | DataGrid | ItemsSource and column definitions |
| PictureBox | Image | Source binding |
| Frame | GroupBox | Direct mapping |
| TabStrip | TabControl | ItemsSource with TabItems |
| ProgressBar | ProgressBar | Value binding |
| StatusBar | StatusBar | Items with bindings |

### Event Handler Mapping

#### VB6 Event Handlers → WPF Commands
```
VB6 Event Handler:
Private Sub btnSave_Click()
    If ValidateData() Then
        SaveCustomer
        MsgBox "Customer saved successfully"
    End If
End Sub

↓

WPF Command:
public class CustomerViewModel : ViewModelBase
{
    public ICommand SaveCommand { get; }

    public CustomerViewModel()
    {
        SaveCommand = new RelayCommand(async () => await SaveAsync(), CanSave);
    }

    private async Task SaveAsync()
    {
        if (ValidateData())
        {
            await _customerService.SaveAsync(Customer);
            _messageService.ShowMessage("Customer saved successfully");
        }
    }

    private bool CanSave()
    {
        return Customer != null && !string.IsNullOrEmpty(Customer.Name);
    }
}
```

**Mapping Rules**:
- Event handlers → Command implementations
- UI logic → ViewModel methods
- Validation → CanExecute logic
- User feedback → Service calls

## Database Object Mapping

### Tables → Entities

#### Customers Table → Customer Entity
```
SQL Table:
CREATE TABLE Customers (
    CustomerID int IDENTITY PRIMARY KEY,
    CompanyName nvarchar(100) NOT NULL,
    ContactName nvarchar(50),
    Address nvarchar(200),
    City nvarchar(50),
    Phone nvarchar(24),
    Email nvarchar(50),
    IsActive bit DEFAULT 1,
    CreatedDate datetime DEFAULT GETDATE(),
    ModifiedDate datetime DEFAULT GETDATE()
)

↓

EF Core Entity:
[Table("Customers")]
public class Customer
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public int CustomerID { get; set; }

    [Required]
    [MaxLength(100)]
    public string CompanyName { get; set; }

    [MaxLength(50)]
    public string ContactName { get; set; }

    [MaxLength(200)]
    public string Address { get; set; }

    [MaxLength(50)]
    public string City { get; set; }

    [MaxLength(24)]
    public string Phone { get; set; }

    [MaxLength(50)]
    public string Email { get; set; }

    public bool IsActive { get; set; } = true;

    public DateTime CreatedDate { get; set; } = DateTime.Now;

    public DateTime ModifiedDate { get; set; } = DateTime.Now;
}
```

### Stored Procedures → EF Core Methods

#### sp_ProcessOrder → ProcessOrderAsync Method
```
SQL Stored Procedure:
CREATE PROCEDURE sp_ProcessOrder
    @OrderID int,
    @ProcessType nvarchar(20) = 'Complete'
AS
BEGIN
    -- Order processing logic
    UPDATE Orders SET Status = 'Processing' WHERE OrderID = @OrderID
    -- Additional processing
END

↓

EF Core Method:
public async Task ProcessOrderAsync(int orderId, string processType = "Complete")
{
    using var transaction = await _context.Database.BeginTransactionAsync();

    try
    {
        var order = await _context.Orders.FindAsync(orderId);
        order.Status = "Processing";

        // Additional processing logic
        await _orderProcessingService.ProcessAsync(order, processType);

        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

## API Integration Mapping

### External API Calls

#### Payment Processing API
```
VB6 API Call:
Public Function ProcessPayment(amount As Currency, cardInfo As clsCard) As Boolean
    Dim xml As String
    xml = CreatePaymentXML(amount, cardInfo)

    Dim http As New MSXML2.XMLHTTP
    http.open "POST", PAYMENT_API_URL
    http.setRequestHeader "Content-Type", "text/xml"
    http.send xml

    If http.status = 200 Then
        ProcessPayment = ParsePaymentResponse(http.responseText)
    Else
        ProcessPayment = False
    End If
End Function

↓

C# API Call:
public async Task<PaymentResult> ProcessPaymentAsync(decimal amount, CardInfo cardInfo)
{
    var paymentRequest = new PaymentRequest
    {
        Amount = amount,
        CardNumber = cardInfo.CardNumber,
        ExpiryMonth = cardInfo.ExpiryMonth,
        ExpiryYear = cardInfo.ExpiryYear,
        CVV = cardInfo.CVV
    };

    var response = await _httpClient.PostAsJsonAsync("api/payments", paymentRequest);

    if (response.IsSuccessStatusCode)
    {
        return await response.Content.ReadFromJsonAsync<PaymentResult>();
    }

    throw new PaymentException("Payment processing failed");
}
```

## Configuration Mapping

### INI Files → appsettings.json

#### VB6 Configuration
```
[Database]
Server=SQLSERVER01
Database=VB6App
UserID=appuser
Password=apppass

[API]
PaymentURL=https://api.paymentgateway.com
APIKey=your-api-key

↓

C# Configuration:
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=SQLSERVER01;Database=VB6App;User ID=appuser;Password=apppass;"
  },
  "ExternalAPIs": {
    "PaymentGateway": {
      "BaseUrl": "https://api.paymentgateway.com",
      "ApiKey": "your-api-key"
    }
  }
}
```

## Error Handling Mapping

### VB6 Error Handling → .NET Exception Handling

#### VB6 Error Patterns
```
On Error GoTo ErrorHandler
' Risky code
Exit Sub
ErrorHandler:
    Select Case Err.Number
        Case 5: MsgBox "Invalid data"
        Case 9: MsgBox "Database error"
        Case Else: MsgBox "Unknown error: " & Err.Description
    End Select
Resume Next

↓

C# Exception Handling:
public async Task SaveCustomerAsync(Customer customer)
{
    try
    {
        await _customerService.SaveAsync(customer);
        await _messageService.ShowSuccessAsync("Customer saved successfully");
    }
    catch (ValidationException ex)
    {
        await _messageService.ShowErrorAsync("Invalid data: " + ex.Message);
    }
    catch (DataException ex)
    {
        await _messageService.ShowErrorAsync("Database error: " + ex.Message);
        _logger.LogError(ex, "Database error saving customer {CustomerId}", customer.CustomerID);
    }
    catch (Exception ex)
    {
        await _messageService.ShowErrorAsync("An unexpected error occurred");
        _logger.LogCritical(ex, "Unexpected error saving customer {CustomerId}", customer.CustomerID);
        throw;
    }
}
```

## Testing Mapping

### VB6 Testing (Limited) → .NET Testing

#### Unit Test Mapping
```
VB6 Manual Testing → xUnit Tests:
' Manual testing, MsgBox results
↓

public class CustomerServiceTests
{
    [Fact]
    public async Task SaveAsync_ValidCustomer_SavesSuccessfully()
    {
        // Arrange
        var customer = new Customer { Name = "Test Customer" };
        var mockRepo = new Mock<ICustomerRepository>();
        var service = new CustomerService(mockRepo.Object);

        // Act
        await service.SaveAsync(customer);

        // Assert
        mockRepo.Verify(r => r.AddAsync(customer), Times.Once);
    }

    [Fact]
    public async Task SaveAsync_InvalidCustomer_ThrowsValidationException()
    {
        // Arrange
        var customer = new Customer { Name = "" }; // Invalid
        var service = new CustomerService(Mock.Of<ICustomerRepository>());

        // Act & Assert
        await Assert.ThrowsAsync<ValidationException>(() => service.SaveAsync(customer));
    }
}
```

This comprehensive component mapping guide provides the detailed transformation patterns needed to convert each VB6 component to its .NET Core equivalent while maintaining functionality and improving architecture.