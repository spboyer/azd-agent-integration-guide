# AGENTS.md for Azure Developer CLI (azd) Projects

This document provides guidance for AI coding agents working on Azure Developer CLI (azd) projects. It complements the README.md by focusing on technical requirements, project structure, and deployment specifics that agents need to understand.

## Project Overview

This is an Azure Developer CLI (azd) project designed for deploying cloud-native applications to Azure. The azd tool simplifies the process of creating, provisioning, and deploying applications by providing a standardized project structure and workflow.

## Azure App Project Standards

This project follows the Azure app project definition standards for integrating AI agents into Azure Developer CLI templates, focusing on consistent implementation, security, validation, and adherence to Azure best practices.

### General Requirements

All templates must comply with the [Template Framework Definition of Done](https://github.com/Azure-Samples/azd-template-artifacts/blob/main/docs/development-guidelines/definition-of-done.md).

### 1. Agent Configuration

**Configuration Standards:**
- **AGENTS.md**: Universal standard for agent documentation (this file)
- **Optional**: AGENTS.yml or AGENTS.json for deterministic configuration approach
- **Configuration source**: All agent configuration must be externalized, never embedded in code
- **Delivery**: Configuration delivered via Azure App Configuration or GitHub secrets
- **Environment isolation**: Each environment (dev, staging, prod) must have clearly separated configuration stores, following [Azure Developer CLI environments](https://learn.microsoft.com/azure/developer/azure-developer-cli/work-with-environments)

**Naming Conventions:**
- Follow standard Azure naming rules
- Use resource prefixes aligned to organizational taxonomy
- Align with organizational high-level governance when available

**Secrets Handling:**
- Agents must never store secrets in code or configuration files
- Retrieve sensitive values securely via User-Assigned Managed Identity (UAMI)
- Use application settings, GitHub secrets, or Key Vault for secret management

**Observability:**
- Emit logs, metrics, and traces in standardized format (OpenTelemetry recommended)
- Ship telemetry to Azure Monitor
- Include proper observability hooks in all agent implementations

### 2. Security Requirements

**Authentication:**
- All agent authentication to Azure services must use User-Assigned Managed Identity (UAMI)
- No service principals, connection strings, or static secrets
- Follow zero-trust security principles

**Authorization:**
- Access provisioned on least-privilege basis through Azure RBAC
- Identities scoped only to required resources (Key Vault, Storage, Event Hub, etc.)
- Regular access reviews and permission auditing

**Data Protection:**
- All data in transit must use TLS 1.2+
- At-rest encryption enabled for all resources using Microsoft-managed keys
- Customer-Managed Keys (CMK) when business requirements mandate
- Data residency and sovereignty considerations

**Production Compliance:**
- Enterprise-grade templates must adhere to compliance requirements (GDPR, HIPAA, SOC2)
- Logging must exclude PII unless properly redacted
- Security scanning and vulnerability assessments
- Regular compliance audits and documentation

### 3. Development Workflows

**Source Control:**
- All agent code in organizational Git repositories
- Enforced branch protection and review policies
- Compliance with Microsoft OSPO guidelines
- Proper commit signing and audit trails

**CI/CD Pipelines:**
- Deploy using [Azure Developer CLI generated pipeline configuration](https://learn.microsoft.com/azure/developer/azure-developer-cli/pipeline-github-actions)
- Automated testing and validation gates
- Environment promotion strategies
- Rollback capabilities and disaster recovery

**Dependency Management:**
- Vulnerability scanning before deployment
- Automated dependency updates and security patches
- License compliance checking
- Supply chain security validation

**Versioning:**
- Follow template CalVer versioning (in development)
- Semantic versioning for application components
- Release notes and change documentation
- Backward compatibility considerations

**Testing Requirements:**
- Minimum coverage requirements for unit, integration, and e2e tests
- CI enforcement before main branch merges
- Performance and load testing
- Security testing integration

### 4. Resources and Infrastructure as Code Configuration

**Infrastructure as Code:**
- All resources provisioned via Bicep (or Terraform for Terraform samples)
- Services correctly configured in `azure.yaml` file
- Infrastructure validation and testing
- Environment-specific configurations

**Resource Governance:**
- Tags applied to all resources as best practice
- Resource groups following standard structure for lifecycle management
- Cost management and optimization
- Resource naming conventions enforcement

**Compute Requirements:**
- Run on Azure Developer CLI + Bicep supported platforms:
  - Azure Functions
  - Azure App Service
  - Azure Container Apps (ACA)
  - Azure Kubernetes Service (AKS)
- Autoscaling configured for production-ready templates
- Right-sizing and performance optimization

**Networking (Production-Ready Templates):**
- Agents run inside secured VNets
- Outbound access restricted via Private Endpoints or Azure Firewall
- Network segmentation and micro-segmentation
- DDoS protection and WAF implementation

### 5. Validation and Testing

**Pre-Deployment Checks:**
- Static analysis (linting, SAST)
- Infrastructure as Code validation (Bicep linter)
- Secrets scanning to prevent credential leaks
- Dependency vulnerability scanning

**Integration & Performance Testing:**
- Performance/load benchmarks for scaling decisions
- Integration testing across all components
- API testing and contract validation
- Database and storage performance testing

**Security Testing:**
- Dependency vulnerability scanning
- Penetration testing for production templates
- Security code analysis
- Container image scanning

**Post-Deployment Validation:**
- Health probes confirm readiness
- Telemetry verification in Azure Monitor/Log Analytics
- End-to-end functional testing
- Performance monitoring and alerting

### 6. Governance and Lifecycle Management

**Ownership:**
- Every agent template must have documented owner
- Team responsibility and escalation contacts
- Template validation notification contact
- Clear support and maintenance responsibilities

**Lifecycle Management:**
- Deprecated or unused agents must be archived
- Removal from template collections when obsolete
- Version lifecycle and support policies
- Migration paths for deprecated features

**Documentation Requirements:**
- Comprehensive README with all required headings
- Architecture diagrams and decision records
- Troubleshooting guides and runbooks
- API documentation and examples

## Prerequisites

- [Azure Developer CLI (azd)](https://aka.ms/azure-dev/install) version 1.5.0 or later
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) (automatically installed with azd)
- An Azure subscription with appropriate permissions
- [Docker](https://docs.docker.com/get-docker/) (if using containerized applications)

## Essential Commands

### Development Workflow
- `azd init` - Initialize a new azd project (only when starting from scratch)
- `azd auth login` - Authenticate with Azure
- `azd env new [environment-name]` - Create a new environment
- `azd env set [key] [value]` - Set environment variables
- `azd provision` - Provision Azure resources without deploying code
- `azd deploy` - Deploy application code to existing resources
- `azd up` - Provision resources AND deploy application (full deployment)
- `azd down` - Destroy all Azure resources
- `azd env list` - List all environments
- `azd env select [environment-name]` - Switch between environments

### Monitoring and Troubleshooting
- `azd monitor` - Open Azure Monitor dashboard
- `azd show` - Display current deployment information
- `azd env get-values` - Show all environment variables

## Infrastructure Guidelines

### Bicep Templates (Recommended)

1. **Main Template Structure**:
   - Use `main.bicep` as the entry point
   - Define target scope (usually `resourceGroup`)
   - Include resource naming conventions with unique suffixes
   - Tag resources with `azd-env-name` for environment tracking
   - **Security**: Implement User-Assigned Managed Identity (UAMI) for all service authentication
   - **Governance**: Apply consistent resource tags for lifecycle management

2. **Required Parameters**:
   ```bicep
   @minLength(1)
   @maxLength(64)
   @description('Name of the environment')
   param environmentName string
   
   @minLength(1)
   @description('Primary location for all resources')
   param location string = resourceGroup().location
   
   @description('User-assigned managed identity for secure authentication')
   param managedIdentityName string = '${resourceToken}-identity'
   ```

3. **Security-First Resource Implementation**:
   ```bicep
   // User-Assigned Managed Identity (Required)
   resource managedIdentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2023-01-31' = {
     name: managedIdentityName
     location: location
     tags: {
       'azd-env-name': environmentName
       'azd-service-name': 'identity'
     }
   }
   
   // Storage Account with security best practices
   resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
     name: '${resourceToken}storage'
     location: location
     identity: {
       type: 'UserAssigned'
       userAssignedIdentities: {
         '${managedIdentity.id}': {}
       }
     }
     properties: {
       // Security: TLS 1.2+ enforcement
       minimumTlsVersion: 'TLS1_2'
       supportsHttpsTrafficOnly: true
       // Security: Disable public access
       allowBlobPublicAccess: false
       // Security: Enable encryption
       encryption: {
         services: {
           blob: { enabled: true }
           file: { enabled: true }
         }
         keySource: 'Microsoft.Storage'
       }
     }
     tags: {
       'azd-env-name': environmentName
       'azd-service-name': 'storage'
     }
   }
   ```

For security, infrastructure, and governance details, see the complete AGENTS.md file in the repository.

---

This AGENTS.md file should be treated as living documentation. Update it when project structure, requirements, or deployment processes change.