# VB6 to .NET Core 10 Full Conversion Guide

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [Prerequisites and Setup](#prerequisites-and-setup)
3. [Application Analysis](#application-analysis)
4. [Component Inventory](#component-inventory)
5. [Migration Planning](#migration-planning)
6. [Project Setup](#project-setup)
7. [Data Access Migration](#data-access-migration)
8. [Business Logic Conversion](#business-logic-conversion)
9. [User Interface Migration](#user-interface-migration)
10. [API and Integration Migration](#api-and-integration-migration)
11. [Testing Strategy](#testing-strategy)
12. [Security Implementation](#security-implementation)
13. [Build and Deployment](#build-and-deployment)
14. [Containerization](#containerization)
15. [Production Deployment](#production-deployment)
16. [Post-Migration Activities](#post-migration-activities)

## Executive Summary

This comprehensive guide provides a complete roadmap for migrating legacy VB6 applications to .NET Core 10. The migration process involves systematic conversion of all components while maintaining business logic integrity and improving application architecture.

### Key Benefits
- Modern .NET Core 10 framework with long-term support
- Cross-platform compatibility (Windows, Linux, macOS)
- Improved performance and scalability
- Enhanced security features
- Better maintainability and developer productivity

### Migration Scope
- Complete codebase conversion
- Database schema and data migration
- UI modernization
- API modernization
- Security enhancements
- Testing and quality assurance
- Deployment automation

## Prerequisites and Setup

### Development Environment Requirements

#### Hardware Requirements
- **Processor**: Quad-core CPU (2.5 GHz or higher)
- **Memory**: 16 GB RAM minimum, 32 GB recommended
- **Storage**: 500 GB SSD for development environment
- **Display**: 1920x1080 resolution minimum

#### Software Requirements
```bash
# .NET Core 10 SDK
wget https://dotnet.microsoft.com/download/dotnet/10.0/dotnet-sdk-10.0.100-linux-x64.tar.gz
mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-10.0.100-linux-x64.tar.gz -C $HOME/dotnet
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet

# Visual Studio Code with extensions
code --install-extension ms-dotnettools.csharp
code --install-extension ms-vscode.vscode-dotnet-runtime
code --install-extension ms-vscode.vscode-json
code --install-extension ms-vscode.powershell

# Docker and Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# SQL Server tools (for database migration)
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/20.04/prod.list | sudo tee /etc/apt/sources.list.d/msprod.list
sudo apt-get update
sudo apt-get install -y mssql-tools unixodbc-dev
```

#### Required NuGet Packages
```xml
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="10.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="10.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Mvc" Version="10.0.0" />
<PackageReference Include="Microsoft.Extensions.Logging" Version="10.0.0" />
<PackageReference Include="xunit" Version="2.9.0" />
<PackageReference Include="Moq" Version="4.20.70" />
<PackageReference Include="AutoMapper" Version="13.0.1" />
<PackageReference Include="FluentValidation" Version="11.9.2" />
<PackageReference Include="Serilog" Version="4.0.0" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="7.0.0" />
```

### Team Skills and Training
- .NET Core development experience
- C# programming proficiency
- Entity Framework Core knowledge
- Unit testing with xUnit
- Docker containerization
- CI/CD pipeline experience

## Application Analysis

### Codebase Assessment

#### Source Code Analysis
```bash
# Count lines of code
find . -name "*.frm" -o -name "*.bas" -o -name "*.cls" | xargs wc -l

# Analyze dependencies
# Extract all references and imports
grep -r "References" *.vbp
grep -r "Object" *.frm *.bas *.cls | grep "=" | sort | uniq

# Identify database connections
grep -r "ConnectionString\|Data Source\|Initial Catalog" * | sort | uniq
```

#### Architecture Documentation
```markdown
## Current Architecture
- **Application Type**: Desktop application
- **UI Framework**: VB6 Forms
- **Data Access**: ADO 2.8
- **Business Logic**: Procedural code in modules
- **External Dependencies**: Crystal Reports, Office Automation
- **Database**: SQL Server 2000/2005
- **Deployment**: Standalone executable
```

#### Complexity Assessment
```csharp
// Code complexity metrics
public class CodeMetrics
{
    public int TotalLinesOfCode { get; set; }
    public int NumberOfForms { get; set; }
    public int NumberOfModules { get; set; }
    public int NumberOfClasses { get; set; }
    public int DatabaseTables { get; set; }
    public int StoredProcedures { get; set; }
    public int ExternalAPICalls { get; set; }
    public int COMReferences { get; set; }
}
```

### Business Logic Analysis

#### Core Business Processes
1. **Authentication and Authorization**
   - User login validation
   - Role-based access control
   - Session management

2. **Data Management**
   - CRUD operations for business entities
   - Data validation and business rules
   - Report generation

3. **Workflow Processing**
   - Business process automation
   - State management
   - Audit trail maintenance

#### Business Rules Extraction
```csharp
// Example business rule extraction
public class BusinessRulesEngine
{
    public bool ValidateOrder(Order order)
    {
        // Extracted from VB6 validation logic
        if (order.TotalAmount <= 0) return false;
        if (order.CustomerId == null) return false;
        if (order.Items.Count == 0) return false;

        // Business-specific rules
        if (order.OrderType == OrderType.Rush && order.TotalAmount < 1000)
            return false;

        return true;
    }

    public decimal CalculateDiscount(Order order)
    {
        // Extracted discount calculation logic
        if (order.TotalAmount >= 5000) return 0.10m;
        if (order.TotalAmount >= 1000) return 0.05m;
        return 0.00m;
    }
}
```

## Component Inventory

### Forms Inventory
```csharp
public class FormInventory
{
    public List<FormComponent> Forms { get; set; } = new();

    public class FormComponent
    {
        public string Name { get; set; }
        public string Purpose { get; set; }
        public List<Control> Controls { get; set; } = new();
        public List<string> EventHandlers { get; set; } = new();
        public bool HasValidation { get; set; }
        public bool HasDatabaseInteraction { get; set; }
    }
}

// Example form analysis
var loginForm = new FormComponent
{
    Name = "frmLogin",
    Purpose = "User authentication",
    Controls = new List<Control>
    {
        new Control { Name = "txtUsername", Type = "TextBox" },
        new Control { Name = "txtPassword", Type = "TextBox", IsPassword = true },
        new Control { Name = "btnLogin", Type = "Button" },
        new Control { Name = "btnCancel", Type = "Button" }
    },
    EventHandlers = new List<string> { "btnLogin_Click", "btnCancel_Click" },
    HasValidation = true,
    HasDatabaseInteraction = true
};
```

### Modules and Classes Inventory
```csharp
public class ModuleInventory
{
    public List<ModuleComponent> Modules { get; set; } = new();

    public class ModuleComponent
    {
        public string Name { get; set; }
        public ModuleType Type { get; set; }
        public List<Function> Functions { get; set; } = new();
        public List<string> Dependencies { get; set; } = new();
    }

    public enum ModuleType
    {
        BusinessLogic,
        DataAccess,
        Utility,
        Constants
    }
}
```

### Database Objects Inventory
```sql
-- Database schema analysis
SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES
ORDER BY TABLE_SCHEMA, TABLE_NAME;

-- Stored procedures inventory
SELECT
    ROUTINE_SCHEMA,
    ROUTINE_NAME,
    ROUTINE_TYPE
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_TYPE = 'PROCEDURE'
ORDER BY ROUTINE_SCHEMA, ROUTINE_NAME;

-- Foreign key relationships
SELECT
    fk.name AS FK_Name,
    tp.name AS Parent_Table,
    tr.name AS Referenced_Table
FROM sys.foreign_keys fk
INNER JOIN sys.tables tp ON fk.parent_object_id = tp.object_id
INNER JOIN sys.tables tr ON fk.referenced_object_id = tr.object_id;
```

## Migration Planning

### Migration Strategy Selection

#### Big Bang Migration
- **Pros**: Complete migration in single effort
- **Cons**: High risk, long downtime
- **When to Use**: Small applications, tight deadlines

#### Incremental Migration
- **Pros**: Reduced risk, phased approach
- **Cons**: Complex management, temporary dual maintenance
- **When to Use**: Large applications, business-critical systems

#### Strangler Fig Pattern
- **Pros**: Zero downtime, gradual migration
- **Cons**: Complex architecture during transition
- **When to Use**: Large legacy systems, 24/7 availability requirements

### Detailed Mapping Plan

#### UI Component Mapping
```csharp
public static class UIComponentMapper
{
    public static Dictionary<string, string> Vb6ToDotNetControls = new()
    {
        ["VB.TextBox"] = "System.Windows.Forms.TextBox",
        ["VB.CommandButton"] = "System.Windows.Forms.Button",
        ["VB.Label"] = "System.Windows.Forms.Label",
        ["VB.ComboBox"] = "System.Windows.Forms.ComboBox",
        ["VB.ListBox"] = "System.Windows.Forms.ListBox",
        ["VB.CheckBox"] = "System.Windows.Forms.CheckBox",
        ["VB.OptionButton"] = "System.Windows.Forms.RadioButton",
        ["VB.Frame"] = "System.Windows.Forms.GroupBox",
        ["MSFlexGridLib.MSFlexGrid"] = "System.Windows.Forms.DataGridView",
        ["MSComCtlLib.StatusBar"] = "System.Windows.Forms.StatusStrip"
    };

    public static string MapControl(string vb6Control)
    {
        return Vb6ToDotNetControls.TryGetValue(vb6Control, out var dotnetControl)
            ? dotnetControl
            : "System.Windows.Forms.Control"; // Fallback
    }
}
```

#### Data Access Mapping
```csharp
public static class DataAccessMapper
{
    public static string ConvertAdoToEf(string adoCode)
    {
        // ADO Recordset to EF pattern
        return adoCode
            .Replace("ADODB.Recordset", "IQueryable<TEntity>")
            .Replace("rs.Open", "context.Entities.Where")
            .Replace("rs.MoveNext", "foreach(var item in query)")
            .Replace("rs.Fields[\"", "item.")
            .Replace("\"]", "");
    }
}
```

#### API Mapping
```csharp
public static class ApiMapper
{
    public static Dictionary<string, string> WindowsApiMappings = new()
    {
        ["user32.dll"] = "System.Windows.Forms",
        ["kernel32.dll"] = "System.Runtime.InteropServices",
        ["gdi32.dll"] = "System.Drawing"
    };

    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);

    // .NET Core equivalent
    public static void ShowMessage(string text, string caption)
    {
        MessageBox(IntPtr.Zero, text, caption, 0);
    }
}
```

## Project Setup

### Solution Structure
```
VB6ToDotNetCore/
├── src/
│   ├── VB6ToDotNetCore.Core/           # Business logic
│   │   ├── Entities/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   └── Validators/
│   ├── VB6ToDotNetCore.Data/           # Data access
│   │   ├── Context/
│   │   ├── Repositories/
│   │   └── Migrations/
│   ├── VB6ToDotNetCore.Web/            # Web API
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Filters/
│   │   └── Middleware/
│   ├── VB6ToDotNetCore.UI/             # Desktop UI
│   │   ├── Forms/
│   │   ├── Controls/
│   │   └── ViewModels/
│   └── VB6ToDotNetCore.Worker/         # Background services
├── tests/
│   ├── VB6ToDotNetCore.Core.Tests/
│   ├── VB6ToDotNetCore.Data.Tests/
│   ├── VB6ToDotNetCore.Web.Tests/
│   └── VB6ToDotNetCore.IntegrationTests/
├── tools/
│   └── VB6ToDotNetCore.MigrationTools/
├── build/
│   ├── Directory.Build.props
│   ├── Directory.Build.targets
│   └── VB6ToDotNetCore.Build.props
├── docker/
│   ├── Dockerfile.api
│   ├── Dockerfile.ui
│   ├── docker-compose.yml
│   └── nginx.conf
└── docs/
    ├── api-documentation.md
    └── deployment-guide.md
```

### Project Files Creation

#### Core Project
```xml
<!-- VB6ToDotNetCore.Core.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="FluentValidation" Version="11.9.2" />
    <PackageReference Include="AutoMapper" Version="13.0.1" />
    <PackageReference Include="Microsoft.Extensions.Logging.Abstractions" Version="10.0.0" />
  </ItemGroup>
</Project>
```

#### Data Project
```xml
<!-- VB6ToDotNetCore.Data.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="10.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="10.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\VB6ToDotNetCore.Core\VB6ToDotNetCore.Core.csproj" />
  </ItemGroup>
</Project>
```

#### Web API Project
```xml
<!-- VB6ToDotNetCore.Web.csproj -->
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="10.0.0" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="7.0.0" />
    <PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\VB6ToDotNetCore.Core\VB6ToDotNetCore.Core.csproj" />
    <ProjectReference Include="..\VB6ToDotNetCore.Data\VB6ToDotNetCore.Data.csproj" />
  </ItemGroup>
</Project>
```

### Configuration Files

#### appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=VB6ToDotNetCore;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "logs/log-.txt",
          "rollingInterval": "Day"
        }
      }
    ]
  },
  "Jwt": {
    "Key": "your-secret-key-here",
    "Issuer": "VB6ToDotNetCore",
    "Audience": "VB6ToDotNetCoreUsers"
  }
}
```

#### Program.cs
```csharp
using Serilog;
using VB6ToDotNetCore.Web.Extensions;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog
builder.Host.UseSerilog((context, config) =>
{
    config.ReadFrom.Configuration(context.Configuration);
});

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add application services
builder.Services.AddApplicationServices(builder.Configuration);
builder.Services.AddDataServices(builder.Configuration);

var app = builder.Build();

// Configure middleware
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Data Access Migration

### Entity Framework Core Setup

#### DbContext Configuration
```csharp
using Microsoft.EntityFrameworkCore;
using VB6ToDotNetCore.Core.Entities;

namespace VB6ToDotNetCore.Data.Context;

public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // DbSets for entities
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Product> Products { get; set; }
    public DbSet<OrderItem> OrderItems { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure entities
        modelBuilder.ApplyConfiguration(new CustomerConfiguration());
        modelBuilder.ApplyConfiguration(new OrderConfiguration());
        modelBuilder.ApplyConfiguration(new ProductConfiguration());
        modelBuilder.ApplyConfiguration(new OrderItemConfiguration());

        // Seed initial data if needed
        modelBuilder.SeedData();
    }
}
```

#### Entity Configurations
```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;
using VB6ToDotNetCore.Core.Entities;

namespace VB6ToDotNetCore.Data.Configurations;

public class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> builder)
    {
        builder.ToTable("Customers");

        builder.HasKey(c => c.Id);

        builder.Property(c => c.Name)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(c => c.Email)
            .IsRequired()
            .HasMaxLength(255);

        builder.Property(c => c.Phone)
            .HasMaxLength(20);

        builder.Property(c => c.CreatedDate)
            .HasDefaultValueSql("GETDATE()");

        // Indexes
        builder.HasIndex(c => c.Email)
            .IsUnique();

        builder.HasIndex(c => c.Name);
    }
}
```

#### Repository Pattern Implementation
```csharp
using System.Linq.Expressions;
using Microsoft.EntityFrameworkCore;
using VB6ToDotNetCore.Core.Entities;
using VB6ToDotNetCore.Core.Interfaces;

namespace VB6ToDotNetCore.Data.Repositories;

public class CustomerRepository : ICustomerRepository
{
    private readonly ApplicationDbContext _context;

    public CustomerRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<Customer> GetByIdAsync(int id)
    {
        return await _context.Customers.FindAsync(id);
    }

    public async Task<IEnumerable<Customer>> GetAllAsync()
    {
        return await _context.Customers.ToListAsync();
    }

    public async Task<IEnumerable<Customer>> FindAsync(Expression<Func<Customer, bool>> predicate)
    {
        return await _context.Customers.Where(predicate).ToListAsync();
    }

    public async Task<Customer> AddAsync(Customer customer)
    {
        _context.Customers.Add(customer);
        await _context.SaveChangesAsync();
        return customer;
    }

    public async Task UpdateAsync(Customer customer)
    {
        _context.Entry(customer).State = EntityState.Modified;
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var customer = await GetByIdAsync(id);
        if (customer != null)
        {
            _context.Customers.Remove(customer);
            await _context.SaveChangesAsync();
        }
    }
}
```

### ADO to EF Core Conversion

#### Connection String Migration
```csharp
// VB6 ADO Connection
Dim conn As ADODB.Connection
Set conn = New ADODB.Connection
conn.ConnectionString = "Provider=SQLOLEDB;Data Source=server;Initial Catalog=database;User ID=user;Password=password"
conn.Open

// .NET Core EF Core
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

#### Query Migration Examples
```csharp
// VB6 ADO Query
Dim rs As ADODB.Recordset
Set rs = New ADODB.Recordset
rs.Open "SELECT * FROM Customers WHERE City = '" & city & "'", conn

// .NET Core EF Core
var customers = await _context.Customers
    .Where(c => c.City == city)
    .ToListAsync();
```

#### Stored Procedure Migration
```csharp
// VB6 ADO Stored Procedure Call
Dim cmd As ADODB.Command
Set cmd = New ADODB.Command
cmd.ActiveConnection = conn
cmd.CommandText = "sp_GetCustomerOrders"
cmd.CommandType = adCmdStoredProc
cmd.Parameters.Append cmd.CreateParameter("@CustomerId", adInteger, adParamInput, , customerId)
Set rs = cmd.Execute

// .NET Core EF Core
var orders = await _context.Orders
    .FromSqlRaw("EXEC sp_GetCustomerOrders @CustomerId = {0}", customerId)
    .ToListAsync();
```

### Database Migration Scripts

#### Initial Migration
```bash
# Create initial migration
dotnet ef migrations add InitialCreate -p VB6ToDotNetCore.Data -s VB6ToDotNetCore.Web

# Apply migration
dotnet ef database update -p VB6ToDotNetCore.Data -s VB6ToDotNetCore.Web
```

#### Data Migration Script
```csharp
using Microsoft.EntityFrameworkCore;
using VB6ToDotNetCore.Data.Context;

namespace VB6ToDotNetCore.Data.Migrations;

public static class DataMigrationHelper
{
    public static async Task MigrateCustomersAsync(ApplicationDbContext context, string oldConnectionString)
    {
        // Connect to old database
        using var oldConnection = new SqlConnection(oldConnectionString);
        await oldConnection.OpenAsync();

        // Extract data from old database
        var oldCustomers = await oldConnection.QueryAsync<dynamic>(
            "SELECT CustomerID, CustomerName, Email, Phone, Address FROM Customers");

        // Transform and insert into new database
        foreach (var oldCustomer in oldCustomers)
        {
            var newCustomer = new Customer
            {
                Id = oldCustomer.CustomerID,
                Name = oldCustomer.CustomerName,
                Email = oldCustomer.Email,
                Phone = oldCustomer.Phone,
                Address = oldCustomer.Address,
                CreatedDate = DateTime.Now
            };

            context.Customers.Add(newCustomer);
        }

        await context.SaveChangesAsync();
    }
}
```

## Business Logic Conversion

### Service Layer Implementation

#### Business Service Base Class
```csharp
using Microsoft.Extensions.Logging;
using VB6ToDotNetCore.Core.Interfaces;

namespace VB6ToDotNetCore.Core.Services;

public abstract class BaseService
{
    protected readonly ILogger _logger;
    protected readonly IUnitOfWork _unitOfWork;

    protected BaseService(ILogger logger, IUnitOfWork unitOfWork)
    {
        _logger = logger;
        _unitOfWork = unitOfWork;
    }

    protected async Task<T> ExecuteWithLoggingAsync<T>(Func<Task<T>> operation, string operationName)
    {
        try
        {
            _logger.LogInformation("Starting {OperationName}", operationName);
            var result = await operation();
            _logger.LogInformation("Completed {OperationName}", operationName);
            return result;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error in {OperationName}: {Message}", operationName, ex.Message);
            throw;
        }
    }
}
```

#### Customer Service Implementation
```csharp
using VB6ToDotNetCore.Core.Entities;
using VB6ToDotNetCore.Core.Interfaces;
using VB6ToDotNetCore.Core.Validators;

namespace VB6ToDotNetCore.Core.Services;

public class CustomerService : BaseService, ICustomerService
{
    private readonly ICustomerRepository _customerRepository;
    private readonly CustomerValidator _validator;

    public CustomerService(
        ILogger<CustomerService> logger,
        IUnitOfWork unitOfWork,
        ICustomerRepository customerRepository,
        CustomerValidator validator)
        : base(logger, unitOfWork)
    {
        _customerRepository = customerRepository;
        _validator = validator;
    }

    public async Task<Customer> GetCustomerByIdAsync(int id)
    {
        return await ExecuteWithLoggingAsync(
            () => _customerRepository.GetByIdAsync(id),
            $"GetCustomerById {id}");
    }

    public async Task<IEnumerable<Customer>> GetAllCustomersAsync()
    {
        return await ExecuteWithLoggingAsync(
            () => _customerRepository.GetAllAsync(),
            "GetAllCustomers");
    }

    public async Task<Customer> CreateCustomerAsync(Customer customer)
    {
        return await ExecuteWithLoggingAsync(async () =>
        {
            // Validate customer
            var validationResult = await _validator.ValidateAsync(customer);
            if (!validationResult.IsValid)
            {
                throw new ValidationException(validationResult.Errors);
            }

            // Check for duplicate email
            var existingCustomer = await _customerRepository.GetByEmailAsync(customer.Email);
            if (existingCustomer != null)
            {
                throw new DuplicateCustomerException("Customer with this email already exists");
            }

            // Create customer
            customer.CreatedDate = DateTime.UtcNow;
            var createdCustomer = await _customerRepository.AddAsync(customer);

            await _unitOfWork.SaveChangesAsync();

            return createdCustomer;
        }, "CreateCustomer");
    }

    public async Task UpdateCustomerAsync(Customer customer)
    {
        await ExecuteWithLoggingAsync(async () =>
        {
            // Validate customer
            var validationResult = await _validator.ValidateAsync(customer);
            if (!validationResult.IsValid)
            {
                throw new ValidationException(validationResult.Errors);
            }

            // Check if customer exists
            var existingCustomer = await _customerRepository.GetByIdAsync(customer.Id);
            if (existingCustomer == null)
            {
                throw new CustomerNotFoundException($"Customer with ID {customer.Id} not found");
            }

            // Update customer
            await _customerRepository.UpdateAsync(customer);
            await _unitOfWork.SaveChangesAsync();
        }, $"UpdateCustomer {customer.Id}");
    }

    public async Task DeleteCustomerAsync(int id)
    {
        await ExecuteWithLoggingAsync(async () =>
        {
            var customer = await _customerRepository.GetByIdAsync(id);
            if (customer == null)
            {
                throw new CustomerNotFoundException($"Customer with ID {id} not found");
            }

            await _customerRepository.DeleteAsync(id);
            await _unitOfWork.SaveChangesAsync();
        }, $"DeleteCustomer {id}");
    }
}
```

### Validation Implementation

#### FluentValidation Setup
```csharp
using FluentValidation;
using VB6ToDotNetCore.Core.Entities;

namespace VB6ToDotNetCore.Core.Validators;

public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(c => c.Name)
            .NotEmpty().WithMessage("Customer name is required")
            .Length(2, 100).WithMessage("Customer name must be between 2 and 100 characters");

        RuleFor(c => c.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(255).WithMessage("Email must not exceed 255 characters");

        RuleFor(c => c.Phone)
            .MaximumLength(20).WithMessage("Phone number must not exceed 20 characters")
            .Matches(@"^[\+]?[1-9][\d]{0,15}$").WithMessage("Invalid phone number format")
            .When(c => !string.IsNullOrEmpty(c.Phone));

        RuleFor(c => c.Address)
            .MaximumLength(500).WithMessage("Address must not exceed 500 characters");
    }
}
```

### Unit of Work Pattern

#### IUnitOfWork Interface
```csharp
using VB6ToDotNetCore.Core.Interfaces;

namespace VB6ToDotNetCore.Core.Interfaces;

public interface IUnitOfWork : IDisposable
{
    ICustomerRepository Customers { get; }
    IOrderRepository Orders { get; }
    IProductRepository Products { get; }

    Task<int> SaveChangesAsync();
    Task BeginTransactionAsync();
    Task CommitTransactionAsync();
    Task RollbackTransactionAsync();
}
```

#### UnitOfWork Implementation
```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Storage;
using VB6ToDotNetCore.Core.Interfaces;
using VB6ToDotNetCore.Data.Context;
using VB6ToDotNetCore.Data.Repositories;

namespace VB6ToDotNetCore.Data;

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private IDbContextTransaction _transaction;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }

    private ICustomerRepository _customers;
    public ICustomerRepository Customers =>
        _customers ??= new CustomerRepository(_context);

    private IOrderRepository _orders;
    public IOrderRepository Orders =>
        _orders ??= new OrderRepository(_context);

    private IProductRepository _products;
    public IProductRepository Products =>
        _products ??= new ProductRepository(_context);

    public async Task<int> SaveChangesAsync()
    {
        return await _context.SaveChangesAsync();
    }

    public async Task BeginTransactionAsync()
    {
        _transaction = await _context.Database.BeginTransactionAsync();
    }

    public async Task CommitTransactionAsync()
    {
        await _transaction.CommitAsync();
    }

    public async Task RollbackTransactionAsync()
    {
        await _transaction.RollbackAsync();
    }

    public void Dispose()
    {
        _context.Dispose();
        _transaction?.Dispose();
    }
}
```

## User Interface Migration

### WPF Application Setup

#### MainWindow.xaml
```xaml
<Window x:Class="VB6ToDotNetCore.UI.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:VB6ToDotNetCore.UI"
        mc:Ignorable="d"
        Title="VB6 to .NET Core Application" Height="600" Width="800"
        WindowStartupLocation="CenterScreen">
    <Window.Resources>
        <Style TargetType="Button" x:Key="ModernButton">
            <Setter Property="Background" Value="#0078D4"/>
            <Setter Property="Foreground" Value="White"/>
            <Setter Property="Padding" Value="10,5"/>
            <Setter Property="Margin" Value="5"/>
            <Setter Property="BorderThickness" Value="0"/>
            <Setter Property="Cursor" Value="Hand"/>
            <Style.Triggers>
                <Trigger Property="IsMouseOver" Value="True">
                    <Setter Property="Background" Value="#106EBE"/>
                </Trigger>
            </Style.Triggers>
        </Style>
    </Window.Resources>

    <DockPanel>
        <Menu DockPanel.Dock="Top">
            <MenuItem Header="_File">
                <MenuItem Header="_New Customer" Click="NewCustomerMenuItem_Click"/>
                <MenuItem Header="_New Order" Click="NewOrderMenuItem_Click"/>
                <Separator/>
                <MenuItem Header="E_xit" Click="ExitMenuItem_Click"/>
            </MenuItem>
            <MenuItem Header="_View">
                <MenuItem Header="_Customers" Click="ViewCustomersMenuItem_Click"/>
                <MenuItem Header="_Orders" Click="ViewOrdersMenuItem_Click"/>
                <MenuItem Header="_Products" Click="ViewProductsMenuItem_Click"/>
            </MenuItem>
            <MenuItem Header="_Help">
                <MenuItem Header="_About" Click="AboutMenuItem_Click"/>
            </MenuItem>
        </Menu>

        <StatusBar DockPanel.Dock="Bottom">
            <StatusBarItem>
                <TextBlock Name="StatusTextBlock" Text="Ready"/>
            </StatusBarItem>
            <StatusBarItem HorizontalAlignment="Right">
                <TextBlock Name="UserTextBlock" Text="User: "/>
            </StatusBarItem>
        </StatusBar>

        <TabControl Name="MainTabControl">
            <TabItem Header="Dashboard">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="*"/>
                    </Grid.RowDefinitions>

                    <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="10">
                        <Button Content="Refresh Dashboard" Style="{StaticResource ModernButton}"
                                Click="RefreshDashboardButton_Click"/>
                    </StackPanel>

                    <DataGrid Grid.Row="1" Name="DashboardDataGrid" Margin="10"
                             AutoGenerateColumns="False" IsReadOnly="True">
                        <DataGrid.Columns>
                            <DataGridTextColumn Header="Metric" Binding="{Binding Metric}"/>
                            <DataGridTextColumn Header="Value" Binding="{Binding Value}"/>
                            <DataGridTextColumn Header="Change" Binding="{Binding Change}"/>
                        </DataGrid.Columns>
                    </DataGrid>
                </Grid>
            </TabItem>

            <TabItem Header="Customers">
                <Grid>
                    <Grid.RowDefinitions>
                        <RowDefinition Height="Auto"/>
                        <RowDefinition Height="*"/>
                    </Grid.RowDefinitions>

                    <StackPanel Grid.Row="0" Orientation="Horizontal" Margin="10">
                        <Button Content="Add Customer" Style="{StaticResource ModernButton}"
                                Click="AddCustomerButton_Click"/>
                        <Button Content="Edit Customer" Style="{StaticResource ModernButton}"
                                Click="EditCustomerButton_Click"/>
                        <Button Content="Delete Customer" Style="{StaticResource ModernButton}"
                                Click="DeleteCustomerButton_Click"/>
                        <TextBox Name="CustomerSearchTextBox" Width="200" Margin="10,0"
                                TextChanged="CustomerSearchTextBox_TextChanged"/>
                    </StackPanel>

                    <DataGrid Grid.Row="1" Name="CustomersDataGrid" Margin="10"
                             AutoGenerateColumns="False" IsReadOnly="True"
                             SelectionChanged="CustomersDataGrid_SelectionChanged">
                        <DataGrid.Columns>
                            <DataGridTextColumn Header="ID" Binding="{Binding Id}"/>
                            <DataGridTextColumn Header="Name" Binding="{Binding Name}"/>
                            <DataGridTextColumn Header="Email" Binding="{Binding Email}"/>
                            <DataGridTextColumn Header="Phone" Binding="{Binding Phone}"/>
                            <DataGridTextColumn Header="Created Date" Binding="{Binding CreatedDate}"/>
                        </DataGrid.Columns>
                    </DataGrid>
                </Grid>
            </TabItem>
        </TabControl>
    </DockPanel>
</Window>
```

#### MainWindow.xaml.cs
```csharp
using System.Windows;
using System.Windows.Controls;
using Microsoft.Extensions.DependencyInjection;
using VB6ToDotNetCore.Core.Interfaces;
using VB6ToDotNetCore.UI.ViewModels;

namespace VB6ToDotNetCore.UI;

public partial class MainWindow : Window
{
    private readonly ICustomerService _customerService;
    private readonly IOrderService _orderService;
    private readonly MainViewModel _viewModel;

    public MainWindow()
    {
        InitializeComponent();

        // Setup dependency injection
        var serviceProvider = SetupDependencyInjection();
        _customerService = serviceProvider.GetRequiredService<ICustomerService>();
        _orderService = serviceProvider.GetRequiredService<IOrderService>();
        _viewModel = new MainViewModel(_customerService, _orderService);

        DataContext = _viewModel;

        Loaded += MainWindow_Loaded;
    }

    private IServiceProvider SetupDependencyInjection()
    {
        var services = new ServiceCollection();

        // Register services
        services.AddSingleton<ICustomerService, CustomerService>();
        services.AddSingleton<IOrderService, OrderService>();
        services.AddSingleton<IUnitOfWork, UnitOfWork>();
        services.AddSingleton<ApplicationDbContext>(provider =>
        {
            var options = new DbContextOptionsBuilder<ApplicationDbContext>()
                .UseSqlServer("Server=localhost;Database=VB6ToDotNetCore;Trusted_Connection=True;")
                .Options;
            return new ApplicationDbContext(options);
        });

        return services.BuildServiceProvider();
    }

    private async void MainWindow_Loaded(object sender, RoutedEventArgs e)
    {
        await LoadDashboardData();
        await LoadCustomers();
    }

    private async Task LoadDashboardData()
    {
        try
        {
            var dashboardData = await _viewModel.LoadDashboardDataAsync();
            DashboardDataGrid.ItemsSource = dashboardData;
            StatusTextBlock.Text = "Dashboard loaded successfully";
        }
        catch (Exception ex)
        {
            StatusTextBlock.Text = $"Error loading dashboard: {ex.Message}";
        }
    }

    private async Task LoadCustomers()
    {
        try
        {
            var customers = await _viewModel.LoadCustomersAsync();
            CustomersDataGrid.ItemsSource = customers;
        }
        catch (Exception ex)
        {
            StatusTextBlock.Text = $"Error loading customers: {ex.Message}";
        }
    }

    private void NewCustomerMenuItem_Click(object sender, RoutedEventArgs e)
    {
        var customerWindow = new CustomerWindow(_customerService);
        customerWindow.ShowDialog();
        // Refresh customers list
        _ = LoadCustomers();
    }

    private void ViewCustomersMenuItem_Click(object sender, RoutedEventArgs e)
    {
        MainTabControl.SelectedIndex = 1; // Switch to Customers tab
    }

    private void ExitMenuItem_Click(object sender, RoutedEventArgs e)
    {
        Application.Current.Shutdown();
    }

    private void AddCustomerButton_Click(object sender, RoutedEventArgs e)
    {
        NewCustomerMenuItem_Click(sender, e);
    }

    private async void CustomerSearchTextBox_TextChanged(object sender, TextChangedEventArgs e)
    {
        var searchText = CustomerSearchTextBox.Text;
        var filteredCustomers = await _viewModel.SearchCustomersAsync(searchText);
        CustomersDataGrid.ItemsSource = filteredCustomers;
    }

    private void CustomersDataGrid_SelectionChanged(object sender, SelectionChangedEventArgs e)
    {
        var selectedCustomer = CustomersDataGrid.SelectedItem as Customer;
        if (selectedCustomer != null)
        {
            StatusTextBlock.Text = $"Selected customer: {selectedCustomer.Name}";
        }
    }

    private async void RefreshDashboardButton_Click(object sender, RoutedEventArgs e)
    {
        await LoadDashboardData();
    }
}
```

### MVVM Pattern Implementation

#### ViewModel Base Class
```csharp
using System.ComponentModel;
using System.Runtime.CompilerServices;

namespace VB6ToDotNetCore.UI.ViewModels;

public abstract class ViewModelBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T field, T value, [CallerMemberName] string propertyName = null)
    {
        if (EqualityComparer<T>.Default.Equals(field, value)) return false;
        field = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
```

#### Main ViewModel
```csharp
using System.Collections.ObjectModel;
using VB6ToDotNetCore.Core.Entities;
using VB6ToDotNetCore.Core.Interfaces;

namespace VB6ToDotNetCore.UI.ViewModels;

public class MainViewModel : ViewModelBase
{
    private readonly ICustomerService _customerService;
    private readonly IOrderService _orderService;

    private ObservableCollection<Customer> _customers;
    public ObservableCollection<Customer> Customers
    {
        get => _customers;
        set => SetProperty(ref _customers, value);
    }

    private ObservableCollection<DashboardMetric> _dashboardMetrics;
    public ObservableCollection<DashboardMetric> DashboardMetrics
    {
        get => _dashboardMetrics;
        set => SetProperty(ref _dashboardMetrics, value);
    }

    public MainViewModel(ICustomerService customerService, IOrderService orderService)
    {
        _customerService = customerService;
        _orderService = orderService;
        Customers = new ObservableCollection<Customer>();
        DashboardMetrics = new ObservableCollection<DashboardMetric>();
    }

    public async Task LoadCustomersAsync()
    {
        var customers = await _customerService.GetAllCustomersAsync();
        Customers.Clear();
        foreach (var customer in customers)
        {
            Customers.Add(customer);
        }
    }

    public async Task<IEnumerable<Customer>> SearchCustomersAsync(string searchText)
    {
        if (string.IsNullOrWhiteSpace(searchText))
        {
            return await _customerService.GetAllCustomersAsync();
        }

        return await _customerService.SearchCustomersAsync(searchText);
    }

    public async Task<List<DashboardMetric>> LoadDashboardDataAsync()
    {
        var metrics = new List<DashboardMetric>();

        // Load various metrics
        var totalCustomers = await _customerService.GetTotalCustomerCountAsync();
        metrics.Add(new DashboardMetric
        {
            Metric = "Total Customers",
            Value = totalCustomers.ToString(),
            Change = "+5%" // Calculate actual change
        });

        var totalOrders = await _orderService.GetTotalOrderCountAsync();
        metrics.Add(new DashboardMetric
        {
            Metric = "Total Orders",
            Value = totalOrders.ToString(),
            Change = "+12%"
        });

        var totalRevenue = await _orderService.GetTotalRevenueAsync();
        metrics.Add(new DashboardMetric
        {
            Metric = "Total Revenue",
            Value = $"${totalRevenue:N2}",
            Change = "+8%"
        });

        return metrics;
    }
}

public class DashboardMetric
{
    public string Metric { get; set; }
    public string Value { get; set; }
    public string Change { get; set; }
}
```

## API and Integration Migration

### Web API Controllers

#### Base Controller
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;

namespace VB6ToDotNetCore.Web.Controllers;

[ApiController]
[Route("api/[controller]")]
public abstract class BaseController : ControllerBase
{
    protected readonly ILogger _logger;

    protected BaseController(ILogger logger)
    {
        _logger = logger;
    }

    protected IActionResult HandleException(Exception ex, string operation)
    {
        _logger.LogError(ex, "Error in {Operation}: {Message}", operation, ex.Message);

        return ex switch
        {
            ValidationException => BadRequest(ex.Message),
            NotFoundException => NotFound(ex.Message),
            UnauthorizedAccessException => Unauthorized(ex.Message),
            _ => StatusCode(500, "An unexpected error occurred")
        };
    }

    protected IActionResult Success(object data = null)
    {
        return data == null ? Ok() : Ok(data);
    }

    protected IActionResult Created(string routeName, object routeValues, object data)
    {
        return CreatedAtRoute(routeName, routeValues, data);
    }
}
```

#### Customers Controller
```csharp
using Microsoft.AspNetCore.Mvc;
using VB6ToDotNetCore.Core.Entities;
using VB6ToDotNetCore.Core.Interfaces;
using VB6ToDotNetCore.Web.Models;

namespace VB6ToDotNetCore.Web.Controllers;

public class CustomersController : BaseController
{
    private readonly ICustomerService _customerService;

    public CustomersController(
        ILogger<CustomersController> logger,
        ICustomerService customerService)
        : base(logger)
    {
        _customerService = customerService;
    }

    [HttpGet]
    public async Task<IActionResult> GetCustomers(
        [FromQuery] string search = null,
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 10)
    {
        try
        {
            IEnumerable<Customer> customers;

            if (!string.IsNullOrWhiteSpace(search))
            {
                customers = await _customerService.SearchCustomersAsync(search);
            }
            else
            {
                customers = await _customerService.GetAllCustomersAsync();
            }

            // Apply pagination
            var paginatedCustomers = customers
                .Skip((page - 1) * pageSize)
                .Take(pageSize)
                .ToList();

            var totalCount = customers.Count();
            var totalPages = (int)Math.Ceiling(totalCount / (double)pageSize);

            var result = new PaginatedResult<Customer>
            {
                Items = paginatedCustomers,
                Page = page,
                PageSize = pageSize,
                TotalCount = totalCount,
                TotalPages = totalPages
            };

            return Success(result);
        }
        catch (Exception ex)
        {
            return HandleException(ex, "GetCustomers");
        }
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetCustomer(int id)
    {
        try
        {
            var customer = await _customerService.GetCustomerByIdAsync(id);
            if (customer == null)
            {
                return NotFound($"Customer with ID {id} not found");
            }

            return Success(customer);
        }
        catch (Exception ex)
        {
            return HandleException(ex, "GetCustomer");
        }
    }

    [HttpPost]
    public async Task<IActionResult> CreateCustomer([FromBody] CreateCustomerRequest request)
    {
        try
        {
            var customer = new Customer
            {
                Name = request.Name,
                Email = request.Email,
                Phone = request.Phone,
                Address = request.Address
            };

            var createdCustomer = await _customerService.CreateCustomerAsync(customer);

            return Created("GetCustomer", new { id = createdCustomer.Id }, createdCustomer);
        }
        catch (Exception ex)
        {
            return HandleException(ex, "CreateCustomer");
        }
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateCustomer(int id, [FromBody] UpdateCustomerRequest request)
    {
        try
        {
            var customer = await _customerService.GetCustomerByIdAsync(id);
            if (customer == null)
            {
                return NotFound($"Customer with ID {id} not found");
            }

            customer.Name = request.Name;
            customer.Email = request.Email;
            customer.Phone = request.Phone;
            customer.Address = request.Address;

            await _customerService.UpdateCustomerAsync(customer);

            return Success();
        }
        catch (Exception ex)
        {
            return HandleException(ex, "UpdateCustomer");
        }
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteCustomer(int id)
    {
        try
        {
            await _customerService.DeleteCustomerAsync(id);
            return Success();
        }
        catch (Exception ex)
        {
            return HandleException(ex, "DeleteCustomer");
        }
    }
}
```

### API Models

#### Request/Response Models
```csharp
namespace VB6ToDotNetCore.Web.Models;

public class CreateCustomerRequest
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    public string Address { get; set; }
}

public class UpdateCustomerRequest
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    public string Address { get; set; }
}

public class PaginatedResult<T>
{
    public IEnumerable<T> Items { get; set; }
    public int Page { get; set; }
    public int PageSize { get; set; }
    public int TotalCount { get; set; }
    public int TotalPages { get; set; }
}
```

### Authentication and Authorization

#### JWT Authentication Setup
```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

namespace VB6ToDotNetCore.Web.Extensions;

public static class AuthenticationExtensions
{
    public static IServiceCollection AddJwtAuthentication(this IServiceCollection services, IConfiguration configuration)
    {
        var jwtSettings = configuration.GetSection("Jwt");

        services.AddAuthentication(options =>
        {
            options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
        })
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = jwtSettings["Issuer"],
                ValidAudience = jwtSettings["Audience"],
                IssuerSigningKey = new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes(jwtSettings["Key"]))
            };

            options.Events = new JwtBearerEvents
            {
                OnAuthenticationFailed = context =>
                {
                    // Log authentication failures
                    var logger = context.HttpContext.RequestServices.GetRequiredService<ILogger<JwtBearerEvents>>();
                    logger.LogWarning("Authentication failed: {Exception}", context.Exception.Message);
                    return Task.CompletedTask;
                }
            };
        });

        return services;
    }
}
```

#### Auth Controller
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.IdentityModel.Tokens;
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using VB6ToDotNetCore.Web.Models;

namespace VB6ToDotNetCore.Web.Controllers;

[ApiController]
[Route("api/[controller]")]
public class AuthController : BaseController
{
    private readonly IConfiguration _configuration;
    private readonly IUserService _userService;

    public AuthController(
        ILogger<AuthController> logger,
        IConfiguration configuration,
        IUserService userService)
        : base(logger)
    {
        _configuration = configuration;
        _userService = userService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        try
        {
            // Validate user credentials
            var user = await _userService.ValidateUserAsync(request.Username, request.Password);
            if (user == null)
            {
                return Unauthorized("Invalid username or password");
            }

            // Generate JWT token
            var token = GenerateJwtToken(user);

            var response = new LoginResponse
            {
                Token = token,
                User = new UserDto
                {
                    Id = user.Id,
                    Username = user.Username,
                    Email = user.Email,
                    Roles = user.Roles
                }
            };

            return Success(response);
        }
        catch (Exception ex)
        {
            return HandleException(ex, "Login");
        }
    }

    [HttpPost("refresh")]
    [Authorize]
    public async Task<IActionResult> RefreshToken()
    {
        try
        {
            var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
            if (string.IsNullOrEmpty(userId))
            {
                return Unauthorized();
            }

            var user = await _userService.GetUserByIdAsync(int.Parse(userId));
            if (user == null)
            {
                return Unauthorized();
            }

            var newToken = GenerateJwtToken(user);

            return Success(new { Token = newToken });
        }
        catch (Exception ex)
        {
            return HandleException(ex, "RefreshToken");
        }
    }

    private string GenerateJwtToken(User user)
    {
        var jwtSettings = _configuration.GetSection("Jwt");

        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        // Add role claims
        foreach (var role in user.Roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSettings["Key"]));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: jwtSettings["Issuer"],
            audience: jwtSettings["Audience"],
            claims: claims,
            expires: DateTime.Now.AddHours(8),
            signingCredentials: creds);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

## Testing Strategy

### Unit Testing Setup

#### Test Base Class
```csharp
using AutoFixture;
using AutoFixture.AutoMoq;
using Microsoft.Extensions.Logging;
using Moq;
using Xunit.Abstractions;

namespace VB6ToDotNetCore.Core.Tests;

public abstract class TestBase
{
    protected readonly IFixture Fixture;
    protected readonly Mock<ILogger> LoggerMock;

    protected TestBase(ITestOutputHelper outputHelper)
    {
        Fixture = new Fixture().Customize(new AutoMoqCustomization());
        LoggerMock = new Mock<ILogger>();

        // Configure AutoFixture
        Fixture.Behaviors.Add(new OmitOnRecursionBehavior());
        Fixture.Customizations.Add(new TestOutputWriter(outputHelper));
    }

    protected T Create<T>()
    {
        return Fixture.Create<T>();
    }

    protected Mock<T> Mock<T>() where T : class
    {
        return Fixture.Freeze<Mock<T>>();
    }
}
```

#### Customer Service Tests
```csharp
using AutoFixture.Xunit2;
using FluentValidation.TestHelper;
using Moq;
using Shouldly;
using VB6ToDotNetCore.Core.Entities;
using VB6ToDotNetCore.Core.Exceptions;
using VB6ToDotNetCore.Core.Services;
using VB6ToDotNetCore.Core.Validators;

namespace VB6ToDotNetCore.Core.Tests.Services;

public class CustomerServiceTests : TestBase
{
    private readonly Mock<ICustomerRepository> _customerRepositoryMock;
    private readonly Mock<IUnitOfWork> _unitOfWorkMock;
    private readonly CustomerService _customerService;

    public CustomerServiceTests(ITestOutputHelper outputHelper) : base(outputHelper)
    {
        _customerRepositoryMock = Mock<ICustomerRepository>();
        _unitOfWorkMock = Mock<IUnitOfWork>();
        _customerService = new CustomerService(
            LoggerMock.Object,
            _unitOfWorkMock.Object,
            _customerRepositoryMock.Object,
            new CustomerValidator());
    }

    [Theory, AutoData]
    public async Task CreateCustomerAsync_ValidCustomer_ReturnsCreatedCustomer(Customer customer)
    {
        // Arrange
        _customerRepositoryMock
            .Setup(repo => repo.GetByEmailAsync(customer.Email))
            .ReturnsAsync((Customer)null);

        _customerRepositoryMock
            .Setup(repo => repo.AddAsync(It.IsAny<Customer>()))
            .ReturnsAsync((Customer c) => c);

        _unitOfWorkMock
            .Setup(uow => uow.SaveChangesAsync())
            .ReturnsAsync(1);

        // Act
        var result = await _customerService.CreateCustomerAsync(customer);

        // Assert
        result.ShouldNotBeNull();
        result.Name.ShouldBe(customer.Name);
        result.Email.ShouldBe(customer.Email);
        _customerRepositoryMock.Verify(repo => repo.AddAsync(customer), Times.Once);
        _unitOfWorkMock.Verify(uow => uow.SaveChangesAsync(), Times.Once);
    }

    [Fact]
    public async Task CreateCustomerAsync_DuplicateEmail_ThrowsException()
    {
        // Arrange
        var customer = Create<Customer>();
        var existingCustomer = Create<Customer>();
        existingCustomer.Email = customer.Email;

        _customerRepositoryMock
            .Setup(repo => repo.GetByEmailAsync(customer.Email))
            .ReturnsAsync(existingCustomer);

        // Act & Assert
        var exception = await Should.ThrowAsync<DuplicateCustomerException>(
            () => _customerService.CreateCustomerAsync(customer));

        exception.Message.ShouldContain("already exists");
    }

    [Theory]
    [InlineData("", "Valid email")]
    [InlineData("Valid Name", "")]
    [InlineData(null, "valid@email.com")]
    public async Task CreateCustomerAsync_InvalidData_ThrowsValidationException(
        string name, string email)
    {
        // Arrange
        var customer = new Customer { Name = name, Email = email };

        // Act & Assert
        await Should.ThrowAsync<ValidationException>(
            () => _customerService.CreateCustomerAsync(customer));
    }

    [Fact]
    public async Task GetCustomerByIdAsync_ExistingCustomer_ReturnsCustomer()
    {
        // Arrange
        var customer = Create<Customer>();
        _customerRepositoryMock
            .Setup(repo => repo.GetByIdAsync(customer.Id))
            .ReturnsAsync(customer);

        // Act
        var result = await _customerService.GetCustomerByIdAsync(customer.Id);

        // Assert
        result.ShouldBe(customer);
    }

    [Fact]
    public async Task GetCustomerByIdAsync_NonExistingCustomer_ReturnsNull()
    {
        // Arrange
        _customerRepositoryMock
            .Setup(repo => repo.GetByIdAsync(It.IsAny<int>()))
            .ReturnsAsync((Customer)null);

        // Act
        var result = await _customerService.GetCustomerByIdAsync(999);

        // Assert
        result.ShouldBeNull();
    }
}
```

### Integration Testing

#### Database Integration Tests
```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using VB6ToDotNetCore.Data.Context;
using Xunit;

namespace VB6ToDotNetCore.Data.Tests.Integration;

[Collection("Database collection")]
public class CustomerRepositoryIntegrationTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    private readonly CustomerRepository _repository;

    public CustomerRepositoryIntegrationTests()
    {
        var configuration = new ConfigurationBuilder()
            .AddJsonFile("appsettings.Test.json")
            .Build();

        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlServer(configuration.GetConnectionString("TestConnection"))
            .Options;

        _context = new ApplicationDbContext(options);
        _repository = new CustomerRepository(_context);

        // Ensure database is created
        _context.Database.EnsureCreated();
    }

    [Fact]
    public async Task AddAsync_ValidCustomer_AddsToDatabase()
    {
        // Arrange
        var customer = new Customer
        {
            Name = "Integration Test Customer",
            Email = "integration@test.com",
            Phone = "123-456-7890",
            CreatedDate = DateTime.UtcNow
        };

        // Act
        var result = await _repository.AddAsync(customer);

        // Assert
        result.Id.ShouldBeGreaterThan(0);
        result.Name.ShouldBe(customer.Name);
        result.Email.ShouldBe(customer.Email);

        // Verify in database
        var savedCustomer = await _context.Customers.FindAsync(result.Id);
        savedCustomer.ShouldNotBeNull();
        savedCustomer.Name.ShouldBe(customer.Name);
    }

    [Fact]
    public async Task GetByIdAsync_ExistingCustomer_ReturnsCustomer()
    {
        // Arrange
        var customer = new Customer
        {
            Name = "Test Customer",
            Email = "test@example.com",
            CreatedDate = DateTime.UtcNow
        };

        _context.Customers.Add(customer);
        await _context.SaveChangesAsync();

        // Act
        var result = await _repository.GetByIdAsync(customer.Id);

        // Assert
        result.ShouldNotBeNull();
        result.Id.ShouldBe(customer.Id);
        result.Name.ShouldBe(customer.Name);
    }

    public void Dispose()
    {
        _context.Database.EnsureDeleted();
        _context.Dispose();
    }
}
```

### API Integration Tests
```csharp
using Microsoft.AspNetCore.Mvc.Testing;
using System.Net.Http.Json;
using VB6ToDotNetCore.Web;
using Xunit;

namespace VB6ToDotNetCore.Web.Tests.Integration;

public class CustomersApiIntegrationTests : IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly WebApplicationFactory<Startup> _factory;
    private readonly HttpClient _client;

    public CustomersApiIntegrationTests(WebApplicationFactory<Startup> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetCustomers_ReturnsSuccessAndCustomers()
    {
        // Act
        var response = await _client.GetAsync("/api/customers");

        // Assert
        response.EnsureSuccessStatusCode();
        var customers = await response.Content.ReadFromJsonAsync<List<Customer>>();
        customers.ShouldNotBeNull();
    }

    [Fact]
    public async Task CreateCustomer_ValidData_ReturnsCreatedCustomer()
    {
        // Arrange
        var request = new
        {
            Name = "API Test Customer",
            Email = "api@test.com",
            Phone = "555-0123"
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/customers", request);

        // Assert
        response.StatusCode.ShouldBe(HttpStatusCode.Created);
        var createdCustomer = await response.Content.ReadFromJsonAsync<Customer>();
        createdCustomer.ShouldNotBeNull();
        createdCustomer.Name.ShouldBe(request.Name);
        createdCustomer.Email.ShouldBe(request.Email);
    }

    [Fact]
    public async Task GetCustomer_ExistingId_ReturnsCustomer()
    {
        // Arrange - Create a customer first
        var createRequest = new
        {
            Name = "Get Test Customer",
            Email = "get@test.com"
        };

        var createResponse = await _client.PostAsJsonAsync("/api/customers", createRequest);
        var createdCustomer = await createResponse.Content.ReadFromJsonAsync<Customer>();

        // Act
        var getResponse = await _client.GetAsync($"/api/customers/{createdCustomer.Id}");

        // Assert
        getResponse.EnsureSuccessStatusCode();
        var retrievedCustomer = await getResponse.Content.ReadFromJsonAsync<Customer>();
        retrievedCustomer.Id.ShouldBe(createdCustomer.Id);
        retrievedCustomer.Name.ShouldBe(createdCustomer.Name);
    }

    [Fact]
    public async Task GetCustomer_NonExistingId_ReturnsNotFound()
    {
        // Act
        var response = await _client.GetAsync("/api/customers/99999");

        // Assert
        response.StatusCode.ShouldBe(HttpStatusCode.NotFound);
    }
}
```

## Security Implementation

### Input Validation and Sanitization

#### XSS Prevention
```csharp
using System.Text.RegularExpressions;

namespace VB6ToDotNetCore.Core.Security;

public static class InputSanitizer
{
    private static readonly Regex HtmlTagRegex = new Regex(@"<[^>]*>", RegexOptions.Compiled);

    public static string SanitizeHtml(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;

        // Remove HTML tags
        return HtmlTagRegex.Replace(input, string.Empty);
    }

    public static string SanitizeSql(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;

        // Basic SQL injection prevention (EF handles most of this)
        return input.Replace("'", "''");
    }

    public static bool ContainsXssPayload(string input)
    {
        if (string.IsNullOrEmpty(input))
            return false;

        var xssPatterns = new[]
        {
            @"<script[^>]*>.*?</script>",
            @"javascript:",
            @"vbscript:",
            @"onload\s*=",
            @"onerror\s*="
        };

        return xssPatterns.Any(pattern =>
            Regex.IsMatch(input, pattern, RegexOptions.IgnoreCase));
    }
}
```

### Data Protection

#### Encryption Service
```csharp
using Microsoft.AspNetCore.DataProtection;
using System.Text;

namespace VB6ToDotNetCore.Core.Security;

public interface IEncryptionService
{
    string Encrypt(string plainText);
    string Decrypt(string cipherText);
    string HashPassword(string password);
    bool VerifyPassword(string password, string hash);
}

public class EncryptionService : IEncryptionService
{
    private readonly IDataProtector _protector;

    public EncryptionService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("VB6ToDotNetCore.Data.v1");
    }

    public string Encrypt(string plainText)
    {
        if (string.IsNullOrEmpty(plainText))
            return plainText;

        var plainTextBytes = Encoding.UTF8.GetBytes(plainText);
        var encryptedBytes = _protector.Protect(plainTextBytes);
        return Convert.ToBase64String(encryptedBytes);
    }

    public string Decrypt(string cipherText)
    {
        if (string.IsNullOrEmpty(cipherText))
            return cipherText;

        try
        {
            var cipherTextBytes = Convert.FromBase64String(cipherText);
            var decryptedBytes = _protector.Unprotect(cipherTextBytes);
            return Encoding.UTF8.GetString(decryptedBytes);
        }
        catch
        {
            // Return original if decryption fails
            return cipherText;
        }
    }

    public string HashPassword(string password)
    {
        return BCrypt.Net.BCrypt.HashPassword(password, BCrypt.Net.BCrypt.GenerateSalt());
    }

    public bool VerifyPassword(string password, string hash)
    {
        return BCrypt.Net.BCrypt.Verify(password, hash);
    }
}
```

### Security Headers Middleware

#### Security Headers Implementation
```csharp
using Microsoft.AspNetCore.Http;
using System.Threading.Tasks;

namespace VB6ToDotNetCore.Web.Middleware;

public class SecurityHeadersMiddleware
{
    private readonly RequestDelegate _next;

    public SecurityHeadersMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // Security headers
        context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
        context.Response.Headers.Add("X-Frame-Options", "DENY");
        context.Response.Headers.Add("X-XSS-Protection", "1; mode=block");
        context.Response.Headers.Add("Referrer-Policy", "strict-origin-when-cross-origin");
        context.Response.Headers.Add("Content-Security-Policy",
            "default-src 'self'; " +
            "script-src 'self' 'unsafe-inline'; " +
            "style-src 'self' 'unsafe-inline'; " +
            "img-src 'self' data: https:; " +
            "font-src 'self'; " +
            "connect-src 'self'");

        // HSTS (HTTP Strict Transport Security)
        if (context.Request.IsHttps)
        {
            context.Response.Headers.Add("Strict-Transport-Security", "max-age=31536000; includeSubDomains");
        }

        await _next(context);
    }
}

// Extension method
public static class SecurityHeadersMiddlewareExtensions
{
    public static IApplicationBuilder UseSecurityHeaders(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<SecurityHeadersMiddleware>();
    }
}
```

### Rate Limiting

#### Rate Limiting Implementation
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Caching.Distributed;
using System.Threading.Tasks;

namespace VB6ToDotNetCore.Web.Middleware;

public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IDistributedCache _cache;
    private readonly ILogger<RateLimitingMiddleware> _logger;

    public RateLimitingMiddleware(
        RequestDelegate next,
        IDistributedCache cache,
        ILogger<RateLimitingMiddleware> logger)
    {
        _next = next;
        _cache = cache;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var clientIp = context.Connection.RemoteIpAddress?.ToString() ?? "unknown";
        var cacheKey = $"rate_limit_{clientIp}";
        var requestCount = await GetRequestCountAsync(cacheKey);

        if (requestCount >= 100) // 100 requests per minute
        {
            _logger.LogWarning("Rate limit exceeded for IP: {ClientIp}", clientIp);
            context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
            await context.Response.WriteAsync("Too many requests. Please try again later.");
            return;
        }

        await IncrementRequestCountAsync(cacheKey);
        await _next(context);
    }

    private async Task<int> GetRequestCountAsync(string cacheKey)
    {
        var cachedValue = await _cache.GetStringAsync(cacheKey);
        return string.IsNullOrEmpty(cachedValue) ? 0 : int.Parse(cachedValue);
    }

    private async Task IncrementRequestCountAsync(string cacheKey)
    {
        var currentCount = await GetRequestCountAsync(cacheKey);
        var newCount = currentCount + 1;

        await _cache.SetStringAsync(cacheKey, newCount.ToString(),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(1)
            });
    }
}
```

## Build and Deployment

### CI/CD Pipeline

#### GitHub Actions Workflow
```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  DOTNET_VERSION: '8.0.x'

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: ${{ env.DOTNET_VERSION }}

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --configuration Release

    - name: Run tests
      run: dotnet test --no-build --configuration Release --verbosity normal --collect:"XPlat Code Coverage"

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/coverage.cobertura.xml

  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test

    steps:
    - uses: actions/checkout@v3

    - name: Run security scan
      uses: securecodewarrior/github-action-security-scan@v1
      with:
        language: csharp

    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: results.sarif

  build-and-push-docker:
    runs-on: ubuntu-latest
    needs: [build-and-test, security-scan]
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push API image
      uses: docker/build-push-action@v4
      with:
        context: .
        file: ./docker/Dockerfile.api
        push: true
        tags: vb6-dotnet-core-api:latest,vb6-dotnet-core-api:${{ github.sha }}

    - name: Build and push Worker image
      uses: docker/build-push-action@v4
      with:
        context: .
        file: ./docker/Dockerfile.worker
        push: true
        tags: vb6-dotnet-core-worker:latest,vb6-dotnet-core-worker:${{ github.sha }}

  deploy-to-staging:
    runs-on: ubuntu-latest
    needs: build-and-push-docker
    if: github.ref == 'refs/heads/main'
    environment: staging

    steps:
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment"
        # Add deployment commands here

  deploy-to-production:
    runs-on: ubuntu-latest
    needs: deploy-to-staging
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production

    steps:
    - name: Deploy to production
      run: |
        echo "Deploying to production environment"
        # Add production deployment commands here
```

### Build Scripts

#### PowerShell Build Script
```powershell
param(
    [string]$Configuration = "Release",
    [switch]$Clean,
    [switch]$Test,
    [switch]$Pack,
    [switch]$Publish,
    [switch]$Docker
)

$ErrorActionPreference = "Stop"

Write-Host "Starting build process..." -ForegroundColor Green

if ($Clean) {
    Write-Host "Cleaning solution..." -ForegroundColor Yellow
    dotnet clean --configuration $Configuration
}

Write-Host "Restoring dependencies..." -ForegroundColor Yellow
dotnet restore

Write-Host "Building solution..." -ForegroundColor Yellow
dotnet build --configuration $Configuration --no-restore

if ($Test) {
    Write-Host "Running tests..." -ForegroundColor Yellow
    dotnet test --configuration $Configuration --no-build --verbosity normal
}

if ($Pack) {
    Write-Host "Creating NuGet packages..." -ForegroundColor Yellow
    dotnet pack --configuration $Configuration --no-build --output ./artifacts
}

if ($Publish) {
    Write-Host "Publishing web application..." -ForegroundColor Yellow
    dotnet publish VB6ToDotNetCore.Web --configuration $Configuration --output ./publish
}

if ($Docker) {
    Write-Host "Building Docker images..." -ForegroundColor Yellow

    # Build API image
    docker build -f docker/Dockerfile.api -t vb6-dotnet-core-api:$Configuration .

    # Build Worker image
    docker build -f docker/Dockerfile.worker -t vb6-dotnet-core-worker:$Configuration .
}

Write-Host "Build completed successfully!" -ForegroundColor Green
```

## Containerization

### Multi-Stage Dockerfiles

#### API Dockerfile
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy csproj files and restore
COPY ["VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj", "VB6ToDotNetCore.Web/"]
COPY ["VB6ToDotNetCore.Core/VB6ToDotNetCore.Core.csproj", "VB6ToDotNetCore.Core/"]
COPY ["VB6ToDotNetCore.Data/VB6ToDotNetCore.Data.csproj", "VB6ToDotNetCore.Data/"]
RUN dotnet restore "VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj"

# Copy source and build
COPY . .
WORKDIR "/src/VB6ToDotNetCore.Web"
RUN dotnet build "VB6ToDotNetCore.Web.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && chown -R appuser:appuser /app
USER appuser

# Copy published app
COPY --from=publish /app/publish .

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

EXPOSE 80
ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Web.dll"]
```

#### Worker Dockerfile
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY ["VB6ToDotNetCore.Worker/VB6ToDotNetCore.Worker.csproj", "VB6ToDotNetCore.Worker/"]
RUN dotnet restore "VB6ToDotNetCore.Worker/VB6ToDotNetCore.Worker.csproj"

COPY . .
WORKDIR "/src/VB6ToDotNetCore.Worker"
RUN dotnet build "VB6ToDotNetCore.Worker.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Worker.csproj" -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/runtime:8.0 AS final
WORKDIR /app
COPY --from=publish /app/publish .

# Create non-root user
RUN adduser --disabled-password --gecos '' workeruser && chown -R workeruser:workeruser /app
USER workeruser

ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Worker.dll"]
```

### Docker Compose Configuration

#### Development Environment
```yaml
version: '3.8'

services:
  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      ACCEPT_EULA: Y
      SA_PASSWORD: YourStrong!Passw0rd
      MSSQL_PID: Express
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql
    networks:
      - vb6network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - vb6network
    restart: unless-stopped

  api:
    build:
      context: .
      dockerfile: docker/Dockerfile.api
    ports:
      - "8080:80"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=YourStrong!Passw0rd;
      - ConnectionStrings__Redis=redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - vb6network
    restart: unless-stopped

  worker:
    build:
      context: .
      dockerfile: docker/Dockerfile.worker
    environment:
      - DOTNET_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User Id=sa;Password=YourStrong!Passw0rd;
    depends_on:
      - db
    networks:
      - vb6network
    restart: unless-stopped

volumes:
  sqlserver_data:
  redis_data:

networks:
  vb6network:
    driver: bridge
```

## Production Deployment

### Kubernetes Manifests

#### API Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-api
  namespace: vb6-dotnet-core
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vb6-api
  template:
    metadata:
      labels:
        app: vb6-api
    spec:
      containers:
      - name: api
        image: vb6-dotnet-core-api:latest
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ASPNETCORE_URLS
          value: "http://+:80"
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: db-connection-string
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: vb6-api-service
  namespace: vb6-dotnet-core
spec:
  selector:
    app: vb6-api
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vb6-api-ingress
  namespace: vb6-dotnet-core
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.yourdomain.com
    secretName: vb6-api-tls
  rules:
  - host: api.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vb6-api-service
            port:
              number: 80
```

### Database Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-db
  namespace: vb6-dotnet-core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vb6-db
  template:
    metadata:
      labels:
        app: vb6-db
    spec:
      containers:
      - name: sqlserver
        image: mcr.microsoft.com/mssql/server:2022-latest
        ports:
        - containerPort: 1433
        env:
        - name: ACCEPT_EULA
          value: "Y"
        - name: SA_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vb6-secrets
              key: db-password
        volumeMounts:
        - name: mssql-data
          mountPath: /var/opt/mssql
        resources:
          limits:
            memory: "2Gi"
            cpu: "1000m"
          requests:
            memory: "1Gi"
            cpu: "500m"
      volumes:
      - name: mssql-data
        persistentVolumeClaim:
          claimName: mssql-pvc

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mssql-pvc
  namespace: vb6-dotnet-core
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

### Horizontal Pod Autoscaler
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vb6-api-hpa
  namespace: vb6-dotnet-core
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vb6-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Post-Migration Activities

### Performance Optimization

#### Database Optimization
```sql
-- Create indexes for better performance
CREATE NONCLUSTERED INDEX IX_Customers_Email ON Customers(Email);
CREATE NONCLUSTERED INDEX IX_Customers_Name ON Customers(Name);
CREATE NONCLUSTERED INDEX IX_Orders_CustomerId ON Orders(CustomerId);
CREATE NONCLUSTERED INDEX IX_Orders_OrderDate ON Orders(OrderDate);

-- Update statistics
UPDATE STATISTICS Customers WITH FULLSCAN;
UPDATE STATISTICS Orders WITH FULLSCAN;
UPDATE STATISTICS Products WITH FULLSCAN;
```

#### Application Performance Monitoring
```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;

namespace VB6ToDotNetCore.Web.Middleware;

public class PerformanceMonitoringMiddleware
{
    private readonly RequestDelegate _next;
    private readonly TelemetryClient _telemetryClient;

    public PerformanceMonitoringMiddleware(RequestDelegate next)
    {
        _next = next;
        _telemetryClient = new TelemetryClient(
            TelemetryConfiguration.CreateDefault());
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var startTime = DateTime.UtcNow;
        var stopwatch = Stopwatch.StartNew();

        try
        {
            await _next(context);
        }
        finally
        {
            stopwatch.Stop();
            var duration = stopwatch.ElapsedMilliseconds;

            // Track request performance
            _telemetryClient.TrackRequest(
                context.Request.Method + " " + context.Request.Path,
                startTime,
                stopwatch.Elapsed,
                context.Response.StatusCode.ToString(),
                context.Response.StatusCode >= 200 && context.Response.StatusCode < 300);

            // Track custom metrics
            _telemetryClient.TrackMetric("RequestDuration", duration);
            _telemetryClient.TrackMetric("ResponseSize",
                context.Response.ContentLength ?? 0);
        }
    }
}
```

### Monitoring and Alerting

#### Application Insights Configuration
```csharp
// Program.cs
builder.Services.AddApplicationInsightsTelemetry();

// Configure custom telemetry
builder.Services.AddSingleton<ITelemetryInitializer, CustomTelemetryInitializer>();

// Custom telemetry initializer
public class CustomTelemetryInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        if (telemetry is RequestTelemetry requestTelemetry)
        {
            // Add custom properties
            requestTelemetry.Properties["ApplicationVersion"] = "1.0.0";
            requestTelemetry.Properties["Environment"] = Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT");
        }
    }
}
```

#### Health Checks Implementation
```csharp
using Microsoft.Extensions.Diagnostics.HealthChecks;

namespace VB6ToDotNetCore.Web.HealthChecks;

public class DatabaseHealthCheck : IHealthCheck
{
    private readonly ApplicationDbContext _context;

    public DatabaseHealthCheck(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Check database connectivity
            await _context.Database.CanConnectAsync(cancellationToken);

            // Check if critical tables exist
            var canQuery = await _context.Customers.AnyAsync(cancellationToken);

            return HealthCheckResult.Healthy("Database is healthy");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Database is unhealthy", ex);
        }
    }
}

// Register health checks
builder.Services.AddHealthChecks()
    .AddCheck<DatabaseHealthCheck>("database")
    .AddSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"))
    .AddRedis(builder.Configuration.GetConnectionString("Redis"));
```

### Documentation and Training

#### API Documentation
```csharp
// Program.cs
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "VB6 to .NET Core API",
        Version = "v1",
        Description = "Migrated VB6 application API",
        Contact = new OpenApiContact
        {
            Name = "Development Team",
            Email = "dev@company.com"
        }
    });

    // Include XML comments
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);

    // Add JWT Bearer token support
    options.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Description = "JWT Authorization header using the Bearer scheme",
        Name = "Authorization",
        In = ParameterLocation.Header,
        Type = SecuritySchemeType.ApiKey,
        Scheme = "Bearer"
    });

    options.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });
});
```

### Backup and Disaster Recovery

#### Database Backup Strategy
```sql
-- Full backup (weekly)
BACKUP DATABASE VB6ToDotNetCore
TO DISK = 'C:\Backups\VB6ToDotNetCore_Full.bak'
WITH FORMAT, INIT, NAME = 'VB6ToDotNetCore Full Backup';

-- Differential backup (daily)
BACKUP DATABASE VB6ToDotNetCore
TO DISK = 'C:\Backups\VB6ToDotNetCore_Diff.bak'
WITH DIFFERENTIAL, INIT, NAME = 'VB6ToDotNetCore Differential Backup';

-- Transaction log backup (hourly)
BACKUP LOG VB6ToDotNetCore
TO DISK = 'C:\Backups\VB6ToDotNetCore_Log.bak'
WITH INIT, NAME = 'VB6ToDotNetCore Log Backup';
```

#### Application Backup
```powershell
# Application backup script
param(
    [string]$BackupPath = "C:\Backups\Application",
    [string]$SourcePath = "C:\inetpub\wwwroot\VB6ToDotNetCore"
)

$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFile = Join-Path $BackupPath "VB6ToDotNetCore_App_$timestamp.zip"

# Create backup directory if it doesn't exist
if (!(Test-Path $BackupPath)) {
    New-Item -ItemType Directory -Path $BackupPath
}

# Create zip archive
Compress-Archive -Path $SourcePath -DestinationPath $backupFile

Write-Host "Application backup created: $backupFile"
```

### Migration Success Metrics

#### Key Performance Indicators
- **Application Performance**: Response time < 500ms for 95% of requests
- **Uptime**: 99.9% availability
- **Error Rate**: < 0.1% of total requests
- **User Satisfaction**: > 90% user acceptance
- **Code Quality**: 95%+ code coverage, 0 critical security issues

#### Success Criteria Checklist
- [ ] All VB6 forms converted to WPF/Windows Forms
- [ ] All ADO code migrated to Entity Framework Core
- [ ] All business logic preserved and tested
- [ ] All external integrations working
- [ ] Performance benchmarks met or exceeded
- [ ] Security audit passed
- [ ] User acceptance testing completed
- [ ] Production deployment successful
- [ ] Monitoring and alerting configured
- [ ] Documentation completed
- [ ] Team trained on new system

### Lessons Learned and Best Practices

#### Technical Lessons
1. **Start with Data Layer**: Database migration is the foundation
2. **Incremental Migration**: Break down large applications into manageable pieces
3. **Comprehensive Testing**: Invest heavily in automated testing
4. **Security First**: Address security concerns early in the process
5. **Performance Monitoring**: Implement monitoring from day one

#### Process Lessons
1. **Stakeholder Communication**: Keep all parties informed throughout
2. **Change Management**: Prepare users for the new system
3. **Training**: Provide adequate training for developers and users
4. **Rollback Plan**: Always have a rollback strategy
5. **Continuous Improvement**: Plan for ongoing enhancements

#### Best Practices for Future Migrations
1. **Modernize Architecture**: Use microservices where appropriate
2. **Cloud-Native Design**: Design for cloud deployment from the start
3. **DevOps Culture**: Implement CI/CD and infrastructure as code
4. **Security by Design**: Integrate security throughout the development process
5. **Observability**: Implement comprehensive monitoring and logging

This comprehensive guide provides a complete roadmap for migrating VB6 applications to .NET Core 10, ensuring a successful transition with minimal business disruption and maximum future-proofing.