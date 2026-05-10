# Testing

This document outlines the comprehensive testing strategy for the VB6 to .NET Core migration project. It includes unit tests, integration tests, and functional tests designed to ensure code quality and business logic correctness.

## Testing Framework Setup

### Tools and Libraries
- **Unit Testing**: xUnit.net
- **Mocking**: Moq
- **Test Coverage**: Coverlet
- **UI Testing**: Selenium WebDriver (for Blazor) or WinAppDriver (for WinForms/WPF)
- **API Testing**: RestSharp or HttpClient
- **Database Testing**: TestContainers or LocalDB

### Project Structure
```
Tests/
├── UnitTests/
│   ├── BusinessLogicTests/
│   ├── DataAccessTests/
│   └── UtilityTests/
├── IntegrationTests/
│   ├── ApiIntegrationTests/
│   ├── DatabaseIntegrationTests/
│   └── UiIntegrationTests/
├── FunctionalTests/
│   ├── EndToEndTests/
│   └── UserAcceptanceTests/
└── TestData/
    ├── SampleData.json
    └── TestDatabase.db
```

## Unit Testing Strategy

### Business Logic Tests

#### CustomerService Tests
```csharp
public class CustomerServiceTests
{
    private readonly Mock<ICustomerRepository> _customerRepositoryMock;
    private readonly CustomerService _customerService;

    public CustomerServiceTests()
    {
        _customerRepositoryMock = new Mock<ICustomerRepository>();
        _customerService = new CustomerService(_customerRepositoryMock.Object);
    }

    [Fact]
    public async Task CreateCustomer_ValidData_ReturnsCustomerId()
    {
        // Arrange
        var customer = new Customer
        {
            Name = "John Doe",
            Email = "john.doe@example.com",
            Phone = "123-456-7890"
        };

        _customerRepositoryMock
            .Setup(repo => repo.AddAsync(It.IsAny<Customer>()))
            .ReturnsAsync(1);

        // Act
        var result = await _customerService.CreateCustomerAsync(customer);

        // Assert
        Assert.Equal(1, result);
        _customerRepositoryMock.Verify(repo => repo.AddAsync(customer), Times.Once);
    }

    [Fact]
    public async Task CreateCustomer_InvalidEmail_ThrowsValidationException()
    {
        // Arrange
        var customer = new Customer
        {
            Name = "John Doe",
            Email = "invalid-email",
            Phone = "123-456-7890"
        };

        // Act & Assert
        await Assert.ThrowsAsync<ValidationException>(() =>
            _customerService.CreateCustomerAsync(customer));
    }

    [Theory]
    [InlineData("", "Valid email")]
    [InlineData("Valid Name", "")]
    [InlineData(null, "valid@email.com")]
    public async Task CreateCustomer_MissingRequiredFields_ThrowsValidationException(
        string name, string email)
    {
        // Arrange
        var customer = new Customer { Name = name, Email = email };

        // Act & Assert
        await Assert.ThrowsAsync<ValidationException>(() =>
            _customerService.CreateCustomerAsync(customer));
    }
}
```

#### OrderProcessor Tests
```csharp
public class OrderProcessorTests
{
    [Fact]
    public void CalculateTotal_ValidOrder_ReturnsCorrectTotal()
    {
        // Arrange
        var order = new Order
        {
            Items = new List<OrderItem>
            {
                new OrderItem { ProductId = 1, Quantity = 2, UnitPrice = 10.00m },
                new OrderItem { ProductId = 2, Quantity = 1, UnitPrice = 25.00m }
            }
        };

        var processor = new OrderProcessor();

        // Act
        var total = processor.CalculateTotal(order);

        // Assert
        Assert.Equal(45.00m, total);
    }

    [Fact]
    public void CalculateTotal_EmptyOrder_ReturnsZero()
    {
        // Arrange
        var order = new Order { Items = new List<OrderItem>() };
        var processor = new OrderProcessor();

        // Act
        var total = processor.CalculateTotal(order);

        // Assert
        Assert.Equal(0.00m, total);
    }

    [Fact]
    public void ApplyDiscount_ValidDiscount_ReturnsDiscountedTotal()
    {
        // Arrange
        var order = new Order { Subtotal = 100.00m };
        var discount = new Discount { Type = DiscountType.Percentage, Value = 10 };

        var processor = new OrderProcessor();

        // Act
        var discountedTotal = processor.ApplyDiscount(order.Subtotal, discount);

        // Assert
        Assert.Equal(90.00m, discountedTotal);
    }
}
```

### Data Access Tests

#### CustomerRepository Tests
```csharp
public class CustomerRepositoryTests
{
    private readonly DbContextOptions<AppDbContext> _options;

    public CustomerRepositoryTests()
    {
        _options = new DbContextOptionsBuilder<AppDbContext>()
            .UseInMemoryDatabase(databaseName: "TestDatabase")
            .Options;
    }

    [Fact]
    public async Task GetById_ExistingCustomer_ReturnsCustomer()
    {
        // Arrange
        using var context = new AppDbContext(_options);
        var repository = new CustomerRepository(context);

        var customer = new Customer { Name = "Test Customer", Email = "test@example.com" };
        context.Customers.Add(customer);
        await context.SaveChangesAsync();

        // Act
        var result = await repository.GetByIdAsync(customer.Id);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(customer.Name, result.Name);
    }

    [Fact]
    public async Task GetById_NonExistingCustomer_ReturnsNull()
    {
        // Arrange
        using var context = new AppDbContext(_options);
        var repository = new CustomerRepository(context);

        // Act
        var result = await repository.GetByIdAsync(999);

        // Assert
        Assert.Null(result);
    }
}
```

## Integration Testing

### Database Integration Tests
```csharp
public class DatabaseIntegrationTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public DatabaseIntegrationTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public async Task CreateOrder_WithTransaction_RollsBackOnFailure()
    {
        // Arrange
        using var transaction = await _fixture.Context.Database.BeginTransactionAsync();
        var orderService = new OrderService(_fixture.Context);

        var order = new Order
        {
            CustomerId = 1,
            Items = new List<OrderItem>
            {
                new OrderItem { ProductId = 999, Quantity = 1 } // Non-existing product
            }
        };

        // Act & Assert
        await Assert.ThrowsAsync<InvalidOperationException>(() =>
            orderService.CreateOrderAsync(order));

        // Verify rollback
        var createdOrder = await _fixture.Context.Orders.FindAsync(order.Id);
        Assert.Null(createdOrder);
    }
}
```

### API Integration Tests
```csharp
public class CustomerApiIntegrationTests : IClassFixture<WebApplicationFactory<Startup>>
{
    private readonly WebApplicationFactory<Startup> _factory;
    private readonly HttpClient _client;

    public CustomerApiIntegrationTests(WebApplicationFactory<Startup> factory)
    {
        _factory = factory;
        _client = _factory.CreateClient();
    }

    [Fact]
    public async Task GetCustomers_ReturnsSuccessStatusCode()
    {
        // Act
        var response = await _client.GetAsync("/api/customers");

        // Assert
        response.EnsureSuccessStatusCode();
        var customers = await response.Content.ReadFromJsonAsync<List<Customer>>();
        Assert.NotNull(customers);
    }

    [Fact]
    public async Task CreateCustomer_ValidData_ReturnsCreatedCustomer()
    {
        // Arrange
        var customer = new Customer
        {
            Name = "Integration Test Customer",
            Email = "integration@test.com"
        };

        // Act
        var response = await _client.PostAsJsonAsync("/api/customers", customer);

        // Assert
        response.EnsureSuccessStatusCode();
        var createdCustomer = await response.Content.ReadFromJsonAsync<Customer>();
        Assert.NotNull(createdCustomer);
        Assert.Equal(customer.Name, createdCustomer.Name);
    }
}
```

## Functional Testing

### End-to-End Test Scenarios

#### Customer Management Workflow
```csharp
[Collection("E2E Tests")]
public class CustomerManagementE2ETests
{
    [Fact]
    public async Task CompleteCustomerLifecycle_Success()
    {
        // Arrange
        var customerData = new
        {
            Name = "E2E Test Customer",
            Email = "e2e@test.com",
            Phone = "555-0123"
        };

        // Act: Create customer
        var createResponse = await _client.PostAsJsonAsync("/api/customers", customerData);
        createResponse.EnsureSuccessStatusCode();
        var createdCustomer = await createResponse.Content.ReadFromJsonAsync<Customer>();

        // Act: Update customer
        createdCustomer.Phone = "555-0124";
        var updateResponse = await _client.PutAsJsonAsync($"/api/customers/{createdCustomer.Id}", createdCustomer);
        updateResponse.EnsureSuccessStatusCode();

        // Act: Retrieve customer
        var getResponse = await _client.GetAsync($"/api/customers/{createdCustomer.Id}");
        getResponse.EnsureSuccessStatusCode();
        var retrievedCustomer = await getResponse.Content.ReadFromJsonAsync<Customer>();

        // Assert
        Assert.Equal("555-0124", retrievedCustomer.Phone);

        // Act: Delete customer
        var deleteResponse = await _client.DeleteAsync($"/api/customers/{createdCustomer.Id}");
        deleteResponse.EnsureSuccessStatusCode();

        // Verify deletion
        var verifyResponse = await _client.GetAsync($"/api/customers/{createdCustomer.Id}");
        Assert.Equal(HttpStatusCode.NotFound, verifyResponse.StatusCode);
    }
}
```

#### Order Processing Workflow
```csharp
[Fact]
public async Task ProcessOrder_CompleteWorkflow_Success()
{
    // Arrange: Create customer and products
    var customer = await CreateTestCustomer();
    var products = await CreateTestProducts(2);

    var orderData = new
    {
        CustomerId = customer.Id,
        Items = new[]
        {
            new { ProductId = products[0].Id, Quantity = 2 },
            new { ProductId = products[1].Id, Quantity = 1 }
        }
    };

    // Act: Create order
    var response = await _client.PostAsJsonAsync("/api/orders", orderData);
    response.EnsureSuccessStatusCode();
    var order = await response.Content.ReadFromJsonAsync<Order>();

    // Assert: Verify order details
    Assert.Equal(2, order.Items.Count);
    Assert.True(order.Total > 0);

    // Act: Process payment (mock)
    var paymentResponse = await _client.PostAsJsonAsync($"/api/orders/{order.Id}/payment",
        new { Amount = order.Total, PaymentMethod = "CreditCard" });
    paymentResponse.EnsureSuccessStatusCode();

    // Assert: Verify order status
    var updatedOrderResponse = await _client.GetAsync($"/api/orders/{order.Id}");
    var updatedOrder = await updatedOrderResponse.Content.ReadFromJsonAsync<Order>();
    Assert.Equal(OrderStatus.Processing, updatedOrder.Status);
}
```

## Code Coverage Strategy

### Coverage Targets
- **Overall Coverage**: 97%
- **Business Logic**: 100%
- **Data Access**: 95%
- **API Controllers**: 90%
- **Utility Classes**: 80%

### Coverage Configuration
```xml
<!-- coverlet.runsettings -->
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="XPlat code coverage">
        <Configuration>
          <Format>json,cobertura,lcov,teamcity</Format>
          <Include>[YourProject]*</Include>
          <Exclude>[YourProject]Test*</Exclude>
          <ExcludeByAttribute>Obsolete,GeneratedCodeAttribute,CompilerGeneratedAttribute</ExcludeByAttribute>
          <SingleHit>false</SingleHit>
          <UseSourceLink>true</UseSourceLink>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
```

### Coverage Analysis
```csharp
// Program.cs for coverage reporting
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();

// Configure coverage
builder.Services.AddCodeCoverage(options =>
{
    options.ReportPath = "coverage";
    options.MinimumCoverage = 97;
    options.FailIfBelowMinimum = true;
});
```

## Test Data Management

### Test Data Setup
```csharp
public class TestDataSeeder
{
    public static async Task SeedTestData(AppDbContext context)
    {
        // Create test customers
        var customers = new List<Customer>
        {
            new Customer { Name = "Test Customer 1", Email = "test1@example.com" },
            new Customer { Name = "Test Customer 2", Email = "test2@example.com" }
        };

        await context.Customers.AddRangeAsync(customers);

        // Create test products
        var products = new List<Product>
        {
            new Product { Name = "Test Product 1", Price = 10.00m },
            new Product { Name = "Test Product 2", Price = 25.00m }
        };

        await context.Products.AddRangeAsync(products);
        await context.SaveChangesAsync();
    }
}
```

## Performance Testing

### Load Testing
```csharp
public class PerformanceTests
{
    [Fact]
    public async Task GetCustomers_UnderLoad_ReturnsWithinTimeLimit()
    {
        // Arrange
        var tasks = new List<Task>();
        var stopwatch = Stopwatch.StartNew();

        // Act: Simulate concurrent requests
        for (int i = 0; i < 100; i++)
        {
            tasks.Add(_client.GetAsync("/api/customers"));
        }

        await Task.WhenAll(tasks);
        stopwatch.Stop();

        // Assert
        Assert.True(stopwatch.ElapsedMilliseconds < 5000); // 5 second limit
    }
}
```

## Continuous Integration

### CI Pipeline Configuration
```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.0.x'
    - name: Restore dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test with Coverage
      run: dotnet test --no-build --verbosity normal /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
    - name: Upload Coverage
      uses: codecov/codecov-action@v1
```

## Test Reporting

### Automated Reporting
- **Coverage Reports**: Generated on each build
- **Test Results**: Published to CI dashboard
- **Performance Metrics**: Tracked over time
- **Quality Gates**: Block merges if coverage < 97%

### Manual Testing Checklist
- [ ] User interface functionality
- [ ] Business rule validation
- [ ] Error handling scenarios
- [ ] Performance under load
- [ ] Security testing
- [ ] Accessibility compliance