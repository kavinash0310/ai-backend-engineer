# LLM application security

## The model is not an access-control boundary

A language model receives instructions and content as tokens. It can confuse trusted instructions with untrusted text embedded in a user message, retrieved document, image, or tool result. Prompt injection tries to alter model behavior, reveal information, or misuse available tools. Because the model is probabilistic, a prompt alone is not a reliable security control.

Enforce permissions in deterministic code. Before retrieval, restrict the search corpus to resources the caller may access. Before executing a tool, validate its schema, check caller authorization, and enforce limits. A model should never receive a database superuser credential or unrestricted shell access.

## Indirect injection in RAG

A retrieved document may contain malicious text such as instructions to reveal another user's files. The model can encounter that text even though the user did not type it. Treat retrieved passages and tool outputs as untrusted data. Delimit and label them, but assume the model may still be influenced.

The decisive control is the backend: retrieval must enforce tenant scope; the model can only call allowlisted tools; each tool rechecks permissions; secrets are not placed in context; high-impact actions require explicit confirmation. If a document assistant can only answer questions, do not give it write tools.

## File and URL handling

Uploaded files can contain oversized content, malformed structures, macros, embedded instructions, or parser exploits. Restrict types and size, scan or safely parse in an isolated process, store outside the executable web root, and apply access checks when downloading. For URL ingestion, SSRF controls must prevent requests to internal addresses and cloud metadata.

Avoid rendering model output as trusted HTML. Escape content or use a safe renderer. Validate structured output against a strict schema, reject unknown fields, and do not execute generated code. Protect against cross-site scripting, SQL injection, and command injection in the ordinary application layer too.

## Abuse, privacy, and cost

Apply per-user and per-tenant rate limits, token budgets, maximum context length, upload limits, and concurrency caps. Otherwise automated abuse can create high provider costs or exhaust the service. Add quotas that reflect plan and resource cost, and return clear retry information.

Prompt and completion logs may contain private data. Minimize retention, redact or hash identifiers when possible, restrict access, and document why logging is needed. Do not put secrets in prompts. Record model name, token count, latency, and failure category without storing raw content unless there is a justified, protected need.

## Threat-model exercise

For a RAG assistant with access to private company folders, list assets, callers, trust boundaries, attacker goals, controls, and tests. Include:
- direct prompt injection from a user;
- indirect instructions inside a retrieved file;
- attempts to retrieve another tenant's document;
- a malicious URL that targets an internal service;
- malformed tool arguments and oversized requests;
- provider outage and accidental logging of private text.

A safe design verifies access on retrieval and again before any action, denies by default, and has a human review path for consequential changes.

Questions: Does a system prompt stop indirect injection? No. What must prevent cross-tenant retrieval? Application-level authorization applied before model context is assembled. Why bound tokens and uploads? They control abuse, latency, and cost.

Further reading: [OWASP Top 10 for LLM Applications](https://owasp.org/www-project-top-10-for-large-language-model-applications/) and [OWASP API Security Top 10](https://api-security.owasp.org/editions/2023/en/0x11-t10/).
