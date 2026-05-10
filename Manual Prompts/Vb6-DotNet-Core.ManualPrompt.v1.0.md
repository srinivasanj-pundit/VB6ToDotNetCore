Write github copilot prompt to give full perfect response for vb6 to .net core 10 conversion considring the below 

to Convert VB6 application source code to .NET Core 10. Write markdown files as agents for each phase: 
1. Perform full analysis
2. Identify every component
3. List component details and things to consider
4. Detail all component mapping plan
5. Full conversion progress
6. List all the issues, todo - categorize [Low, Medium, High, Critical], give corresponding resolutions recommendations
7. Create xUnit test cases covering unit test individual component, integration testing, regression testing, code coverage upto 97%, understand business logics and write functional test method
8. Build each project, assess for code vulerabilities, recommend the resolution
9. Write Docker containerization and orchestration files
Create the files in the below directory
"D:\GitHubRepository\VB6ToDotNetCore"

-----------
act as a VB6 SDK and .NET CORE 10 SDK

To convert a vb6 complex application to .Net core 10
Considering multiple design frameworks and patterns 

forms - can be angular.js and typescript
forms - can be asp.net core razor web page
forms - can be asp.net core blazor 

Optionally, a .Net core API as microservices

business logics - appropriate and segregates class libraries for data models, validations, business logics, logging, authentication, authorization, exception handling, routing, automapper,  configurations, 

ADO.NET - for data access layer, consdering a segregated class for each table definition with relationships
Entity Framework core - for data access layer

Dockers for containerization

Kubernetes for orchestrations

Azure Devops for CI/CD pipelines

additionally, identify vb6 external references, activex controls, com components and recommend and convert to appropriate .net core same feature

build the application
