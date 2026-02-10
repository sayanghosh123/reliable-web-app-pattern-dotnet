# AGENTS.md — Repository Guide for Autonomous Agents

## Overview

This repository implements the **Reliable Web App (RWA) pattern** for .NET on Azure. It contains a production-ready reference architecture with a frontend web application, a backend API, shared models, and full Azure infrastructure defined in Bicep.

## Tech Stack

| Layer            | Technology                                                    |
| ---------------- | ------------------------------------------------------------- |
| Runtime          | .NET 8 (C#)                                                   |
| Frontend         | ASP.NET Core MVC with Razor Views                             |
| Backend API      | ASP.NET Core Web API                                          |
| ORM              | Entity Framework Core 8                                       |
| Database         | Azure SQL Database                                            |
| Caching          | Azure Cache for Redis (`IDistributedCache`)                   |
| Search           | Azure AI Search                                               |
| Storage          | Azure Blob Storage                                            |
| Identity         | Microsoft Entra ID via Microsoft Identity Web                 |
| Resilience       | Polly (retry, circuit breaker, jitter backoff)                |
| Infrastructure   | Azure Bicep (Hub-and-Spoke topology)                          |
| Deployment       | Azure Developer CLI (`azd`)                                   |
| Observability    | Application Insights + Log Analytics                          |
| Security         | Azure Key Vault, Azure Front Door with WAF, Private Endpoints |

## Repository Structure

```text
.
├── src/                                    # Application source code
│   ├── Relecloud.sln                       # .NET solution file
│   ├── global.json                         # .NET SDK version (8.0)
│   ├── Relecloud.Web.CallCenter/           # Frontend MVC application (ASP.NET Core)
│   ├── Relecloud.Web.CallCenter.Api/       # Backend Web API (ASP.NET Core)
│   └── Relecloud.Web.Models/              # Shared models and service interfaces
│
├── infra/                                  # Azure Bicep infrastructure-as-code
│   ├── main.bicep                          # Primary orchestration template
│   ├── main.parameters.json                # Parameter defaults
│   ├── core/                               # Core resource modules (network, database, security, etc.)
│   ├── modules/                            # Composed modules (hub-network, spoke-network, application-resources)
│   ├── scripts/                            # Deployment lifecycle scripts (pre/post provision, deploy, down)
│   └── types/                              # Custom Bicep user-defined types
│
├── testscripts/                            # Deployment validation and test scripts
│   ├── validate-deployment.ps1             # Primary deployment health-check script
│   ├── validate-deployment.sh              # Linux version of validation script
│   ├── setup.ps1                           # Provision infrastructure with deployment options
│   └── cleanup.ps1                         # Clean up provisioned environment
│
├── workshop/                               # Workshop and training materials
├── assets/                                 # Diagrams, images, and SLA documents
├── azure.yaml                              # Azure Developer CLI orchestration config
├── .azdo/                                  # Azure DevOps pipeline definitions
├── .github/                                # GitHub Actions workflows and issue templates
└── .devcontainer/                          # Dev container configuration
```

## Common Commands

### Deploy the Application

```bash
azd up
```

Provisions all Azure infrastructure and deploys both the frontend and API services. The `azure.yaml` file orchestrates lifecycle hooks for pre-provisioning, post-provisioning, pre-deploy, and post-deploy steps.

### Tear Down the Environment

```bash
azd down --purge
```

Removes all provisioned Azure resources. The `--purge` flag ensures soft-deleted resources (e.g., Key Vault) are fully purged.

### Build the Solution

```bash
dotnet build src/Relecloud.sln
```

### Run Tests

```bash
dotnet test src/Relecloud.sln
```

### Validate a Deployment

After deploying, run the health-check script to verify the environment:

**PowerShell:**

```powershell
./testscripts/validate-deployment.ps1 -ResourceGroupName <your-resource-group-name>
```

**Bash:**

```bash
./testscripts/validate-deployment.sh <your-resource-group-name>
```

This script checks that the resource group exists, validates App Service settings, verifies App Configuration connectivity, and provides troubleshooting recommendations for known issues.

### Provision Only (No Code Deploy)

```bash
azd provision
```

### Deploy Code Only (No Infrastructure Changes)

```bash
azd deploy
```

## Key Architecture Decisions

- **Hub-and-Spoke Networking:** A central hub VNet hosts shared resources (Firewall, Bastion, DDoS Protection) while the spoke VNet isolates application workloads with private endpoints.
- **Retry with Jitter:** HTTP calls use Polly's `DecorrelatedJitterBackoffV2` strategy to avoid thundering-herd failures.
- **Managed Identity:** Services authenticate to Azure resources using `DefaultAzureCredential` and User-Assigned Managed Identities — no secrets in code.
- **Cache-Aside Pattern:** Azure Cache for Redis is used to reduce database load for frequently accessed data.
