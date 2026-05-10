# Build and Deployment Guide

## .NET Core Application Build Process

This document provides comprehensive guidelines for building, packaging, and deploying the migrated .NET Core 10 application.

## Development Environment Setup

### Prerequisites

#### Required Software
```powershell
# Install .NET Core 10 SDK
winget install Microsoft.DotNet.SDK.10

# Install Visual Studio 2022 Enterprise
winget install Microsoft.VisualStudio.2022.Enterprise

# Install Git
winget install Git.Git

# Install Azure CLI
winget install Microsoft.AzureCLI

# Install Docker Desktop
winget install Docker.DockerDesktop
```

#### Development Tools Configuration
```bash
# Configure .NET CLI
dotnet --version  # Should show 10.x.x
dotnet tool install --global dotnet-ef
dotnet tool install --global dotnet-reportgenerator-globaltool

# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"

# Configure Azure DevOps
az login
az devops configure --defaults organization=https://dev.azure.com/your-org project=VB6ToDotNetCore
```

### Project Structure Setup

#### Solution Configuration
```xml
<!-- VB6ToDotNetCore.sln -->
Microsoft Visual Studio Solution File, Format Version 12.00
# Visual Studio Version 17
VisualStudioVersion = 17.0.31903.59
MinimumVisualStudioVersion = 10.0.40219.1
Project("{9A19103F-16F7-4668-BE54-9A1E7A4F7556}") = "VB6ToDotNetCore.Core", "src\VB6ToDotNetCore.Core\VB6ToDotNetCore.Core.csproj", "{GUID-1}"
EndProject
Project("{9A19103F-16F7-4668-BE54-9A1E7A4F7556}") = "VB6ToDotNetCore.Data", "src\VB6ToDotNetCore.Data\VB6ToDotNetCore.Data.csproj", "{GUID-2}"
EndProject
Project("{9A19103F-16F7-4668-BE54-9A1E7A4F7556}") = "VB6ToDotNetCore.Web", "src\VB6ToDotNetCore.Web\VB6ToDotNetCore.Web.csproj", "{GUID-3}"
EndProject
Project("{9A19103F-16F7-4668-BE54-9A1E7A4F7556}") = "VB6ToDotNetCore.UI", "src\VB6ToDotNetCore.UI\VB6ToDotNetCore.UI.csproj", "{GUID-4}"
EndProject
Global
    GlobalSection(SolutionConfigurationPlatforms) = preSolution
        Debug|Any CPU = Debug|Any CPU
        Release|Any CPU = Release|Any CPU
    EndSection
EndGlobal
```

#### Directory.Build.props
```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <LangVersion>12.0</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <CodeAnalysisRuleSet>..\build\VB6ToDotNetCore.ruleset</CodeAnalysisRuleSet>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.2.0-beta.556">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>
</Project>
```

## Build Process

### Local Development Build

#### Build Commands
```bash
# Clean solution
dotnet clean

# Restore packages
dotnet restore

# Build solution
dotnet build

# Build specific project
dotnet build src/VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj

# Build with detailed output
dotnet build --verbosity detailed

# Build for specific configuration
dotnet build --configuration Release
```

#### Build Verification
```bash
# Run tests
dotnet test

# Run specific test project
dotnet test tests/VB6ToDotNetCore.Core.Tests/VB6ToDotNetCore.Core.Tests.csproj

# Run tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Generate coverage report
reportgenerator -reports:**/coverage.cobertura.xml -targetdir:./coverage -reporttypes:Html
```

### CI/CD Pipeline Configuration

#### Azure DevOps Pipeline (azure-pipelines.yml)
```yaml
trigger:
  branches:
    include:
    - main
    - develop
  paths:
    exclude:
    - docs/
    - README.md

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '10.x'

steps:
- task: UseDotNet@2
  displayName: 'Install .NET Core $(dotnetVersion)'
  inputs:
    version: '$(dotnetVersion)'

- task: DotNetCoreCLI@2
  displayName: 'Restore packages'
  inputs:
    command: 'restore'
    projects: '**/*.csproj'

- task: DotNetCoreCLI@2
  displayName: 'Build solution'
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration) --no-restore'

- task: DotNetCoreCLI@2
  displayName: 'Run unit tests'
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration $(buildConfiguration) --no-build --collect "XPlat Code Coverage"'

- task: DotNetCoreCLI@2
  displayName: 'Publish web application'
  inputs:
    command: 'publish'
    publishWebProjects: true
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/web'
    zipAfterPublish: true

- task: DotNetCoreCLI@2
  displayName: 'Publish WPF application'
  inputs:
    command: 'publish'
    projects: 'src/VB6ToDotNetCore.UI/VB6ToDotNetCore.UI.csproj'
    arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/ui --runtime win-x64 --self-contained true'
    zipAfterPublish: true

- task: PublishBuildArtifacts@1
  displayName: 'Publish build artifacts'
  inputs:
    pathtoPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
```

#### GitHub Actions Pipeline (.github/workflows/build.yml)
```yaml
name: Build and Test

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '10.x'

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --no-restore --configuration Release

    - name: Test
      run: dotnet test --no-build --verbosity normal --configuration Release --collect:"XPlat Code Coverage"

    - name: Upload coverage reports
      uses: codecov/codecov-action@v4
      with:
        file: ./coverage.cobertura.xml
```

## Packaging

### Web Application Packaging

#### ASP.NET Core Publish Profile
```xml
<!-- Properties/PublishProfiles/FolderProfile.pubxml -->
<Project>
  <PropertyGroup>
    <Configuration>Release</Configuration>
    <Platform>Any CPU</Platform>
    <PublishDir>bin\Release\net10.0\publish\</PublishDir>
    <PublishProtocol>FileSystem</PublishProtocol>
    <TargetFramework>net10.0</TargetFramework>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <SelfContained>true</SelfContained>
    <PublishSingleFile>false</PublishSingleFile>
    <PublishTrimmed>true</PublishTrimmed>
    <TrimMode>link</TrimMode>
  </PropertyGroup>
</Project>
```

#### Docker Containerization
```dockerfile
# Dockerfile.api
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY ["VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj", "VB6ToDotNetCore.Web/"]
COPY ["VB6ToDotNetCore.Core/VB6ToDotNetCore.Core.csproj", "VB6ToDotNetCore.Core/"]
COPY ["VB6ToDotNetCore.Data/VB6ToDotNetCore.Data.csproj", "VB6ToDotNetCore.Data/"]
RUN dotnet restore "VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj"
COPY . .
WORKDIR "/src/VB6ToDotNetCore.Web"
RUN dotnet build "VB6ToDotNetCore.Web.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "VB6ToDotNetCore.Web.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "VB6ToDotNetCore.Web.dll"]
```

### Desktop Application Packaging

#### WPF ClickOnce Deployment
```xml
<!-- VB6ToDotNetCore.UI.csproj -->
<PropertyGroup>
  <PublishProtocol>ClickOnce</PublishProtocol>
  <Install>true</Install>
  <InstallFrom>Web</InstallFrom>
  <UpdateEnabled>true</UpdateEnabled>
  <UpdateMode>Foreground</UpdateMode>
  <UpdateInterval>7</UpdateInterval>
  <UpdateIntervalUnits>Days</UpdateIntervalUnits>
  <IsWebBootstrapper>true</IsWebBootstrapper>
  <BootstrapperEnabled>true</BootstrapperEnabled>
  <PublishUrl>http://deploy.company.com/vb6-app/</PublishUrl>
  <InstallUrl>http://deploy.company.com/vb6-app/</InstallUrl>
  <SupportUrl>http://support.company.com</SupportUrl>
  <ErrorReportUrl>http://support.company.com/error</ErrorReportUrl>
  <ProductName>VB6 to .NET Core Application</ProductName>
  <PublisherName>Company Name</PublisherName>
  <SuiteName>Business Applications</SuiteName>
  <MinimumRequiredVersion>1.0.0.0</MinimumRequiredVersion>
  <CreateDesktopShortcut>true</CreateDesktopShortcut>
  <PublishWizardCompleted>true</PublishWizardCompleted>
</PropertyGroup>
```

#### MSI Installer Creation
```xml
<!-- VB6ToDotNetCore.UI.csproj -->
<ItemGroup>
  <PackageReference Include="WixToolset.Sdk" Version="4.0.1" />
</ItemGroup>

<Target Name="BuildInstaller" AfterTargets="Publish">
  <Exec Command="candle.exe Product.wxs -out Product.wixobj" />
  <Exec Command="light.exe Product.wixobj -out $(PublishDir)VB6ToDotNetCore.msi" />
</Target>
```

## Deployment Strategies

### Development Environment

#### Local Development Deployment
```bash
# Run web application
dotnet run --project src/VB6ToDotNetCore.Web/VB6ToDotNetCore.Web.csproj

# Run WPF application
dotnet run --project src/VB6ToDotNetCore.UI/VB6ToDotNetCore.UI.csproj

# Run with specific configuration
dotnet run --configuration Debug --environment Development

# Run database migrations
dotnet ef database update --project src/VB6ToDotNetCore.Data/VB6ToDotNetCore.Data.csproj
```

#### Docker Development
```bash
# Build and run with Docker Compose
docker-compose -f docker-compose.dev.yml up --build

# Run database migrations in container
docker-compose -f docker-compose.dev.yml exec api dotnet ef database update
```

### Staging Environment

#### Azure App Service Deployment
```yaml
# Azure DevOps deployment pipeline
- stage: DeployStaging
  dependsOn: Build
  jobs:
  - deployment: DeployWebApp
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'Azure-Subscription'
              appName: 'vb6-app-staging'
              package: '$(Pipeline.Workspace)/drop/web/VB6ToDotNetCore.Web.zip'

  - deployment: DeployDatabase
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: SqlAzureDacpacDeployment@1
            inputs:
              azureSubscription: 'Azure-Subscription'
              serverName: 'sql-staging.database.windows.net'
              databaseName: 'VB6ToDotNetCore-Staging'
              sqlUsername: '$(SqlUsername)'
              sqlPassword: '$(SqlPassword)'
              deployType: 'DacpacTask'
              dacpacFile: '$(Pipeline.Workspace)/drop/database/VB6ToDotNetCore.Database.dacpac'
```

#### Database Deployment
```sql
-- Database migration script
USE [VB6ToDotNetCore]
GO

-- Create new tables
CREATE TABLE [dbo].[Customers_New] (
    [CustomerID] INT IDENTITY(1,1) PRIMARY KEY,
    [CompanyName] NVARCHAR(100) NOT NULL,
    [ContactName] NVARCHAR(50) NULL,
    -- ... other columns
);

-- Migrate data
INSERT INTO [dbo].[Customers_New] (
    CompanyName, ContactName, Address, City, Phone, Email, IsActive
)
SELECT
    CompanyName, ContactName, Address, City, Phone, Email, 1
FROM [dbo].[Customers_Old];

-- Rename tables
EXEC sp_rename 'Customers_Old', 'Customers_Backup';
EXEC sp_rename 'Customers_New', 'Customers';

-- Update indexes and constraints
CREATE INDEX IX_Customers_CompanyName ON Customers(CompanyName);
CREATE INDEX IX_Customers_Email ON Customers(Email);
```

### Production Environment

#### Blue-Green Deployment
```yaml
# Production deployment with blue-green strategy
- stage: DeployProduction
  dependsOn: DeployStaging
  jobs:
  - deployment: BlueGreenDeployment
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            inputs:
              azureSubscription: 'Azure-Subscription'
              scriptType: 'pscore'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Get current production slot
                $currentSlot = az webapp deployment slot list --name vb6-app-prod --resource-group rg-vb6-app --query "[?isDefault].name" -o tsv

                # Determine target slot
                $targetSlot = if ($currentSlot -eq "blue") { "green" } else { "blue" }

                # Deploy to target slot
                az webapp deployment source config-zip --name vb6-app-prod --resource-group rg-vb6-app --slot $targetSlot --src $(Pipeline.Workspace)/drop/web/VB6ToDotNetCore.Web.zip

                # Run smoke tests on target slot
                # ... smoke test logic ...

                # Swap slots
                az webapp deployment slot swap --name vb6-app-prod --resource-group rg-vb6-app --slot $targetSlot
```

#### Kubernetes Deployment
```yaml
# Kubernetes manifests
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vb6-app-api
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
        image: vb6app.azurecr.io/api:$(Build.BuildId)
        ports:
        - containerPort: 80
        env:
        - name: ASPNETCORE_ENVIRONMENT
          value: "Production"
        - name: ConnectionStrings__DefaultConnection
          valueFrom:
            secretKeyRef:
              name: db-secret
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
```

## Configuration Management

### appsettings.json Structure
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=sql-prod.database.windows.net;Database=VB6ToDotNetCore;User ID=appuser;Password=***;"
  },
  "ExternalAPIs": {
    "PaymentGateway": {
      "BaseUrl": "https://api.paymentgateway.com",
      "ApiKey": "***",
      "Timeout": 30
    },
    "ShippingService": {
      "BaseUrl": "https://api.shipping.com",
      "ApiKey": "***"
    }
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    },
    "ApplicationInsights": {
      "InstrumentationKey": "***"
    }
  },
  "Security": {
    "Jwt": {
      "Issuer": "https://vb6-app.company.com",
      "Audience": "vb6-app-clients",
      "Key": "***"
    },
    "Cors": {
      "AllowedOrigins": ["https://vb6-app.company.com"],
      "AllowedMethods": ["GET", "POST", "PUT", "DELETE"],
      "AllowedHeaders": ["Authorization", "Content-Type"]
    }
  },
  "Features": {
    "EnableNewUI": true,
    "EnableAdvancedReporting": false,
    "EnableExternalIntegrations": true
  }
}
```

### Environment-Specific Configuration
```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=VB6ToDotNetCore-Dev;"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug"
    }
  }
}

// appsettings.Production.json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=sql-prod.database.windows.net;Database=VB6ToDotNetCore;User ID=appuser;Password=***;"
  },
  "Logging": {
    "ApplicationInsights": {
      "InstrumentationKey": "***"
    }
  }
}
```

## Monitoring and Alerting

### Application Insights Configuration
```xml
<!-- VB6ToDotNetCore.Web.csproj -->
<PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.21.0" />

<!-- Program.cs -->
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddApplicationInsightsTelemetry();
```

### Health Checks
```csharp
// Program.cs
builder.Services.AddHealthChecks()
    .AddSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"))
    .AddUrlGroup(new Uri("https://api.paymentgateway.com/health"), name: "Payment Gateway")
    .AddApplicationInsightsPublisher();

// Controller
[HttpGet("/health")]
public IActionResult Health() => Ok("Healthy");

[HttpGet("/health/ready")]
public async Task<IActionResult> Readiness([FromServices] HealthCheckService healthCheckService)
{
    var report = await healthCheckService.CheckHealthAsync();
    return report.Status == HealthStatus.Healthy ? Ok() : StatusCode(503);
}
```

### Log Aggregation
```json
// Serilog configuration
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File", "Serilog.Sinks.ApplicationInsights"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/vb6-app-.log",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 7
        }
      },
      {
        "Name": "ApplicationInsights",
        "Args": {
          "instrumentationKey": "***"
        }
      }
    ],
    "Enrich": ["FromLogContext", "WithMachineName", "WithThreadId"],
    "Properties": {
      "Application": "VB6ToDotNetCore"
    }
  }
}
```

## Rollback Procedures

### Application Rollback
```bash
# Stop application
docker-compose down

# Restore previous version
docker tag vb6-app:previous vb6-app:latest

# Start application
docker-compose up -d

# Verify application health
curl http://localhost/health
```

### Database Rollback
```sql
-- Database rollback script
USE [VB6ToDotNetCore]
GO

-- Restore from backup
RESTORE DATABASE [VB6ToDotNetCore]
FROM DISK = 'C:\Backups\VB6ToDotNetCore_PreMigration.bak'
WITH REPLACE;

-- Or rollback specific changes
-- (Depends on migration strategy)
```

### Complete Environment Rollback
```yaml
# Rollback pipeline
- stage: Rollback
  jobs:
  - job: RollbackApplication
    steps:
    - task: AzureCLI@2
      inputs:
        scriptType: 'pscore'
        inlineScript: |
          # Swap back to previous slot
          az webapp deployment slot swap --name vb6-app-prod --resource-group rg-vb6-app --slot previous

  - job: RollbackDatabase
    steps:
    - task: SqlAzureDacpacDeployment@1
      inputs:
        azureSubscription: 'Azure-Subscription'
        serverName: 'sql-prod.database.windows.net'
        databaseName: 'VB6ToDotNetCore'
        sqlUsername: '$(SqlUsername)'
        sqlPassword: '$(SqlPassword)'
        deployType: 'DacpacTask'
        dacpacFile: 'database/rollback.dacpac'
```

This comprehensive build and deployment guide ensures consistent, reliable, and automated delivery of the .NET Core 10 application across all environments.