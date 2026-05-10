# Technical Migration Requirements

## .NET Core 10 Specifications

### Framework Requirements
- **Target Framework**: .NET 10.0
- **Language Version**: C# 12.0
- **Nullable Reference Types**: Enabled
- **Implicit Usings**: Enabled
- **Top-level Statements**: Allowed where appropriate

### Project Structure Requirements
```
VB6ToDotNetCore.sln
├── src/
│   ├── VB6ToDotNetCore.Core/           # Domain models, business logic
│   │   ├── Entities/
│   │   ├── Services/
│   │   ├── Interfaces/
│   │   ├── Validators/
│   │   ├── Exceptions/
│   │   └── DTOs/
│   ├── VB6ToDotNetCore.Data/           # Data access layer
│   │   ├── Context/
│   │   ├── Repositories/
│   │   ├── Migrations/
│   │   └── Configurations/
│   ├── VB6ToDotNetCore.Web/            # Web API
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Filters/
│   │   ├── Middleware/
│   │   └── Extensions/
│   ├── VB6ToDotNetCore.UI/             # Desktop UI
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   ├── Controls/
│   │   └── Services/
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

### Package Dependencies

#### Core Packages
```xml
<PackageReference Include="Microsoft.EntityFrameworkCore" Version="10.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="10.0.0" />
<PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="10.0.0" />
<PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="10.0.0" />
<PackageReference Include="Microsoft.Extensions.Logging" Version="10.0.0" />
<PackageReference Include="Microsoft.Extensions.Configuration" Version="10.0.0" />
```

#### Web API Packages
```xml
<PackageReference Include="Microsoft.AspNetCore.Mvc" Version="10.0.0" />
<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="10.0.0" />
<PackageReference Include="Swashbuckle.AspNetCore" Version="7.0.0" />
<PackageReference Include="Serilog.AspNetCore" Version="8.0.0" />
<PackageReference Include="Microsoft.AspNetCore.DataProtection" Version="10.0.0" />
```

#### Testing Packages
```xml
<PackageReference Include="xunit" Version="2.9.0" />
<PackageReference Include="xunit.runner.visualstudio" Version="2.8.2" />
<PackageReference Include="Moq" Version="4.20.70" />
<PackageReference Include="AutoFixture" Version="4.18.1" />
<PackageReference Include="AutoFixture.AutoMoq" Version="4.18.1" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
<PackageReference Include="coverlet.collector" Version="6.0.2" />
```

#### UI Packages (WPF)
```xml
<PackageReference Include="Microsoft.Xaml.Behaviors.Wpf" Version="1.1.39" />
<PackageReference Include="MaterialDesignThemes" Version="4.9.0" />
<PackageReference Include="Prism.Core" Version="8.1.97" />
<PackageReference Include="Prism.Wpf" Version="8.1.97" />
```

### Coding Standards

#### Naming Conventions
- **Classes**: PascalCase
- **Methods**: PascalCase
- **Properties**: PascalCase
- **Fields**: camelCase with underscore prefix
- **Constants**: PascalCase
- **Interfaces**: PascalCase with 'I' prefix
- **Enums**: PascalCase

#### Code Quality Rules
```xml
<!-- .editorconfig -->
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.cs]
indent_style = space
indent_size = 4

# Code style rules
dotnet_style_qualification_for_field = false:suggestion
dotnet_style_qualification_for_property = false:suggestion
dotnet_style_qualification_for_method = false:suggestion
dotnet_style_qualification_for_event = false:suggestion

# Language rules
csharp_style_var_for_built_in_types = true:suggestion
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = true:suggestion

# Expression-bodied members
csharp_style_expression_bodied_methods = true:suggestion
csharp_style_expression_bodied_properties = true:suggestion
csharp_style_expression_bodied_indexers = true:suggestion
csharp_style_expression_bodied_accessors = true:suggestion

# Pattern matching
csharp_style_pattern_matching_over_is_with_cast_check = true:suggestion
csharp_style_pattern_matching_over_as_with_null_check = true:suggestion

# Null checking
csharp_style_conditional_delegate_call = true:suggestion

# Modifier preferences
csharp_preferred_modifier_order = public,private,protected,internal,static,extern,new,virtual,abstract,sealed,override,readonly,unsafe,volatile,async:suggestion

# Expression-level preferences
csharp_prefer_braces = true:suggestion
csharp_style_deconstructed_variable_declaration = true:suggestion
csharp_prefer_simple_default_expression = true:suggestion
csharp_style_prefer_local_over_anonymous_function = true:suggestion
csharp_style_inlined_variable_declaration = true:suggestion
```

#### Architecture Patterns
- **Dependency Injection**: Constructor injection preferred
- **Repository Pattern**: For data access abstraction
- **Unit of Work**: For transaction management
- **Service Layer**: For business logic encapsulation
- **MVVM**: For UI separation of concerns
- **Mediator Pattern**: For complex interactions

### Security Requirements

#### Authentication & Authorization
- JWT Bearer token authentication
- Role-based access control (RBAC)
- Claims-based authorization
- Secure password hashing (BCrypt)
- Account lockout policies

#### Data Protection
- AES encryption for sensitive data
- Data Protection API for configuration
- Secure connection strings
- Input validation and sanitization

#### Security Headers
- Content Security Policy (CSP)
- HTTP Strict Transport Security (HSTS)
- X-Frame-Options
- X-Content-Type-Options
- Referrer Policy

### Performance Requirements

#### Response Times
- API endpoints: < 500ms for 95% of requests
- UI interactions: < 100ms response time
- Database queries: < 200ms average
- Report generation: < 30 seconds

#### Scalability Targets
- Concurrent users: 1000+
- Requests per second: 100 RPS
- Database connections: Connection pooling
- Memory usage: < 512MB per service instance

#### Caching Strategy
- Redis for distributed caching
- In-memory caching for static data
- Output caching for API responses
- CDN for static assets

### Monitoring and Logging

#### Logging Requirements
- Structured logging with Serilog
- Log levels: Trace, Debug, Information, Warning, Error, Fatal
- Centralized logging with correlation IDs
- Performance logging for slow operations
- Security event logging

#### Health Checks
- Application health endpoints
- Database connectivity checks
- External service dependencies
- Custom health check implementations

#### Metrics Collection
- Response times and throughput
- Error rates and types
- Resource utilization (CPU, memory, disk)
- Business metrics (user activity, transaction volume)

### Testing Requirements

#### Code Coverage
- Overall coverage: 95% minimum
- Business logic: 100% coverage
- Data access layer: 95% coverage
- API controllers: 90% coverage
- UI code: 80% coverage (where testable)

#### Test Categories
- **Unit Tests**: Individual method/component testing
- **Integration Tests**: Component interaction testing
- **API Tests**: Endpoint testing with authentication
- **UI Tests**: Automated UI interaction testing
- **Performance Tests**: Load and stress testing
- **Security Tests**: Penetration testing and vulnerability scanning

### Deployment Requirements

#### Containerization
- Multi-stage Docker builds
- Non-root user execution
- Health check implementations
- Security scanning integration

#### Orchestration
- Kubernetes manifests for production
- Horizontal Pod Autoscaling
- ConfigMaps and Secrets management
- Rolling deployment strategy

#### CI/CD Pipeline
- Automated builds on commit
- Multi-stage testing (unit → integration → e2e)
- Security scanning integration
- Automated deployment to staging/production

### Documentation Requirements

#### API Documentation
- OpenAPI/Swagger specifications
- Request/response examples
- Authentication requirements
- Error response formats

#### Code Documentation
- XML documentation comments
- Architecture decision records (ADRs)
- README files for each component
- Deployment and configuration guides

#### User Documentation
- User manuals and guides
- API integration guides
- Troubleshooting documentation
- Training materials

### Compliance Requirements

#### Data Protection
- GDPR compliance for EU users
- Data encryption at rest and in transit
- Data retention policies
- Right to erasure implementation

#### Accessibility
- WCAG 2.1 AA compliance
- Keyboard navigation support
- Screen reader compatibility
- High contrast theme support

#### Industry Standards
- OWASP security standards
- REST API design standards
- Database normalization standards
- Code quality standards (SonarQube)