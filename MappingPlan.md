# Mapping Plan

This document outlines the detailed mapping strategy for converting VB6 components to .NET Core equivalents. The mapping covers UI, data access, business logic, and infrastructure components.

## UI Migration Strategy

### Form Mapping
| VB6 Component | .NET Core Equivalent | Migration Approach |
|---------------|---------------------|-------------------|
| VB6 Form | Windows Forms Form / WPF Window | Direct conversion with layout adjustments |
| MDI Parent | WPF MDI or TabControl | Refactor to modern navigation patterns |
| Modal Dialog | Window with ShowDialog() | Preserve modal behavior |
| Modeless Dialog | Window with Show() | Maintain multiple window support |

### Control Mapping
| VB6 Control | .NET Core Control | Notes |
|-------------|------------------|-------|
| TextBox | TextBox | Direct mapping |
| CommandButton | Button | Direct mapping |
| Label | Label | Direct mapping |
| ComboBox | ComboBox | Direct mapping |
| ListBox | ListBox | Direct mapping |
| CheckBox | CheckBox | Direct mapping |
| OptionButton | RadioButton | Group RadioButtons |
| PictureBox | Image | Direct mapping |
| Frame | GroupBox | Direct mapping |
| MSFlexGrid | DataGridView | Enhanced functionality |
| Calendar | DatePicker | Improved UX |
| ProgressBar | ProgressBar | Direct mapping |
| StatusBar | StatusBar | Direct mapping |

### ActiveX Control Replacements
| VB6 ActiveX | .NET Core Alternative | Migration Effort |
|-------------|----------------------|-----------------|
| MSFlexGrid | DataGrid | Medium |
| FarPoint Spread | Syncfusion Grid | High |
| Crystal Reports | RDLC Reports | Medium |
| Calendar Control | WPF Calendar | Low |

## Data Access Migration

### ADO to Entity Framework Core
| VB6 ADO Pattern | .NET Core EF Core | Migration Notes |
|----------------|------------------|----------------|
| ADODB.Connection | DbContext | Context class |
| ADODB.Recordset | DbSet<TEntity> | Entity collections |
| Connection.Open() | context.Database.OpenConnection() | Async methods |
| Recordset.Open() | context.Entities.ToList() | LINQ queries |
| rs.MoveNext() | foreach loop | Iterator pattern |
| rs.Fields("field") | entity.Field | Strongly typed |

### Database Provider Mapping
| Database | VB6 Provider | .NET Core Provider |
|----------|--------------|-------------------|
| SQL Server | SQLOLEDB | Microsoft.Data.SqlClient |
| Oracle | MSDAORA | Oracle.ManagedDataAccess.Core |
| Access | Microsoft.Jet.OLEDB.4.0 | System.Data.OleDb (limited) |
| MySQL | MySQL ODBC | MySqlConnector |

### Query Migration Examples
```vb
' VB6 ADO
Dim rs As ADODB.Recordset
Set rs = New ADODB.Recordset
rs.Open "SELECT * FROM Customers", conn
```

```csharp
// .NET Core EF Core
var customers = await context.Customers.ToListAsync();
```

## Business Logic Migration

### Module to Class Library
| VB6 Module | .NET Core Class | Organization |
|------------|----------------|-------------|
| modBusinessLogic | BusinessLogicService | Service layer |
| modValidation | ValidationService | Service layer |
| modCommon | CommonUtilities | Shared library |
| modConstants | Constants | Static class |

### Data Type Mapping
| VB6 Type | .NET Core Type | Notes |
|----------|----------------|-------|
| Integer | int | Direct |
| Long | long | Direct |
| Single | float | Direct |
| Double | double | Direct |
| String | string | Direct |
| Variant | object | Avoid, use specific types |
| Date | DateTime | Direct |
| Boolean | bool | Direct |
| Byte | byte | Direct |

### Function Migration
```vb
' VB6 Function
Public Function CalculateTotal(price As Double, quantity As Integer) As Double
    CalculateTotal = price * quantity
End Function
```

```csharp
// .NET Core Method
public double CalculateTotal(double price, int quantity)
{
    return price * quantity;
}
```

## API and System Integration

### Windows API Migration
| VB6 API Call | .NET Core Equivalent | Approach |
|--------------|---------------------|----------|
| Declare Function | DllImport | P/Invoke |
| user32.dll functions | System.Windows.Forms | Managed wrappers |
| kernel32.dll | System.Runtime.InteropServices | P/Invoke |
| File operations | System.IO | Managed classes |

### COM Object Replacements
| VB6 COM | .NET Core Alternative | Migration Path |
|---------|----------------------|----------------|
| CDO.Message | System.Net.Mail | SmtpClient |
| Scripting.FSO | System.IO | File/FileInfo |
| MSXML2.DOMDocument | System.Xml | XmlDocument |
| ADODB.Stream | System.IO.Stream | Stream classes |

## Error Handling Migration

### VB6 Error Handling
```vb
On Error GoTo ErrorHandler
' code
Exit Sub
ErrorHandler:
    MsgBox "Error: " & Err.Description
```

### .NET Core Exception Handling
```csharp
try
{
    // code
}
catch (Exception ex)
{
    MessageBox.Show($"Error: {ex.Message}");
}
```

## Configuration Migration

### Settings Migration
| VB6 Method | .NET Core Method | File Format |
|------------|------------------|-------------|
| App.Path | AppDomain.CurrentDomain.BaseDirectory | N/A |
| GetSetting | ConfigurationManager | app.config |
| SaveSetting | ConfigurationManager | app.config |
| Registry | ProtectedConfiguration | Encrypted config |

## Security Enhancements

### Authentication Migration
- **VB6**: Basic username/password
- **.NET Core**: ASP.NET Core Identity or custom JWT

### Data Protection
- **Encryption**: Implement AES encryption
- **Secure Storage**: Use Data Protection API

## Testing Framework Mapping

### Unit Testing
- **VB6**: Manual testing
- **.NET Core**: xUnit, NUnit, or MSTest

### UI Testing
- **VB6**: Manual
- **.NET Core**: Selenium WebDriver or WinAppDriver

## Build and Deployment

### Project Structure
```
Solution/
├── Core/ (Business logic)
├── UI/ (WPF/WinForms)
├── Data/ (EF Core)
├── Tests/ (xUnit)
└── WebAPI/ (ASP.NET Core Web API)
```

### Build Tools
- **VB6**: Visual Studio 6.0
- **.NET Core**: dotnet CLI, MSBuild

### Deployment
- **VB6**: Setup.exe, MSI
- **.NET Core**: ClickOnce, MSI, Docker containers

## Migration Phases

1. **Phase 1**: Infrastructure setup (.NET Core project structure)
2. **Phase 2**: Data access layer (EF Core migration)
3. **Phase 3**: Business logic conversion
4. **Phase 4**: UI migration
5. **Phase 5**: Testing and validation
6. **Phase 6**: Deployment and optimization