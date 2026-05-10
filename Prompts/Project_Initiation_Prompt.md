# VB6 to .NET Core 10 Migration Prompts

## Project Initiation Prompt

**Project**: VB6 to .NET Core 10 Migration
**Date**: May 10, 2026
**Objective**: Complete migration of legacy VB6 application to modern .NET Core 10 framework

### Requirements
- Convert VB6 desktop application to .NET Core 10
- Maintain all existing business functionality
- Improve performance and security
- Enable cross-platform deployment
- Implement modern development practices

### Scope
- Full codebase analysis and documentation
- Component-by-component migration
- Database schema and data migration
- UI modernization
- API development for integrations
- Comprehensive testing strategy
- Production deployment with monitoring

### Success Criteria
- 100% functional equivalence
- 95%+ code coverage
- Performance improvement over VB6
- Zero security vulnerabilities
- Successful production deployment

## Technical Requirements

### Source Application Details
- **Language**: Visual Basic 6.0
- **UI Framework**: VB6 Forms
- **Database**: SQL Server 2000/2005
- **External Dependencies**: Crystal Reports, Office Automation
- **Deployment**: Desktop application

### Target Architecture
- **Framework**: .NET Core 10
- **UI**: WPF/Windows Forms with MVVM
- **Database**: SQL Server 2022 with EF Core
- **API**: ASP.NET Core Web API
- **Deployment**: Docker containers with Kubernetes
- **CI/CD**: GitHub Actions/Azure DevOps

### Quality Standards
- Code coverage: 95% minimum
- Performance: 500ms response time for 95% of requests
- Security: OWASP Top 10 compliance
- Documentation: 100% API documentation
- Accessibility: WCAG 2.1 AA compliance

## Migration Phases

### Phase 1: Analysis and Planning
- Codebase assessment
- Component inventory
- Risk analysis
- Migration strategy definition

### Phase 2: Infrastructure Setup
- .NET Core project structure
- Development environment setup
- CI/CD pipeline configuration
- Testing framework implementation

### Phase 3: Data Layer Migration
- Database schema migration
- EF Core implementation
- Data access layer conversion
- Stored procedure migration

### Phase 4: Business Logic Conversion
- Service layer implementation
- Validation logic migration
- Business rules extraction
- Error handling modernization

### Phase 5: UI Migration
- WPF application development
- MVVM pattern implementation
- Control mapping and conversion
- Responsive design implementation

### Phase 6: API Development
- REST API design
- Authentication and authorization
- Integration endpoints
- API documentation

### Phase 7: Testing and Quality Assurance
- Unit testing implementation
- Integration testing
- Performance testing
- Security testing

### Phase 8: Deployment and Monitoring
- Containerization with Docker
- Kubernetes orchestration
- Monitoring and logging setup
- Production deployment

## Constraints and Assumptions

### Technical Constraints
- Must maintain backward compatibility with existing data
- Limited access to original VB6 development environment
- Third-party component dependencies
- Legacy database structure preservation

### Business Constraints
- Zero downtime during migration
- User training and adoption
- Regulatory compliance requirements
- Budget and timeline limitations

### Assumptions
- Access to source code and database backups
- Availability of original developers for knowledge transfer
- Modern development infrastructure available
- Cloud hosting environment accessible

## Deliverables

### Documentation
- Complete migration guide
- API documentation
- User manuals
- Deployment guides

### Code Artifacts
- .NET Core source code
- Docker configurations
- Kubernetes manifests
- CI/CD pipelines

### Testing Assets
- Unit test suites
- Integration test suites
- Performance test scripts
- Security test reports

### Deployment Assets
- Container images
- Infrastructure as code
- Monitoring configurations
- Backup and recovery procedures

## Risk Assessment

### High Risk Items
- Complex business logic conversion
- Third-party component compatibility
- Database migration integrity
- User interface modernization

### Mitigation Strategies
- Incremental migration approach
- Comprehensive testing strategy
- Parallel system operation during transition
- Extensive user acceptance testing

## Communication Plan

### Stakeholders
- Development Team
- Business Users
- IT Operations
- Management
- External Vendors

### Communication Methods
- Weekly status reports
- Monthly steering committee meetings
- Technical documentation repository
- User training sessions
- Change management notifications

## Success Metrics

### Technical Metrics
- Code migration completion percentage
- Test coverage percentage
- Performance benchmark results
- Security scan results

### Business Metrics
- User acceptance test results
- System availability percentage
- Error rate reduction
- User productivity improvement

### Quality Metrics
- Defect density
- Mean time to resolution
- Customer satisfaction scores
- Compliance audit results