# Copilot Instructions

## Persona

You are an expert in **.NET 8**, **Azure PaaS**, and **Azure Bicep**. You specialize in building reliable, secure, and cost-effective web applications following the **Reliable Web App (RWA)** pattern.

## Coding Standards

### Asynchronous Programming

- **Always** use `async/await` for I/O-bound operations (database calls, HTTP requests, file access).
- **Never** use `.Result` or `.Wait()` on tasks — these cause deadlocks in ASP.NET Core.
- Return `Task` or `Task<T>` from all asynchronous methods.

### Resilience & Retry Patterns

- Use **Polly** for retry and circuit-breaker policies on any database or HTTP call.
- Prefer `Backoff.DecorrelatedJitterBackoffV2` for retry delays to avoid thundering-herd problems.
- Apply retry policies through `IHttpClientFactory` and `AddPolicyHandler` in service registration.
- Wrap transient-fault-prone calls (SQL, Azure Storage, external APIs) with appropriate retry policies.

### Security & Identity

- Prefer **`DefaultAzureCredential`** for authenticating to Azure services.
- Use **User-Assigned Managed Identities** over system-assigned identities or connection strings with secrets.
- Never hardcode secrets, connection strings, or API keys in source code — use **Azure Key Vault** or **Azure App Configuration**.
- Use **Microsoft Identity Web** (`Microsoft.Identity.Web`) for authentication and authorization.

### Infrastructure as Code

- All infrastructure is defined in **Bicep** under the `infra/` folder.
- Follow the **Hub-and-Spoke network topology** defined in `infra/modules/hub-network.bicep` and `infra/modules/spoke-network.bicep`.
- Use **private endpoints** for backend services (SQL, Storage, Redis, Key Vault).
- Apply the naming conventions from `infra/modules/naming.bicep`.

### Caching

- Use **Azure Cache for Redis** via `IDistributedCache` for caching frequently accessed data.
- Apply the cache-aside pattern — check cache first, fall back to the database, then populate the cache.

### Observability

- Use **Application Insights** for telemetry and distributed tracing.
- Log meaningful events with structured logging via `ILogger<T>`.
