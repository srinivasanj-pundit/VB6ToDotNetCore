# Conversion Instructions

## VB6 to .NET Core 10 Migration Guidelines

This document provides detailed instructions and guidelines for converting VB6 application components to .NET Core 10.

## General Conversion Principles

### Language Syntax Conversion

#### Variable Declarations
```
VB6 → C#
Dim x As Integer → int x;
Dim name As String → string name;
Dim isValid As Boolean → bool isValid;
Dim amount As Currency → decimal amount;
Dim data As Variant → object data; (avoid if possible)
```

#### Control Structures
```
VB6 If-Then-Else → C# if-else:
If condition Then
    ' code
ElseIf otherCondition Then
    ' code
Else
    ' code
End If

↓

if (condition)
{
    // code
}
else if (otherCondition)
{
    // code
}
else
{
    // code
}
```

#### Loops
```
VB6 For-Next → C# for:
For i = 1 To 10
    ' code
Next i

↓

for (int i = 1; i <= 10; i++)
{
    // code
}
```

```
VB6 For-Each → C# foreach:
For Each item In collection
    ' code
Next item

↓

foreach (var item in collection)
{
    // code
}
```

```
VB6 Do-While → C# while:
Do While condition
    ' code
Loop

↓

while (condition)
{
    // code
}
```

#### Functions and Subroutines
```
VB6 Function → C# method:
Public Function CalculateTotal(amount As Double, taxRate As Double) As Double
    CalculateTotal = amount * (1 + taxRate)
End Function

↓

public double CalculateTotal(double amount, double taxRate)
{
    return amount * (1 + taxRate);
}
```

```
VB6 Sub → C# void method:
Public Sub ProcessOrder(orderId As Long)
    ' processing logic
End Sub

↓

public void ProcessOrder(long orderId)
{
    // processing logic
}
```

### Data Type Mapping

#### Primitive Types
| VB6 Type | .NET Type | Notes |
|----------|-----------|-------|
| Integer | int | 32-bit signed integer |
| Long | long | 64-bit signed integer |
| Single | float | 32-bit floating point |
| Double | double | 64-bit floating point |
| Currency | decimal | 128-bit decimal for financial calculations |
| String | string | Unicode string |
| Boolean | bool | true/false |
| Byte | byte | 8-bit unsigned integer |
| Date | DateTime | Date and time structure |
| Variant | object | Avoid using, use specific types |

#### Arrays
```
VB6 → C#:
Dim numbers(1 To 10) As Integer → int[] numbers = new int[10];
Dim names() As String → string[] names;
ReDim Preserve names(0 To 5) → Array.Resize(ref names, 6);
```

#### Collections
```
VB6 Collection → C# generic collections:
Dim customers As New Collection → List<Customer> customers = new List<Customer>();
Dim lookup As Dictionary → Dictionary<string, Customer> lookup = new Dictionary<string, Customer>();
```

### Error Handling

#### VB6 Error Handling
```
VB6 On Error:
On Error GoTo ErrorHandler
' code that might fail
Exit Sub
ErrorHandler:
    MsgBox "Error: " & Err.Description
    Resume Next
```

#### .NET Exception Handling
```
C# try-catch-finally:
try
{
    // code that might fail
}
catch (Exception ex)
{
    MessageBox.Show($"Error: {ex.Message}");
}
finally
{
    // cleanup code
}
```

#### Custom Exceptions
```
C# custom exception:
public class BusinessRuleException : Exception
{
    public BusinessRuleException(string message) : base(message) { }
}

// Usage:
throw new BusinessRuleException("Invalid customer data");
```

### Object-Oriented Programming

#### Classes and Objects
```
VB6 Class → C# class:
' Customer.cls
Public Name As String
Public Address As String

Public Sub Save()
    ' save logic
End Sub

↓

// Customer.cs
public class Customer
{
    public string Name { get; set; }
    public string Address { get; set; }

    public void Save()
    {
        // save logic
    }
}
```

#### Properties
```
VB6 Property → C# property:
Public Property Get FullName() As String
    FullName = firstName & " " & lastName
End Property

Public Property Let FullName(value As String)
    ' parsing logic
End Property

↓

public string FullName
{
    get { return $"{firstName} {lastName}"; }
    set
    {
        // parsing logic
    }
}
```

#### Inheritance
```
VB6 inheritance (limited) → C# inheritance:
' Not directly supported in VB6
↓

public class PremiumCustomer : Customer
{
    public decimal CreditLimit { get; set; }

    public override void Save()
    {
        // premium save logic
        base.Save();
    }
}
```

#### Interfaces
```
VB6 interfaces (limited) → C# interfaces:
' Not directly supported in VB6
↓

public interface ICustomerRepository
{
    Customer GetById(int id);
    void Save(Customer customer);
}

public class CustomerRepository : ICustomerRepository
{
    // implementation
}
```

### Database Access Conversion

#### ADO to Entity Framework Core

##### Connection Management
```
VB6 ADO:
Dim conn As New ADODB.Connection
conn.ConnectionString = "connection string"
conn.Open

↓

C# EF Core:
using var context = new ApplicationDbContext();
```

##### Query Execution
```
VB6 ADO Query:
Dim rs As New ADODB.Recordset
rs.Open "SELECT * FROM Customers WHERE City = 'London'", conn

↓

C# EF Core Query:
var customers = context.Customers
    .Where(c => c.City == "London")
    .ToList();
```

##### Parameterized Queries
```
VB6 ADO Parameters:
Dim cmd As New ADODB.Command
cmd.CommandText = "SELECT * FROM Products WHERE Price > ?"
cmd.Parameters.Append cmd.CreateParameter("Price", adCurrency, adParamInput, , minPrice)

↓

C# EF Core Parameters:
var products = context.Products
    .Where(p => p.UnitPrice > minPrice)
    .ToList();
```

##### CRUD Operations
```
VB6 ADO Insert:
Dim sql As String
sql = "INSERT INTO Customers (Name, City) VALUES ('" & name & "', '" & city & "')"
conn.Execute sql

↓

C# EF Core Insert:
var customer = new Customer { Name = name, City = city };
context.Customers.Add(customer);
context.SaveChanges();
```

### UI Conversion Guidelines

#### Windows Forms Migration

##### Form Structure
```
VB6 Form → Windows Forms:
' Form1.frm with controls
↓

public partial class CustomerForm : Form
{
    private TextBox txtName;
    private Button btnSave;

    public CustomerForm()
    {
        InitializeComponent();
    }
}
```

##### Event Handlers
```
VB6 Event → C# Event:
Private Sub btnSave_Click()
    ' save logic
End Sub

↓

private void btnSave_Click(object sender, EventArgs e)
{
    // save logic
}
```

##### Control Properties
```
VB6 Control Properties → C# Properties:
txtName.Text = "John Doe"
txtName.Enabled = True
txtName.MaxLength = 50

↓

txtName.Text = "John Doe";
txtName.Enabled = true;
txtName.MaxLength = 50;
```

#### WPF Migration (Recommended)

##### XAML Structure
```
VB6 Form → WPF Window:
<!-- MainWindow.xaml -->
<Window x:Class="WpfApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation">
    <Grid>
        <TextBox x:Name="txtName" Text="{Binding Customer.Name}" />
        <Button x:Name="btnSave" Content="Save" Click="btnSave_Click" />
    </Grid>
</Window>
```

##### MVVM Pattern Implementation
```
C# ViewModel:
public class CustomerViewModel : INotifyPropertyChanged
{
    private string name;
    public string Name
    {
        get => name;
        set
        {
            name = value;
            OnPropertyChanged();
        }
    }

    public ICommand SaveCommand { get; }

    public CustomerViewModel()
    {
        SaveCommand = new RelayCommand(Save);
    }

    private void Save()
    {
        // save logic
    }
}
```

### Business Logic Conversion

#### Validation Logic
```
VB6 Validation → C# Validation:
Public Function ValidateCustomer(cust As Customer) As Boolean
    If Len(cust.Name) = 0 Then
        ValidateCustomer = False
        Exit Function
    End If
    ' more validation
    ValidateCustomer = True
End Function

↓

public ValidationResult ValidateCustomer(Customer customer)
{
    var result = new ValidationResult();

    if (string.IsNullOrEmpty(customer.Name))
    {
        result.Errors.Add("Name is required");
    }

    // more validation

    return result;
}
```

#### Business Rules Engine
```
VB6 Business Rules → C# Business Rules:
Public Function CalculateDiscount(order As Order) As Double
    If order.Total > 1000 Then
        CalculateDiscount = 0.1
    ElseIf order.Customer.Type = "Premium" Then
        CalculateDiscount = 0.05
    Else
        CalculateDiscount = 0
    End If
End Function

↓

public decimal CalculateDiscount(Order order)
{
    if (order.Total > 1000)
        return 0.1m;
    else if (order.Customer.Type == CustomerType.Premium)
        return 0.05m;
    else
        return 0m;
}
```

### File System Operations

#### File I/O
```
VB6 File Operations → C# File Operations:
Open "data.txt" For Input As #1
Line Input #1, textLine
Close #1

↓

string text = File.ReadAllText("data.txt");
```

#### Directory Operations
```
VB6 Directory → C# Directory:
Dim files() As String
files = Dir("C:\Data\*.txt")

↓

string[] files = Directory.GetFiles(@"C:\Data", "*.txt");
```

### Configuration Management

#### INI Files to Configuration
```
VB6 INI Reading → C# Configuration:
Dim value As String
value = GetPrivateProfileString("Settings", "Server", "", "app.ini")

↓

// Using Microsoft.Extensions.Configuration
var config = new ConfigurationBuilder()
    .AddIniFile("app.ini")
    .Build();
string value = config["Settings:Server"];
```

### Testing Conversion

#### Unit Testing Framework
```
VB6 Testing (limited) → C# xUnit:
' No direct equivalent
↓

public class CustomerServiceTests
{
    [Fact]
    public void CalculateDiscount_Over1000_Returns10Percent()
    {
        // Arrange
        var order = new Order { Total = 1500 };

        // Act
        var discount = _service.CalculateDiscount(order);

        // Assert
        Assert.Equal(0.1m, discount);
    }
}
```

### Security Considerations

#### Input Validation
```
VB6 Basic Validation → C# Comprehensive Validation:
' Basic length checks
↓

public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(c => c.Name)
            .NotEmpty()
            .MaximumLength(100)
            .Matches(@"^[a-zA-Z\s]+$");

        RuleFor(c => c.Email)
            .NotEmpty()
            .EmailAddress();
    }
}
```

#### SQL Injection Prevention
```
VB6 Vulnerable → C# Parameterized:
sql = "SELECT * FROM Users WHERE Name = '" & userName & "'"

↓

var user = context.Users
    .FirstOrDefault(u => u.Name == userName);
```

### Performance Optimization

#### String Concatenation
```
VB6 Inefficient → C# Efficient:
Dim result As String
For i = 1 To 1000
    result = result & "item" & i
Next i

↓

var result = new StringBuilder();
for (int i = 1; i <= 1000; i++)
{
    result.Append($"item{i}");
}
string finalResult = result.ToString();
```

#### Memory Management
```
VB6 Manual → C# Automatic:
' Set object = Nothing (not required in .NET)
↓

// Automatic garbage collection
// Use using statements for disposables
using (var stream = new FileStream(path, FileMode.Open))
{
    // use stream
} // automatically disposed
```

### Code Organization

#### Namespace Structure
```
VB6 Flat Structure → C# Namespaced:
' All modules in root
↓

namespace VB6ToDotNetCore.Core.Entities
{
    public class Customer { }
}

namespace VB6ToDotNetCore.Core.Services
{
    public class CustomerService { }
}
```

#### Dependency Injection
```
VB6 Tight Coupling → C# DI:
' Direct instantiation
Dim service As New CustomerService

↓

public class OrderController
{
    private readonly ICustomerService _customerService;

    public OrderController(ICustomerService customerService)
    {
        _customerService = customerService;
    }
}
```

### Error Logging

#### Logging Framework
```
VB6 Basic Logging → C# Structured Logging:
Open "error.log" For Append As #1
Print #1, "Error: " & Err.Description
Close #1

↓

_logger.LogError(ex, "Error processing order {OrderId}", orderId);
```

### API Integration

#### Web Service Calls
```
VB6 SOAP → C# HttpClient:
Dim soap As New MSXML2.XMLHTTP
soap.open "POST", "http://api.example.com/service"
soap.send xmlData

↓

using var client = new HttpClient();
var response = await client.PostAsync("http://api.example.com/service",
    new StringContent(jsonData, Encoding.UTF8, "application/json"));
```

### Deployment Considerations

#### Application Configuration
```
VB6 app.ini → C# appsettings.json:
[Database]
Server=localhost
Database=MyDB

↓

{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDB;..."
  }
}
```

#### Build Process
```
VB6 IDE Build → C# CLI/Docker:
' Manual compilation
↓

dotnet build
dotnet publish -c Release
docker build -t myapp .
```

These conversion guidelines provide a comprehensive reference for migrating VB6 code to modern .NET Core 10. Each section includes specific examples and best practices to ensure accurate and efficient conversion.