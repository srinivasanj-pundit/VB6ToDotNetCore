# Build and Security

This document outlines the build process, security assessment, and vulnerability management strategy for the migrated .NET Core application.

## Build Configuration

### Project Structure
```
VB6ToDotNetCore.sln
├── src/
│   ├── VB6ToDotNetCore.Core/ (Business logic)
│   ├── VB6ToDotNetCore.Data/ (Data access)
│   ├── VB6ToDotNetCore.Web/ (Web API)
│   ├── VB6ToDotNetCore.UI/ (WPF/WinForms)
│   └── VB6ToDotNetCore.Worker/ (Background services)
├── tests/
│   ├── VB6ToDotNetCore.Core.Tests/
│   ├── VB6ToDotNetCore.Data.Tests/
│   ├── VB6ToDotNetCore.Web.Tests/
│   └── VB6ToDotNetCore.IntegrationTests/
├── tools/
│   └── VB6ToDotNetCore.MigrationTools/
└── build/
    ├── Directory.Build.props
    ├── Directory.Build.targets
    └── VB6ToDotNetCore.Build.props
```

### Build Properties
```xml
<!-- Directory.Build.props -->
<Project>
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <LangVersion>10.0</LangVersion>
    <Nullable>enable</Nullable>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <CodeAnalysisRuleSet>$(MSBuildThisFileDirectory)CodeAnalysis.ruleset</CodeAnalysisRuleSet>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.1.118">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Roslynator.Analyzers" Version="4.1.0">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

### Build Targets
```xml
<!-- Directory.Build.targets -->
<Project>
  <Target Name="SecurityScan" AfterTargets="Build">
    <Exec Command="dotnet tool run security-scan --project $(MSBuildProjectFile)" />
  </Target>

  <Target Name="VulnerabilityCheck" AfterTargets="Restore">
    <Exec Command="dotnet list package --vulnerable" />
  </Target>
</Project>
```

## Build Scripts

### PowerShell Build Script
```powershell
# build.ps1
param(
    [string]$Configuration = "Release",
    [switch]$Clean,
    [switch]$Test,
    [switch]$Pack,
    [switch]$Publish
)

$ErrorActionPreference = "Stop"

if ($Clean) {
    Write-Host "Cleaning solution..." -ForegroundColor Green
    dotnet clean --configuration $Configuration
}

Write-Host "Restoring packages..." -ForegroundColor Green
dotnet restore

Write-Host "Building solution..." -ForegroundColor Green
dotnet build --configuration $Configuration --no-restore

if ($Test) {
    Write-Host "Running tests..." -ForegroundColor Green
    dotnet test --configuration $Configuration --no-build --verbosity normal
}

if ($Pack) {
    Write-Host "Creating packages..." -ForegroundColor Green
    dotnet pack --configuration $Configuration --no-build --output ./artifacts
}

if ($Publish) {
    Write-Host "Publishing..." -ForegroundColor Green
    dotnet publish VB6ToDotNetCore.Web --configuration $Configuration --output ./publish
}
```

### Azure DevOps Pipeline
```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '6.0.x'

steps:
- task: UseDotNet@2
  displayName: 'Install .NET Core SDK'
  inputs:
    version: $(dotnetVersion)

- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build projects'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'Run tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration $(buildConfiguration) --no-build --collect "Code coverage"'

- task: DotNetCoreCLI@2
  displayName: 'Publish artifacts'
  inputs:
    command: 'publish'
    projects: 'src/VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj'
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish build artifacts'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
```

## Security Assessment

### Vulnerability Scanning Tools
- **NuGet Package Vulnerabilities**: `dotnet list package --vulnerable`
- **Static Application Security Testing (SAST)**: SonarQube, CodeQL
- **Container Scanning**: Trivy (for Docker images)
- **Dependency Scanning**: OWASP Dependency Check

### Security Code Analysis Rules
```xml
<!-- CodeAnalysis.ruleset -->
<RuleSet Name="VB6ToDotNetCore Rules" Description="Security-focused code analysis rules">
  <Rules AnalyzerId="Microsoft.CodeAnalysis.CSharp" RuleNamespace="Microsoft.CodeAnalysis.CSharp">
    <Rule Id="CA2100" Action="Error" /> <!-- Review SQL queries for security vulnerabilities -->
    <Rule Id="CA3001" Action="Error" /> <!-- Review code for SQL injection vulnerabilities -->
    <Rule Id="CA3002" Action="Error" /> <!-- Review code for XSS vulnerabilities -->
    <Rule Id="CA3003" Action="Error" /> <!-- Review code for file path injection vulnerabilities -->
    <Rule Id="CA3004" Action="Error" /> <!-- Review code for information disclosure vulnerabilities -->
    <Rule Id="CA3005" Action="Error" /> <!-- Review code for LDAP injection vulnerabilities -->
    <Rule Id="CA3006" Action="Error" /> <!-- Review code for process command injection vulnerabilities -->
  </Rules>
</RuleSet>
```

## Common Security Vulnerabilities and Resolutions

### SQL Injection Prevention
```csharp
// Vulnerable (DO NOT USE)
public List<Customer> GetCustomers(string name)
{
    using var connection = new SqlConnection(_connectionString);
    var query = $"SELECT * FROM Customers WHERE Name = '{name}'";
    // ...
}

// Secure
public async Task<List<Customer>> GetCustomersAsync(string name)
{
    using var connection = new SqlConnection(_connectionString);
    var query = "SELECT * FROM Customers WHERE Name = @Name";
    using var command = new SqlCommand(query, connection);
    command.Parameters.AddWithValue("@Name", name);

    await connection.OpenAsync();
    using var reader = await command.ExecuteReaderAsync();

    var customers = new List<Customer>();
    while (await reader.ReadAsync())
    {
        customers.Add(new Customer
        {
            Id = reader.GetInt32(0),
            Name = reader.GetString(1),
            Email = reader.GetString(2)
        });
    }

    return customers;
}
```

### Cross-Site Scripting (XSS) Protection
```csharp
// In ASP.NET Core Controller
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult CreateCustomer([FromBody] CustomerModel model)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    // Model binding automatically handles XSS prevention
    var customer = new Customer
    {
        Name = model.Name, // Automatically sanitized
        Email = model.Email
    };

    // Additional server-side validation
    if (!IsValidEmail(model.Email))
    {
        return BadRequest("Invalid email format");
    }

    _customerService.CreateCustomer(customer);
    return CreatedAtAction(nameof(GetCustomer), new { id = customer.Id }, customer);
}
```

### Authentication and Authorization
```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options =>
        {
            options.TokenValidationParameters = new TokenValidationParameters
            {
                ValidateIssuer = true,
                ValidateAudience = true,
                ValidateLifetime = true,
                ValidateIssuerSigningKey = true,
                ValidIssuer = Configuration["Jwt:Issuer"],
                ValidAudience = Configuration["Jwt:Audience"],
                IssuerSigningKey = new SymmetricSecurityKey(
                    Encoding.UTF8.GetBytes(Configuration["Jwt:Key"]))
            };
        });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("AdminOnly", policy =>
            policy.RequireRole("Admin"));
        options.AddPolicy("CustomerAccess", policy =>
            policy.RequireClaim("CustomerId"));
    });
}

// Controller with authorization
[Authorize]
[ApiController]
[Route("api/[controller]")]
public class CustomersController : ControllerBase
{
    [HttpGet("{id}")]
    [Authorize(Policy = "CustomerAccess")]
    public IActionResult GetCustomer(int id)
    {
        // Implementation
    }

    [HttpPost]
    [Authorize(Roles = "Admin")]
    public IActionResult CreateCustomer([FromBody] Customer customer)
    {
        // Implementation
    }
}
```

### Data Protection
```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"C:\keys"))
        .ProtectKeysWithCertificate("thumbprint");
}

// Usage in service
public class EncryptionService
{
    private readonly IDataProtector _protector;

    public EncryptionService(IDataProtectionProvider provider)
    {
        _protector = provider.CreateProtector("CustomerData");
    }

    public string EncryptSensitiveData(string plainText)
    {
        return _protector.Protect(plainText);
    }

    public string DecryptSensitiveData(string protectedText)
    {
        return _protector.Unprotect(protectedText);
    }
}
```

### Secure Configuration Management
```json
// appsettings.json (Development)
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=VB6ToDotNetCore;Trusted_Connection=True;"
  },
  "Jwt": {
    "Key": "your-development-key-here",
    "Issuer": "VB6ToDotNetCore",
    "Audience": "VB6ToDotNetCoreUsers"
  },
  "Encryption": {
    "Key": "development-encryption-key"
  }
}
```

```csharp
// Program.cs
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json", optional: true)
    .AddEnvironmentVariables()
    .AddAzureKeyVault(new Uri("https://your-keyvault.vault.azure.net/"), new DefaultAzureCredential())
    .Build();
```

## Dependency Management

### NuGet Package Audit
```xml
<!-- VB6ToDotNetCore.Build.props -->
<Project>
  <PropertyGroup>
    <NuGetAudit>true</NuGetAudit>
    <NuGetAuditLevel>moderate</NuGetAuditLevel>
  </PropertyGroup>
</Project>
```

### Package Update Strategy
- **Patch Updates**: Apply immediately (security fixes)
- **Minor Updates**: Test and apply within 1 week
- **Major Updates**: Plan migration with testing phase

### Vulnerable Package Handling
```powershell
# check-vulnerabilities.ps1
dotnet list package --vulnerable --include-transitive | Out-File -FilePath vulnerabilities.txt

if (Get-Content vulnerabilities.txt | Select-String "Critical|High") {
    Write-Error "Critical or High vulnerabilities found!"
    exit 1
}
```

## Container Security

### Dockerfile Best Practices
```dockerfile
# Use official .NET runtime as base image
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Use SDK for building
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src

# Copy and restore dependencies
COPY ["VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj", "VB6ToDotNetCore.Web/"]
RUN dotnet restore "VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj"

# Copy source and build
COPY . .
WORKDIR "/src/VB6ToDotNetCore.Web"
RUN dotnet build "VB6ToDotNetCore.Web.csproj" -c Release -o /app/build

# Publish
FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Web.csproj" -c Release -o /app/publish

# Final stage
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .

# Create non-root user
RUN adduser --disabled-password --gecos '' appuser && chown -R appuser:appuser /app
USER appuser

ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Web.dll"]
```

### Container Scanning
```yaml
# .github/workflows/container-scan.yml
name: Container Security Scan
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Build Docker image
      run: docker build -t vb6-dotnet-core .

    - name: Scan image
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'image'
        scan-ref: 'vb6-dotnet-core'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v1
      if: always()
      with:
        sarif_file: 'trivy-results.sarif'
```

## Security Monitoring

### Application Insights Configuration
```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry();

    // Configure security monitoring
    services.Configure<TelemetryConfiguration>(config =>
    {
        config.TelemetryProcessorChainBuilder.Use(next =>
            new SecurityTelemetryProcessor(next));
    });
}
```

### Custom Security Telemetry
```csharp
public class SecurityTelemetryProcessor : ITelemetryProcessor
{
    private readonly ITelemetryProcessor _next;

    public SecurityTelemetryProcessor(ITelemetryProcessor next)
    {
        _next = next;
    }

    public void Process(ITelemetry item)
    {
        if (item is ExceptionTelemetry exceptionTelemetry)
        {
            // Check for sensitive data in exception messages
            if (ContainsSensitiveData(exceptionTelemetry.Exception?.Message))
            {
                // Sanitize or drop the telemetry
                return;
            }
        }

        _next.Process(item);
    }

    private bool ContainsSensitiveData(string? message)
    {
        if (string.IsNullOrEmpty(message)) return false;

        // Check for connection strings, passwords, etc.
        return message.Contains("password") ||
               message.Contains("connection string") ||
               message.Contains("api key");
    }
}
```

## Compliance and Auditing

### Security Headers
```csharp
// Startup.cs
public void Configure(IApplicationBuilder app)
{
    app.UseSecurityHeaders(options =>
    {
        options.AddContentSecurityPolicy(builder =>
        {
            builder.AddDefaultSrc().Self();
            builder.AddScriptSrc().Self().WithNonce();
            builder.AddStyleSrc().Self();
        });

        options.AddStrictTransportSecurityMaxAgeIncludeSubdomains(maxAgeInSeconds: 63072000);
        options.AddReferrerPolicyStrictOriginWhenCrossOrigin();
        options.AddXContentTypeOptionsNoSniff();
        options.AddXFrameOptionsDeny();
    });
}
```

### Audit Logging
```csharp
public class AuditService
{
    private readonly ILogger<AuditService> _logger;

    public AuditService(ILogger<AuditService> _logger)
    {
        _logger = logger;
    }

    public void LogSecurityEvent(string eventType, string details, string userId = null)
    {
        var auditEntry = new
        {
            Timestamp = DateTime.UtcNow,
            EventType = eventType,
            UserId = userId,
            Details = details,
            IpAddress = GetClientIpAddress(),
            UserAgent = GetUserAgent()
        };

        _logger.LogInformation("Security Audit: {@AuditEntry}", auditEntry);
    }

    public void LogAuthenticationAttempt(string username, bool success, string reason = null)
    {
        LogSecurityEvent(
            "Authentication",
            success ? "Successful login" : $"Failed login: {reason}",
            username);
    }
}
```

## Incident Response Plan

### Security Incident Process
1. **Detection**: Monitor alerts and logs
2. **Assessment**: Evaluate impact and scope
3. **Containment**: Isolate affected systems
4. **Recovery**: Restore from clean backups
5. **Lessons Learned**: Update security measures

### Emergency Contacts
- **Security Team**: security@company.com
- **DevOps Team**: devops@company.com
- **Management**: management@company.com

### Communication Template
```
Subject: Security Incident - [Brief Description]

Incident Details:
- Date/Time: [Timestamp]
- Affected Systems: [List]
- Impact: [Description]
- Current Status: [Investigation/Contained/Resolved]
- Next Steps: [Actions]

Please acknowledge receipt.
```

This comprehensive build and security strategy ensures the migrated application maintains high security standards while providing robust build processes for continuous integration and deployment.