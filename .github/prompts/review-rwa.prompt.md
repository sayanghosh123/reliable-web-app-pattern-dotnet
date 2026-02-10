# Review Code Against Reliable Web App (RWA) Best Practices

Review the selected code for adherence to the **Reliable Web App (RWA)** pattern. Evaluate the code against the three pillars below and provide actionable feedback.

## 1. Reliability

- Is there retry logic (e.g., Polly) on database calls, HTTP calls, or Azure service interactions?
- Are circuit-breaker policies applied to prevent cascading failures?
- Does the code use `async/await` correctly, avoiding `.Result` or `.Wait()` which can cause deadlocks?
- Are transient faults handled gracefully with appropriate backoff strategies?

## 2. Security

- Are there any hardcoded secrets, connection strings, or API keys?
- Does the code use `DefaultAzureCredential` or Managed Identity instead of stored credentials?
- Is user input validated and sanitized before use?
- Are authentication and authorization properly enforced (e.g., `[Authorize]` attributes, role checks)?

## 3. Performance

- Are there N+1 query patterns (e.g., a database call inside a loop)?
- Is caching used for frequently accessed, rarely changing data (cache-aside pattern)?
- Are database queries projected (using `.Select()`) instead of fetching full entities when only a subset of fields is needed?
- Are `IHttpClientFactory` and typed HTTP clients used instead of manually instantiated `HttpClient`?

## Expected Output

Provide your findings as:

- A **bulleted list of issues** found, grouped by pillar (Reliability, Security, Performance).
- For each issue, include the relevant line or code snippet and a brief explanation.
- A **refactored code block** that addresses the identified issues, with comments explaining each change.
