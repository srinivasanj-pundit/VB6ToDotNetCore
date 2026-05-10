# Recommendations and Best Practices

## Migration Strategy Recommendations

This document provides comprehensive recommendations and best practices for the VB6 to .NET Core 10 migration project.

## Architecture Recommendations

### Clean Architecture Implementation

#### Recommended Layer Structure
```
VB6ToDotNetCore/
├── src/
│   ├── VB6ToDotNetCore.Domain/          # Domain entities, value objects, interfaces
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Interfaces/
│   │   ├── Services/
│   │   └── Events/
│   ├── VB6ToDotNetCore.Application/     # Application services, DTOs, commands/queries
│   │   ├── Services/
│   │   ├── DTOs/
│   │   ├── Commands/
│   │   ├── Queries/
│   │   └── Validators/
│   ├── VB6ToDotNetCore.Infrastructure/  # External concerns (EF, APIs, file system)
│   │   ├── Data/
│   │   ├── ExternalServices/
│   │   ├── FileStorage/
│   │   └── Messaging/
│   ├── VB6ToDotNetCore.Web/             # ASP.NET Core Web API
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Filters/
│   │   └── Extensions/
│   └── VB6ToDotNetCore.UI/              # WPF Desktop Application
│       ├── Views/
│       ├── ViewModels/
│       ├── Controls/
│       └── Services/
├── tests/
│   ├── VB6ToDotNetCore.Domain.Tests/
│   ├── VB6ToDotNetCore.Application.Tests/
│   ├── VB6ToDotNetCore.IntegrationTests/
│   └── VB6ToDotNetCore.UI.Tests/
└── build/
    ├── scripts/
    └── docker/
```

#### Dependency Direction
```
UI/Web → Application → Domain ← Infrastructure
```

### Design Patterns Recommendations

#### Repository Pattern Implementation
```csharp
// IRepository interface
public interface IRepository<T> where T : class, IEntity
{
    Task<T> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate);
    Task AddAsync(T entity);
    Task UpdateAsync(T entity);
    Task DeleteAsync(T entity);
    Task<int> CountAsync(Expression<Func<T, bool>> predicate = null);
}

// Generic repository implementation
public class Repository<T> : IRepository<T> where T : class, IEntity
{
    protected readonly DbContext _context;
    protected readonly DbSet<T> _dbSet;

    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }

    // Implementation methods...
}
```

#### Unit of Work Pattern
```csharp
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

public class UnitOfWork : IUnitOfWork
{
    private readonly ApplicationDbContext _context;
    private bool _disposed = false;

    public UnitOfWork(ApplicationDbContext context)
    {
        _context = context;
    }

    // Repository properties and transaction methods...
}
```

#### CQRS Pattern for Complex Operations
```csharp
// Commands
public class CreateOrderCommand : IRequest<int>
{
    public int CustomerId { get; set; }
    public List<OrderItemDto> Items { get; set; }
    // ... other properties
}

public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, int>
{
    private readonly IUnitOfWork _unitOfWork;
    private readonly IMediator _mediator;

    public CreateOrderCommandHandler(IUnitOfWork unitOfWork, IMediator mediator)
    {
        _unitOfWork = unitOfWork;
        _mediator = mediator;
    }

    public async Task<int> Handle(CreateOrderCommand request, CancellationToken cancellationToken)
    {
        // Business logic implementation...
    }
}

// Queries
public class GetCustomerOrdersQuery : IRequest<List<OrderDto>>
{
    public int CustomerId { get; set; }
    public DateTime? FromDate { get; set; }
    public DateTime? ToDate { get; set; }
}

public class GetCustomerOrdersQueryHandler : IRequestHandler<GetCustomerOrdersQuery, List<OrderDto>>
{
    private readonly IOrderRepository _orderRepository;

    public GetCustomerOrdersQueryHandler(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public async Task<List<OrderDto>> Handle(GetCustomerOrdersQuery request, CancellationToken cancellationToken)
    {
        // Query logic implementation...
    }
}
```

## Data Access Recommendations

### Entity Framework Core Configuration

#### DbContext Configuration
```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    public DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Apply configurations
        modelBuilder.ApplyConfiguration(new CustomerConfiguration());
        modelBuilder.ApplyConfiguration(new OrderConfiguration());
        modelBuilder.ApplyConfiguration(new ProductConfiguration());

        // Global query filters
        modelBuilder.Entity<Customer>().HasQueryFilter(c => !c.IsDeleted);
        modelBuilder.Entity<Order>().HasQueryFilter(o => !o.IsDeleted);

        base.OnModelCreating(modelBuilder);
    }

    public override Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        // Audit trail implementation
        foreach (var entry in ChangeTracker.Entries<AuditableEntity>())
        {
            switch (entry.State)
            {
                case EntityState.Added:
                    entry.Entity.CreatedDate = DateTime.UtcNow;
                    entry.Entity.CreatedBy = _currentUserService.GetUserId();
                    break;
                case EntityState.Modified:
                    entry.Entity.LastModifiedDate = DateTime.UtcNow;
                    entry.Entity.LastModifiedBy = _currentUserService.GetUserId();
                    break;
            }
        }

        return base.SaveChangesAsync(cancellationToken);
    }
}
```

#### Entity Configuration
```csharp
public class CustomerConfiguration : IEntityTypeConfiguration<Customer>
{
    public void Configure(EntityTypeBuilder<Customer> builder)
    {
        builder.ToTable("Customers");

        builder.HasKey(c => c.Id);

        builder.Property(c => c.CompanyName)
            .IsRequired()
            .HasMaxLength(100);

        builder.Property(c => c.Email)
            .HasMaxLength(255)
            .HasAnnotation("RegularExpression", @"^[^@\s]+@[^@\s]+\.[^@\s]+$");

        builder.HasIndex(c => c.Email)
            .IsUnique();

        builder.HasMany(c => c.Orders)
            .WithOne(o => o.Customer)
            .HasForeignKey(o => o.CustomerId)
            .OnDelete(DeleteBehavior.Restrict);
    }
}
```

### Database Migration Strategy

#### Migration Script Template
```csharp
[Migration(20231201000000)]
public partial class AddCustomerAuditFields : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<DateTime>(
            name: "CreatedDate",
            table: "Customers",
            nullable: false,
            defaultValue: new DateTime(2023, 12, 1, 0, 0, 0, 0, DateTimeKind.Utc));

        migrationBuilder.AddColumn<string>(
            name: "CreatedBy",
            table: "Customers",
            maxLength: 255,
            nullable: true);

        migrationBuilder.AddColumn<DateTime>(
            name: "LastModifiedDate",
            table: "Customers",
            nullable: true);

        migrationBuilder.AddColumn<string>(
            name: "LastModifiedBy",
            table: "Customers",
            maxLength: 255,
            nullable: true);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "CreatedDate",
            table: "Customers");

        migrationBuilder.DropColumn(
            name: "CreatedBy",
            table: "Customers");

        migrationBuilder.DropColumn(
            name: "LastModifiedDate",
            table: "Customers");

        migrationBuilder.DropColumn(
            name: "LastModifiedBy",
            table: "Customers");
    }
}
```

## API Development Recommendations

### REST API Design

#### Controller Structure
```csharp
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
[ApiVersion("1.0")]
[Produces("application/json")]
public class CustomersController : ControllerBase
{
    private readonly IMediator _mediator;
    private readonly ILogger<CustomersController> _logger;

    public CustomersController(IMediator mediator, ILogger<CustomersController> logger)
    {
        _mediator = mediator;
        _logger = logger;
    }

    [HttpGet]
    [ProducesResponseType(typeof(PagedResult<CustomerDto>), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<PagedResult<CustomerDto>>> GetCustomers(
        [FromQuery] GetCustomersQuery query)
    {
        try
        {
            var result = await _mediator.Send(query);
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving customers");
            return StatusCode(500, "Internal server error");
        }
    }

    [HttpGet("{id}")]
    [ProducesResponseType(typeof(CustomerDto), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<CustomerDto>> GetCustomer(int id)
    {
        var query = new GetCustomerByIdQuery { Id = id };
        var result = await _mediator.Send(query);

        if (result == null)
            return NotFound();

        return Ok(result);
    }

    [HttpPost]
    [ProducesResponseType(typeof(int), StatusCodes.Status201Created)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<ActionResult<int>> CreateCustomer(CreateCustomerCommand command)
    {
        var result = await _mediator.Send(command);
        return CreatedAtAction(nameof(GetCustomer), new { id = result }, result);
    }

    [HttpPut("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> UpdateCustomer(int id, UpdateCustomerCommand command)
    {
        if (id != command.Id)
            return BadRequest();

        await _mediator.Send(command);
        return NoContent();
    }

    [HttpDelete("{id}")]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<IActionResult> DeleteCustomer(int id)
    {
        await _mediator.Send(new DeleteCustomerCommand { Id = id });
        return NoContent();
    }
}
```

#### API Versioning Strategy
```csharp
// Startup.cs
services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = new UrlSegmentApiVersionReader();
});

// For backward compatibility
[ApiVersion("1.0")]
[ApiVersion("2.0")]
public class CustomersController : ControllerBase
{
    [HttpGet]
    [MapToApiVersion("1.0")]
    public async Task<ActionResult<CustomerDtoV1>> GetCustomerV1(int id)
    {
        // V1 implementation
    }

    [HttpGet]
    [MapToApiVersion("2.0")]
    public async Task<ActionResult<CustomerDtoV2>> GetCustomerV2(int id)
    {
        // V2 implementation with additional fields
    }
}
```

### Error Handling and Validation

#### Global Exception Handler
```csharp
public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "An unhandled exception occurred");

        var problemDetails = new ProblemDetails
        {
            Status = GetStatusCode(exception),
            Title = GetTitle(exception),
            Detail = GetDetail(exception),
            Instance = httpContext.Request.Path
        };

        if (exception is ValidationException validationException)
        {
            problemDetails.Extensions["errors"] = validationException.Errors;
        }

        httpContext.Response.StatusCode = problemDetails.Status.Value;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }

    private static int GetStatusCode(Exception exception) =>
        exception switch
        {
            ValidationException => StatusCodes.Status400BadRequest,
            NotFoundException => StatusCodes.Status404NotFound,
            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,
            ForbiddenException => StatusCodes.Status403Forbidden,
            _ => StatusCodes.Status500InternalServerError
        };

    private static string GetTitle(Exception exception) =>
        exception switch
        {
            ValidationException => "Validation Error",
            NotFoundException => "Resource Not Found",
            UnauthorizedAccessException => "Unauthorized",
            ForbiddenException => "Forbidden",
            _ => "Internal Server Error"
        };

    private static string GetDetail(Exception exception) =>
        exception switch
        {
            ValidationException => "One or more validation errors occurred",
            NotFoundException => "The requested resource was not found",
            UnauthorizedAccessException => "Authentication is required",
            ForbiddenException => "Access to this resource is forbidden",
            _ => "An unexpected error occurred"
        };
}
```

#### FluentValidation Implementation
```csharp
public class CreateCustomerCommandValidator : AbstractValidator<CreateCustomerCommand>
{
    public CreateCustomerCommandValidator()
    {
        RuleFor(x => x.CompanyName)
            .NotEmpty().WithMessage("Company name is required")
            .MaximumLength(100).WithMessage("Company name must not exceed 100 characters");

        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Email must be a valid email address")
            .MaximumLength(255).WithMessage("Email must not exceed 255 characters");

        RuleFor(x => x.Phone)
            .MaximumLength(20).WithMessage("Phone number must not exceed 20 characters")
            .Matches(@"^\+?[\d\s\-\(\)]+$").WithMessage("Phone number contains invalid characters")
            .When(x => !string.IsNullOrEmpty(x.Phone));

        RuleFor(x => x.Address)
            .SetValidator(new AddressValidator())
            .When(x => x.Address != null);
    }
}

public class AddressValidator : AbstractValidator<AddressDto>
{
    public AddressValidator()
    {
        RuleFor(x => x.Street)
            .NotEmpty().WithMessage("Street address is required")
            .MaximumLength(200).WithMessage("Street address must not exceed 200 characters");

        RuleFor(x => x.City)
            .NotEmpty().WithMessage("City is required")
            .MaximumLength(100).WithMessage("City must not exceed 100 characters");

        RuleFor(x => x.PostalCode)
            .NotEmpty().WithMessage("Postal code is required")
            .Matches(@"^\d{5}(-\d{4})?$").WithMessage("Postal code must be in valid format")
            .When(x => x.Country == "US");

        RuleFor(x => x.Country)
            .NotEmpty().WithMessage("Country is required")
            .Length(2, 2).WithMessage("Country code must be 2 characters")
            .Must(BeValidCountryCode).WithMessage("Invalid country code");
    }

    private bool BeValidCountryCode(string countryCode)
    {
        // Implement country code validation logic
        return !string.IsNullOrEmpty(countryCode) && countryCode.Length == 2;
    }
}
```

## UI Development Recommendations

### WPF MVVM Implementation

#### ViewModel Base Class
```csharp
public abstract class ViewModelBase : INotifyPropertyChanged, IDisposable
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

    protected virtual void Dispose(bool disposing)
    {
        if (disposing)
        {
            // Dispose managed resources
        }
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

#### Customer ViewModel Example
```csharp
public class CustomerViewModel : ViewModelBase
{
    private readonly ICustomerService _customerService;
    private readonly IDialogService _dialogService;
    private ObservableCollection<CustomerDto> _customers;
    private CustomerDto _selectedCustomer;
    private string _searchText;
    private bool _isLoading;

    public CustomerViewModel(ICustomerService customerService, IDialogService dialogService)
    {
        _customerService = customerService;
        _dialogService = dialogService;

        Customers = new ObservableCollection<CustomerDto>();
        LoadCustomersCommand = new AsyncRelayCommand(LoadCustomersAsync);
        SaveCustomerCommand = new AsyncRelayCommand(SaveCustomerAsync, CanSaveCustomer);
        DeleteCustomerCommand = new AsyncRelayCommand(DeleteCustomerAsync, CanDeleteCustomer);
        NewCustomerCommand = new RelayCommand(CreateNewCustomer);
    }

    public ObservableCollection<CustomerDto> Customers
    {
        get => _customers;
        set => SetProperty(ref _customers, value);
    }

    public CustomerDto SelectedCustomer
    {
        get => _selectedCustomer;
        set
        {
            SetProperty(ref _selectedCustomer, value);
            SaveCustomerCommand.RaiseCanExecuteChanged();
            DeleteCustomerCommand.RaiseCanExecuteChanged();
        }
    }

    public string SearchText
    {
        get => _searchText;
        set
        {
            SetProperty(ref _searchText, value);
            _ = LoadCustomersAsync();
        }
    }

    public bool IsLoading
    {
        get => _isLoading;
        set => SetProperty(ref _isLoading, value);
    }

    public AsyncRelayCommand LoadCustomersCommand { get; }
    public AsyncRelayCommand SaveCustomerCommand { get; }
    public AsyncRelayCommand DeleteCustomerCommand { get; }
    public RelayCommand NewCustomerCommand { get; }

    private async Task LoadCustomersAsync()
    {
        try
        {
            IsLoading = true;
            var customers = await _customerService.GetCustomersAsync(SearchText);
            Customers.Clear();
            foreach (var customer in customers)
            {
                Customers.Add(customer);
            }
        }
        catch (Exception ex)
        {
            await _dialogService.ShowErrorAsync("Failed to load customers", ex.Message);
        }
        finally
        {
            IsLoading = false;
        }
    }

    private async Task SaveCustomerAsync()
    {
        if (SelectedCustomer == null) return;

        try
        {
            IsLoading = true;
            await _customerService.SaveCustomerAsync(SelectedCustomer);
            await _dialogService.ShowMessageAsync("Customer saved successfully");
        }
        catch (ValidationException ex)
        {
            await _dialogService.ShowValidationErrorsAsync(ex.Errors);
        }
        catch (Exception ex)
        {
            await _dialogService.ShowErrorAsync("Failed to save customer", ex.Message);
        }
        finally
        {
            IsLoading = false;
        }
    }

    private bool CanSaveCustomer() => SelectedCustomer != null && !SelectedCustomer.HasErrors;

    private async Task DeleteCustomerAsync()
    {
        if (SelectedCustomer == null) return;

        if (await _dialogService.ShowConfirmationAsync(
            "Delete Customer",
            $"Are you sure you want to delete {SelectedCustomer.CompanyName}?"))
        {
            try
            {
                IsLoading = true;
                await _customerService.DeleteCustomerAsync(SelectedCustomer.Id);
                Customers.Remove(SelectedCustomer);
                await _dialogService.ShowMessageAsync("Customer deleted successfully");
            }
            catch (Exception ex)
            {
                await _dialogService.ShowErrorAsync("Failed to delete customer", ex.Message);
            }
            finally
            {
                IsLoading = false;
            }
        }
    }

    private bool CanDeleteCustomer() => SelectedCustomer != null;

    private void CreateNewCustomer()
    {
        var newCustomer = new CustomerDto();
        Customers.Add(newCustomer);
        SelectedCustomer = newCustomer;
    }
}
```

### UI Testing Recommendations

#### UI Test Structure
```csharp
[TestClass]
public class CustomerViewModelTests
{
    private Mock<ICustomerService> _customerServiceMock;
    private Mock<IDialogService> _dialogServiceMock;
    private CustomerViewModel _viewModel;

    [TestInitialize]
    public void TestInitialize()
    {
        _customerServiceMock = new Mock<ICustomerService>();
        _dialogServiceMock = new Mock<IDialogService>();
        _viewModel = new CustomerViewModel(_customerServiceMock.Object, _dialogServiceMock.Object);
    }

    [TestMethod]
    public async Task LoadCustomersAsync_ShouldPopulateCustomersCollection()
    {
        // Arrange
        var expectedCustomers = new List<CustomerDto>
        {
            new CustomerDto { Id = 1, CompanyName = "Test Company 1" },
            new CustomerDto { Id = 2, CompanyName = "Test Company 2" }
        };

        _customerServiceMock
            .Setup(x => x.GetCustomersAsync(It.IsAny<string>()))
            .ReturnsAsync(expectedCustomers);

        // Act
        await _viewModel.LoadCustomersCommand.ExecuteAsync(null);

        // Assert
        Assert.AreEqual(2, _viewModel.Customers.Count);
        Assert.AreEqual("Test Company 1", _viewModel.Customers[0].CompanyName);
        Assert.AreEqual("Test Company 2", _viewModel.Customers[1].CompanyName);
    }

    [TestMethod]
    public async Task SaveCustomerAsync_ValidCustomer_ShouldSaveSuccessfully()
    {
        // Arrange
        var customer = new CustomerDto
        {
            Id = 1,
            CompanyName = "Valid Company",
            Email = "test@company.com"
        };

        _viewModel.Customers.Add(customer);
        _viewModel.SelectedCustomer = customer;

        _customerServiceMock
            .Setup(x => x.SaveCustomerAsync(customer))
            .Returns(Task.CompletedTask);

        _dialogServiceMock
            .Setup(x => x.ShowMessageAsync(It.IsAny<string>()))
            .Returns(Task.CompletedTask);

        // Act
        await _viewModel.SaveCustomerCommand.ExecuteAsync(null);

        // Assert
        _customerServiceMock.Verify(x => x.SaveCustomerAsync(customer), Times.Once);
        _dialogServiceMock.Verify(x => x.ShowMessageAsync("Customer saved successfully"), Times.Once);
    }

    [TestMethod]
    public void CanSaveCustomer_SelectedCustomerWithoutErrors_ReturnsTrue()
    {
        // Arrange
        var customer = new CustomerDto { Id = 1, CompanyName = "Test Company" };
        _viewModel.Customers.Add(customer);
        _viewModel.SelectedCustomer = customer;

        // Act & Assert
        Assert.IsTrue(_viewModel.SaveCustomerCommand.CanExecute(null));
    }

    [TestMethod]
    public void CanSaveCustomer_NoSelectedCustomer_ReturnsFalse()
    {
        // Arrange
        _viewModel.SelectedCustomer = null;

        // Act & Assert
        Assert.IsFalse(_viewModel.SaveCustomerCommand.CanExecute(null));
    }
}
```

## Security Recommendations

### Authentication and Authorization

#### JWT Token Implementation
```csharp
public class JwtTokenService : ITokenService
{
    private readonly JwtSettings _jwtSettings;

    public JwtTokenService(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }

    public string GenerateToken(User user)
    {
        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim(ClaimTypes.Name, user.UserName)
        };

        // Add roles
        foreach (var role in user.Roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role.Name));
        }

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.SecretKey));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpiryMinutes),
            signingCredentials: creds);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public ClaimsPrincipal ValidateToken(string token)
    {
        var tokenHandler = new JwtSecurityTokenHandler();
        var key = Encoding.UTF8.GetBytes(_jwtSettings.SecretKey);

        var validationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(key),
            ValidateIssuer = true,
            ValidIssuer = _jwtSettings.Issuer,
            ValidateAudience = true,
            ValidAudience = _jwtSettings.Audience,
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero
        };

        try
        {
            var principal = tokenHandler.ValidateToken(token, validationParameters, out _);
            return principal;
        }
        catch
        {
            return null;
        }
    }
}
```

#### API Security Middleware
```csharp
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
        context.Response.Headers.Add("Content-Security-Policy", "default-src 'self'");

        // CORS headers (if needed)
        if (context.Request.Method == "OPTIONS")
        {
            context.Response.StatusCode = 200;
            return;
        }

        await _next(context);
    }
}
```

### Data Protection

#### Encryption Service
```csharp
public class DataProtectionService : IDataProtectionService
{
    private readonly IDataProtector _protector;

    public DataProtectionService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("VB6ToDotNetCore.DataProtection");
    }

    public string Protect(string plainText)
    {
        if (string.IsNullOrEmpty(plainText))
            return plainText;

        return _protector.Protect(plainText);
    }

    public string Unprotect(string protectedText)
    {
        if (string.IsNullOrEmpty(protectedText))
            return protectedText;

        try
        {
            return _protector.Unprotect(protectedText);
        }
        catch (CryptographicException)
        {
            // Log the error and return original text or throw
            throw new InvalidOperationException("Could not unprotect the data. The data may be corrupted or the key may have changed.");
        }
    }
}
```

## Performance Optimization Recommendations

### Caching Strategies

#### In-Memory Caching
```csharp
public class CachedCustomerService : ICustomerService
{
    private readonly ICustomerService _innerService;
    private readonly IMemoryCache _cache;
    private readonly ILogger<CachedCustomerService> _logger;

    public CachedCustomerService(
        ICustomerService innerService,
        IMemoryCache cache,
        ILogger<CachedCustomerService> logger)
    {
        _innerService = innerService;
        _cache = cache;
        _logger = logger;
    }

    public async Task<CustomerDto> GetCustomerByIdAsync(int id)
    {
        var cacheKey = $"Customer_{id}";

        if (!_cache.TryGetValue(cacheKey, out CustomerDto customer))
        {
            _logger.LogInformation("Customer {Id} not found in cache, fetching from database", id);
            customer = await _innerService.GetCustomerByIdAsync(id);

            if (customer != null)
            {
                var cacheOptions = new MemoryCacheEntryOptions()
                    .SetSlidingExpiration(TimeSpan.FromMinutes(30))
                    .SetAbsoluteExpiration(TimeSpan.FromHours(2));

                _cache.Set(cacheKey, customer, cacheOptions);
            }
        }
        else
        {
            _logger.LogInformation("Customer {Id} retrieved from cache", id);
        }

        return customer;
    }

    public async Task UpdateCustomerAsync(CustomerDto customer)
    {
        await _innerService.UpdateCustomerAsync(customer);

        // Invalidate cache
        var cacheKey = $"Customer_{customer.Id}";
        _cache.Remove(cacheKey);

        _logger.LogInformation("Cache invalidated for customer {Id}", customer.Id);
    }
}
```

#### Distributed Caching with Redis
```csharp
public class RedisCacheService : ICacheService
{
    private readonly IDistributedCache _cache;
    private readonly ILogger<RedisCacheService> _logger;

    public RedisCacheService(IDistributedCache cache, ILogger<RedisCacheService> logger)
    {
        _cache = cache;
        _logger = logger;
    }

    public async Task<T> GetAsync<T>(string key)
    {
        var cachedData = await _cache.GetStringAsync(key);
        if (cachedData == null)
            return default;

        try
        {
            return JsonSerializer.Deserialize<T>(cachedData);
        }
        catch (JsonException ex)
        {
            _logger.LogWarning(ex, "Failed to deserialize cached data for key {Key}", key);
            await RemoveAsync(key);
            return default;
        }
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan? expiration = null)
    {
        var options = new DistributedCacheEntryOptions();
        if (expiration.HasValue)
        {
            options.SetSlidingExpiration(expiration.Value);
        }
        else
        {
            options.SetSlidingExpiration(TimeSpan.FromMinutes(30));
        }

        var serializedData = JsonSerializer.Serialize(value);
        await _cache.SetStringAsync(key, serializedData, options);
    }

    public async Task RemoveAsync(string key)
    {
        await _cache.RemoveAsync(key);
    }

    public async Task<bool> ExistsAsync(string key)
    {
        var value = await _cache.GetStringAsync(key);
        return value != null;
    }
}
```

### Database Optimization

#### Query Optimization
```csharp
public class OptimizedOrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;

    public OptimizedOrderRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<List<OrderSummaryDto>> GetOrderSummariesAsync(DateTime fromDate, DateTime toDate)
    {
        return await _context.Orders
            .Where(o => o.OrderDate >= fromDate && o.OrderDate <= toDate)
            .Select(o => new OrderSummaryDto
            {
                OrderId = o.Id,
                OrderDate = o.OrderDate,
                CustomerName = o.Customer.CompanyName,
                TotalAmount = o.OrderItems.Sum(oi => oi.Quantity * oi.UnitPrice),
                Status = o.Status
            })
            .AsNoTracking()
            .ToListAsync();
    }

    public async Task<OrderDetailsDto> GetOrderDetailsAsync(int orderId)
    {
        return await _context.Orders
            .Where(o => o.Id == orderId)
            .Select(o => new OrderDetailsDto
            {
                OrderId = o.Id,
                OrderDate = o.OrderDate,
                Customer = new CustomerDto
                {
                    Id = o.Customer.Id,
                    CompanyName = o.Customer.CompanyName,
                    ContactName = o.Customer.ContactName,
                    Email = o.Customer.Email
                },
                Items = o.OrderItems.Select(oi => new OrderItemDto
                {
                    ProductId = oi.Product.Id,
                    ProductName = oi.Product.ProductName,
                    Quantity = oi.Quantity,
                    UnitPrice = oi.UnitPrice,
                    Total = oi.Quantity * oi.UnitPrice
                }).ToList(),
                TotalAmount = o.OrderItems.Sum(oi => oi.Quantity * oi.UnitPrice)
            })
            .AsNoTracking()
            .FirstOrDefaultAsync();
    }
}
```

#### Connection Resiliency
```csharp
// Startup.cs or Program.cs
services.AddDbContext<ApplicationDbContext>(options =>
{
    options.UseSqlServer(connectionString,
        sqlServerOptionsAction: sqlOptions =>
        {
            sqlOptions.EnableRetryOnFailure(
                maxRetryCount: 5,
                maxRetryDelay: TimeSpan.FromSeconds(30),
                errorNumbersToAdd: new List<int> { 4060, 40197, 40501, 40613, 49918, 49919, 49920, 11001 });
        });
});
```

## Monitoring and Logging Recommendations

### Structured Logging

#### Serilog Configuration
```csharp
public static class LoggingConfiguration
{
    public static void ConfigureLogging(WebApplicationBuilder builder)
    {
        builder.Host.UseSerilog((context, services, configuration) =>
        {
            configuration
                .ReadFrom.Configuration(context.Configuration)
                .ReadFrom.Services(services)
                .Enrich.FromLogContext()
                .Enrich.WithMachineName()
                .Enrich.WithThreadId()
                .Enrich.WithCorrelationId()
                .WriteTo.Console(
                    outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}")
                .WriteTo.File(
                    path: "logs/vb6-app-.log",
                    rollingInterval: RollingInterval.Day,
                    retainedFileCountLimit: 7,
                    outputTemplate: "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj} {Properties:j}{NewLine}{Exception}")
                .WriteTo.ApplicationInsights(
                    services.GetRequiredService<TelemetryConfiguration>(),
                    TelemetryConverter.Traces);
        });
    }
}
```

#### Logging Best Practices
```csharp
public class CustomerService : ICustomerService
{
    private readonly ILogger<CustomerService> _logger;
    private readonly IUnitOfWork _unitOfWork;

    public CustomerService(ILogger<CustomerService> _logger, IUnitOfWork unitOfWork)
    {
        _logger = logger;
        _unitOfWork = unitOfWork;
    }

    public async Task<CustomerDto> GetCustomerByIdAsync(int customerId)
    {
        using (_logger.BeginScope(new Dictionary<string, object>
        {
            ["CustomerId"] = customerId,
            ["Operation"] = "GetCustomerById"
        }))
        {
            _logger.LogInformation("Retrieving customer with ID {CustomerId}", customerId);

            try
            {
                var customer = await _unitOfWork.Customers.GetByIdAsync(customerId);

                if (customer == null)
                {
                    _logger.LogWarning("Customer with ID {CustomerId} not found", customerId);
                    return null;
                }

                _logger.LogInformation("Successfully retrieved customer {CustomerName}", customer.CompanyName);
                return customer;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to retrieve customer with ID {CustomerId}", customerId);
                throw;
            }
        }
    }

    public async Task CreateCustomerAsync(CreateCustomerCommand command)
    {
        using (_logger.BeginScope(new Dictionary<string, object>
        {
            ["Operation"] = "CreateCustomer",
            ["CompanyName"] = command.CompanyName,
            ["Email"] = command.Email
        }))
        {
            _logger.LogInformation("Creating new customer {CompanyName}", command.CompanyName);

            try
            {
                var customer = new Customer
                {
                    CompanyName = command.CompanyName,
                    ContactName = command.ContactName,
                    Email = command.Email,
                    Phone = command.Phone,
                    Address = command.Address
                };

                await _unitOfWork.Customers.AddAsync(customer);
                await _unitOfWork.SaveChangesAsync();

                _logger.LogInformation("Successfully created customer {CompanyName} with ID {CustomerId}",
                    customer.CompanyName, customer.Id);

                return customer.Id;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to create customer {CompanyName}", command.CompanyName);
                throw;
            }
        }
    }
}
```

### Health Checks and Metrics

#### Application Health Checks
```csharp
public static class HealthCheckConfiguration
{
    public static void ConfigureHealthChecks(WebApplicationBuilder builder)
    {
        builder.Services.AddHealthChecks()
            .AddSqlServer(
                connectionString: builder.Configuration.GetConnectionString("DefaultConnection"),
                name: "Database",
                tags: new[] { "db", "sql", "sqlserver" })
            .AddRedis(
                redisConnectionString: builder.Configuration.GetConnectionString("Redis"),
                name: "Redis Cache",
                tags: new[] { "cache", "redis" })
            .AddUrlGroup(
                uri: new Uri("https://api.paymentgateway.com/health"),
                name: "Payment Gateway",
                tags: new[] { "external", "payment" })
            .AddApplicationInsightsPublisher();

        builder.Services.AddHealthChecksUI(setup =>
        {
            setup.AddHealthCheckEndpoint("API Health", "/health");
            setup.AddWebhookNotification("Teams", "https://outlook.office.com/webhook/...",
                payload: "{ \"text\": \"Health Check: {LIVENESS}\" }");
        });
    }
}
```

#### Application Metrics
```csharp
public class ApplicationMetricsService
{
    private readonly Meter _meter;
    private readonly Counter<long> _ordersCreated;
    private readonly Histogram<double> _orderProcessingTime;
    private readonly UpDownCounter<long> _activeUsers;

    public ApplicationMetricsService()
    {
        _meter = new Meter("VB6ToDotNetCore", "1.0.0");

        _ordersCreated = _meter.CreateCounter<long>(
            "orders_created_total",
            description: "Total number of orders created");

        _orderProcessingTime = _meter.CreateHistogram<double>(
            "order_processing_duration_seconds",
            description: "Time taken to process orders");

        _activeUsers = _meter.CreateUpDownCounter<long>(
            "active_users",
            description: "Number of currently active users");
    }

    public void OrderCreated(string customerType, string orderType)
    {
        _ordersCreated.Add(1, new KeyValuePair<string, object>("customer_type", customerType),
                          new KeyValuePair<string, object>("order_type", orderType));
    }

    public void RecordOrderProcessingTime(double durationSeconds, string orderType)
    {
        _orderProcessingTime.Record(durationSeconds,
            new KeyValuePair<string, object>("order_type", orderType));
    }

    public void UserLoggedIn()
    {
        _activeUsers.Add(1);
    }

    public void UserLoggedOut()
    {
        _activeUsers.Add(-1);
    }
}
```

## Testing Recommendations

### Unit Testing Best Practices

#### Test Structure
```csharp
[TestClass]
public class CustomerServiceTests : TestBase
{
    private Mock<IUnitOfWork> _unitOfWorkMock;
    private Mock<ICustomerRepository> _customerRepositoryMock;
    private Mock<ILogger<CustomerService>> _loggerMock;
    private CustomerService _service;

    [TestInitialize]
    public void TestInitialize()
    {
        _unitOfWorkMock = new Mock<IUnitOfWork>();
        _customerRepositoryMock = new Mock<ICustomerRepository>();
        _loggerMock = new Mock<ILogger<CustomerService>>();

        _unitOfWorkMock.Setup(u => u.Customers).Returns(_customerRepositoryMock.Object);

        _service = new CustomerService(
            _unitOfWorkMock.Object,
            _loggerMock.Object);
    }

    [TestMethod]
    public async Task GetCustomerByIdAsync_ExistingCustomer_ReturnsCustomer()
    {
        // Arrange
        var customerId = 1;
        var expectedCustomer = new CustomerDto
        {
            Id = customerId,
            CompanyName = "Test Company",
            Email = "test@company.com"
        };

        _customerRepositoryMock
            .Setup(r => r.GetByIdAsync(customerId))
            .ReturnsAsync(expectedCustomer);

        // Act
        var result = await _service.GetCustomerByIdAsync(customerId);

        // Assert
        Assert.IsNotNull(result);
        Assert.AreEqual(expectedCustomer.Id, result.Id);
        Assert.AreEqual(expectedCustomer.CompanyName, result.CompanyName);
        Assert.AreEqual(expectedCustomer.Email, result.Email);

        _customerRepositoryMock.Verify(r => r.GetByIdAsync(customerId), Times.Once);
    }

    [TestMethod]
    public async Task GetCustomerByIdAsync_NonExistingCustomer_ReturnsNull()
    {
        // Arrange
        var customerId = 999;
        _customerRepositoryMock
            .Setup(r => r.GetByIdAsync(customerId))
            .ReturnsAsync((CustomerDto)null);

        // Act
        var result = await _service.GetCustomerByIdAsync(customerId);

        // Assert
        Assert.IsNull(result);
        _customerRepositoryMock.Verify(r => r.GetByIdAsync(customerId), Times.Once);
    }

    [TestMethod]
    public async Task CreateCustomerAsync_ValidCustomer_CreatesAndReturnsId()
    {
        // Arrange
        var command = new CreateCustomerCommand
        {
            CompanyName = "New Company",
            Email = "new@company.com",
            Phone = "123-456-7890"
        };

        var createdCustomer = new Customer
        {
            Id = 123,
            CompanyName = command.CompanyName,
            Email = command.Email,
            Phone = command.Phone
        };

        _customerRepositoryMock
            .Setup(r => r.AddAsync(It.IsAny<Customer>()))
            .Callback<Customer>(c => c.Id = 123)
            .Returns(Task.CompletedTask);

        _unitOfWorkMock
            .Setup(u => u.SaveChangesAsync(It.IsAny<CancellationToken>()))
            .ReturnsAsync(1);

        // Act
        var result = await _service.CreateCustomerAsync(command);

        // Assert
        Assert.AreEqual(123, result);

        _customerRepositoryMock.Verify(r => r.AddAsync(It.Is<Customer>(c =>
            c.CompanyName == command.CompanyName &&
            c.Email == command.Email &&
            c.Phone == command.Phone)), Times.Once);

        _unitOfWorkMock.Verify(u => u.SaveChangesAsync(It.IsAny<CancellationToken>()), Times.Once);
    }

    [TestMethod]
    [ExpectedException(typeof(ValidationException))]
    public async Task CreateCustomerAsync_InvalidEmail_ThrowsValidationException()
    {
        // Arrange
        var command = new CreateCustomerCommand
        {
            CompanyName = "Test Company",
            Email = "invalid-email", // Invalid email format
            Phone = "123-456-7890"
        };

        // Act
        await _service.CreateCustomerAsync(command);

        // Assert - Exception expected
    }
}
```

#### Integration Testing
```csharp
[TestClass]
public class CustomerApiIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public CustomerApiIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory.WithWebHostBuilder(builder =>
        {
            builder.ConfigureServices(services =>
            {
                // Replace real database with test database
                var descriptor = services.SingleOrDefault(
                    d => d.ServiceType == typeof(DbContextOptions<ApplicationDbContext>));

                if (descriptor != null)
                {
                    services.Remove(descriptor);
                }

                services.AddDbContext<ApplicationDbContext>(options =>
                {
                    options.UseInMemoryDatabase("TestDatabase");
                });

                // Seed test data
                var sp = services.BuildServiceProvider();
                using var scope = sp.CreateScope();
                var db = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
                db.Database.EnsureCreated();
                SeedTestData(db);
            });
        });

        _client = _factory.CreateClient();
    }

    [TestMethod]
    public async Task GetCustomers_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/api/v1/customers");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.AreEqual("application/json; charset=utf-8", response.Content.Headers.ContentType.ToString());
    }

    [TestMethod]
    public async Task CreateCustomer_ValidData_ReturnsCreatedStatus()
    {
        // Arrange
        var customer = new
        {
            companyName = "Integration Test Company",
            email = "integration@test.com",
            phone = "555-123-4567"
        };

        var content = new StringContent(JsonSerializer.Serialize(customer), Encoding.UTF8, "application/json");

        // Act
        var response = await _client.PostAsync("/api/v1/customers", content);

        // Assert
        Assert.AreEqual(HttpStatusCode.Created, response.StatusCode);

        var responseContent = await response.Content.ReadAsStringAsync();
        var createdCustomer = JsonSerializer.Deserialize<CustomerDto>(responseContent, new JsonSerializerOptions
        {
            PropertyNameCaseInsensitive = true
        });

        Assert.IsNotNull(createdCustomer);
        Assert.AreEqual(customer.companyName, createdCustomer.CompanyName);
        Assert.AreEqual(customer.email, createdCustomer.Email);
    }

    private static void SeedTestData(ApplicationDbContext context)
    {
        context.Customers.AddRange(
            new Customer { Id = 1, CompanyName = "Test Company 1", Email = "test1@company.com" },
            new Customer { Id = 2, CompanyName = "Test Company 2", Email = "test2@company.com" }
        );
        context.SaveChanges();
    }
}
```

## Deployment Recommendations

### Containerization Strategy

#### Multi-Stage Dockerfile
```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0-alpine AS build
WORKDIR /src

# Copy csproj files and restore dependencies
COPY ["src/VB6ToDotNetCore.Core/VB6ToDotNetCore.Core.csproj", "src/VB6ToDotNetCore.Core/"]
COPY ["src/VB6ToDotNetCore.Data/VB6ToDotNetCore.Data.csproj", "src/VB6ToDotNetCore.Data/"]
COPY ["src/VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj", "src/VB6ToDotNetCore.Web/"]
RUN dotnet restore "src/VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj"

# Copy everything else and build
COPY . .
WORKDIR "/src/src/VB6ToDotNetCore.Web"
RUN dotnet build "VB6ToDotNetCore.Web.csproj" -c Release -o /app/build

# Publish stage
FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0-alpine AS final
WORKDIR /app

# Install cultures for localization
RUN apk add --no-cache icu-libs
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -u 1001 -S appuser -G appgroup

# Copy published app
COPY --from=publish /app/publish .

# Set ownership
RUN chown -R appuser:appgroup /app
USER appuser

EXPOSE 80
EXPOSE 443

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/health || exit 1

ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Web.dll"]
```

#### Docker Compose for Development
```yaml
version: '3.8'

services:
  api:
    build:
      context: .
      dockerfile: src/VB6ToDotNetCore.Web/Dockerfile
    ports:
      - "5000:80"
      - "5001:443"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ConnectionStrings__DefaultConnection=Server=db;Database=VB6ToDotNetCore;User=sa;Password=YourStrong!Passw0rd;
    depends_on:
      - db
    volumes:
      - ./src/VB6ToDotNetCore.Web/https:/https:ro
    networks:
      - vb6-network

  db:
    image: mcr.microsoft.com/mssql/server:2022-latest
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong!Passw0rd
      - MSSQL_PID=Express
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql
      - ./database/init:/docker-entrypoint-initdb.d
    networks:
      - vb6-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - vb6-network

volumes:
  sqlserver_data:
  redis_data:

networks:
  vb6-network:
    driver: bridge
```

### Kubernetes Deployment

#### Deployment Manifest
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-app-api
  labels:
    app: vb6-app-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: vb6-app-api
  template:
    metadata:
      labels:
        app: vb6-app-api
    spec:
      containers:
      - name: api
        image: vb6app.azurecr.io/api:latest
        ports:
        - containerPort: 80
          name: http
        - containerPort: 443
          name: https
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ASPNETCORE_URLS
          value: "http://+:80;https://+:443"
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connectionstring
        - name: Redis__ConnectionString
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: connectionstring
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health/live
            port: http
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /health/ready
            port: http
            scheme: HTTP
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /health/startup
            port: http
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30
      imagePullSecrets:
      - name: acr-secret
```

#### Service Manifest
```yaml
apiVersion: v1
kind: Service
metadata:
  name: vb6-app-api-service
  labels:
    app: vb6-app-api
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    app: vb6-app-api
```

#### Ingress Configuration
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vb6-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.vb6-app.company.com
    secretName: vb6-app-tls
  rules:
  - host: api.vb6-app.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vb6-app-api-service
            port:
              number: 80
```

This comprehensive set of recommendations provides best practices and implementation guidance for successfully migrating the VB6 application to .NET Core 10 while ensuring maintainability, performance, security, and scalability.