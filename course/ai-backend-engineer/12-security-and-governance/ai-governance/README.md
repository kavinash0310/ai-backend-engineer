# AI Security and Governance

Traditional security remains mandatory: authentication, authorization, OAuth2/JWT validation, RBAC/ABAC, TLS, SQL injection, XSS/CSRF/SSRF, secrets, encryption and audit.

AI-specific threats include direct/indirect prompt injection, jailbreaks, sensitive-data disclosure, insecure output handling, RAG poisoning, malicious documents, excessive agent authority, tool abuse, untrusted tool results and model/supply-chain risk.

Apply controls outside model instructions: trusted identity, tenant-aware queries, allowlists, schema/business validation, quotas, sandboxing, data minimization, retention, approvals and audit. Map mitigations to the current [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/) and recheck its edition/date.


## Detailed lesson

- [AI application threat model](./01-ai-application-threat-model.md) — assets, trust boundaries, prompt injection, data disclosure, tool abuse, supply chain and incident response.
- Map findings to the current [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/).
