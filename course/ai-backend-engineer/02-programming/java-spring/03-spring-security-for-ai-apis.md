# 3. Spring Security for AI APIs

Model-backed features can expose private information through ordinary API flaws and generated behavior. Security starts with assets, identities, permissions, and trust boundaries—not with a prompt telling the model to protect secrets.

## 3.1 Threat model

List assets (documents, prompts, outputs, embeddings, credentials), actors (anonymous user, tenant user, administrator, compromised service), entry points (API, upload, webhook, tool, queue message), and trust boundaries (browser/API, API/database, service/provider).

~~~mermaid
flowchart LR
  U[User input and files] --> API[Spring API]
  API --> AUTH[OAuth2 resource server]
  API --> DB[(Tenant data)]
  API --> RET[Retriever with ACL filters]
  RET --> CTX[Selected context]
  CTX --> LLM[External or hosted model]
  LLM --> OUT[Untrusted generated output]
  OUT --> API
  API -->|allowlist and authorization| TOOL[Approved tool]
  TOOL --> SYS[Business system]
~~~

Retrieved documents and generated text are untrusted. The system must enforce permissions deterministically.

## 3.2 Authentication, authorization, identity

- **Authentication** answers who or what made the request.
- **Authorization** answers whether that identity may perform this action on this resource.
- **Tenant isolation** prevents one customer from reading or modifying another customer's records.

For JWT resource servers, validate signature with trusted issuer keys, and check issuer, audience, expiry, not-before, and scopes/claims. A valid signature alone does not mean the token was intended for your API. Prefer supported OAuth2/OIDC libraries; never implement token parsing or signature verification yourself.

Pass trusted actor and tenant identity into the use case. Query by resource ID plus ownership/tenant constraint, or check a policy before returning data. Random IDs reduce guessing; they do not replace authorization. This addresses broken object-level authorization.

Configure Spring Security's resource-server JWT support and explicit route permissions. Deny by default; grant minimum roles/scopes. Fine-grained checks can live at controller/service level, but enforce them close to the data/use-case boundary too. For RAG, apply ACL filters before text reaches an external model. A later answer filter cannot undo disclosure.

## 3.3 AI-specific threats

- **Prompt injection:** malicious user or retrieved text tries to override system behavior.
- **Sensitive data disclosure:** prompts, chunks, outputs, logs, traces, or provider retention expose private information.
- **Excessive agency:** model-selected actions have real-world impact.
- **Unsafe output handling:** generated SQL, HTML, URLs, commands, or tool arguments are trusted.
- **Denial of service/cost abuse:** huge inputs, loops, expensive tools, unbounded output.
- **Supply-chain/model risks:** provider/model changes, unsafe packages/artifacts, unreviewed prompt changes.

Mitigate in deterministic code: file and input limits, output schemas, allowlisted tools, quotas, authorization on every action, human approval for high-impact operations, data minimization, and audit trails.

## 3.4 Secrets, privacy, retention, logging

Use secret management or managed identity. Rotate provider keys and scope them minimally. Redact authorization headers, API keys, prompts, document text, and sensitive outputs from logs/traces by default. Store content only when there is a product need, retention schedule, lawful basis, and access policy.

Before sending data to a model provider, understand its data handling, region, retention, training, and contract settings. Do not promise privacy based only on a client toggle or model choice.

## 3.5 Safe tool calling

Treat model tool calls as proposals. Allowlist tool names, validate argument schemas, independently authorize the current actor, bound tool results, and use idempotency for side effects. Never give a model arbitrary SQL or a general shell. Require user confirmation for consequential actions when policy requires it.

## Practice

Create a threat model for the RAG capstone. Write tests proving a user cannot retrieve another tenant's document by changing an ID, using semantic search, or asking for its citation. Simulate prompt injection and prove tool permissions do not change.

## Questions and answers

**If a document is in the vector index, can any user retrieve it?**  
No. Index presence is not authorization. Filter by tenant and document ACL during retrieval and verify access again when rendering citations.

**Can a stronger system prompt fix prompt injection?**  
A prompt guides behavior but cannot enforce security. Use least privilege, deterministic authorization, strict tool allowlists, output validation, and monitoring.

**Is a signed JWT automatically authorized?**  
No. Validate issuer, audience, expiry, scopes/roles, tenant, and resource permission.

**Should prompts appear in traces for debugging?**  
Not by default. Use correlation IDs and safe metadata. Content capture needs explicit privacy controls, restricted access, and retention.

## Mastery checkpoint

Show the threat model, authorization path, negative tests, and data flow. Explain why model instructions cannot replace access control.