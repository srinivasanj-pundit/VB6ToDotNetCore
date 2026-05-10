# VB6 to .NET Core 10 Conversion Project

You are tasked with converting a legacy VB6 application into .NET Core 10.  
Create **Markdown files** as agents for each phase of the migration.  
The files must be placed in the directory:  
`D:\GitHubRepository\VB6ToDotNetCore`

## Requirements

For each phase, generate a separate `.md` file with detailed content:

1. **Analysis.md**  
   - Perform full analysis of the VB6 application.  
   - Document architecture, dependencies, external libraries, and business logic.

2. **Components.md**  
   - Identify every component in the VB6 source code.  
   - Include forms, modules, classes, ActiveX controls, COM objects, APIs.

3. **ComponentDetails.md**  
   - List component details and considerations.  
   - Highlight compatibility issues, deprecated features, and modernization needs.

4. **MappingPlan.md**  
   - Provide detailed mapping of VB6 components to .NET Core equivalents.  
   - Include UI migration (WinForms/WPF/Blazor), data access (ADO → EF Core), and API replacements.

5. **ConversionProgress.md**  
   - Track full conversion progress.  
   - Document milestones, completed modules, pending tasks.

6. **IssuesAndResolutions.md**  
   - List all issues and TODO items.  
   - Categorize severity: [Low, Medium, High, Critical].  
   - Provide recommended resolutions for each.

7. **Testing.md**  
   - Write xUnit test cases.  
   - Cover unit tests for individual components, integration testing, regression testing.  
   - Ensure code coverage up to 97%.  
   - Understand business logic and write functional test methods.

8. **BuildAndSecurity.md**  
   - Build each project.  
   - Assess for code vulnerabilities.  
   - Recommend resolutions (e.g., dependency updates, secure coding practices).

9. **Dockerization.md**  
   - Write Docker containerization and orchestration files.  
   - Include Dockerfile, docker-compose.yml, and Kubernetes manifests if needed.

## Output Format

- Each file must be **Markdown** (`.md`).  
- Use headings, bullet points, and tables for clarity.  
- Provide actionable details, not just summaries.  
- Ensure consistency across all files.

## Directory Structure

<sandbox>\Repository\VB6ToDotNetCore
│
├── Analysis.md
├── Components.md
├── ComponentDetails.md
├── MappingPlan.md
├── ConversionProgress.md
├── IssuesAndResolutions.md
├── Testing.md
├── BuildAndSecurity.md
└── Dockerization.md

---

## Example Agent File (IssuesAndResolutions.md)

```markdown
# Issues and Resolutions

## Critical
- **COM Interop Failure**  
  Resolution: Replace with .NET Core compatible NuGet packages.

## High
- **ADO Recordset Usage**  
  Resolution: Migrate to Entity Framework Core.

## Medium
- **VB6 String Functions**  
  Resolution: Map to .NET Core System.String methods.

## Low
- **UI Layout Differences**  
  Resolution: Adjust WinForms/WPF layouts manually.
