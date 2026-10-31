# AI application security and threat modeling

## Scope

AI systems inherit ordinary application risks and add risks from untrusted prompts, retrieved documents, model outputs, training data and tools. Security comes from layered controls in code, identity systems, infrastructure and operations. A prompt instruction is not an access-control boundary.

## 1. Start with assets, actors and trust boundaries

List what must be protected:
- user/tenant data and documents;
- credentials, API keys and model artifacts;
- system prompts and internal policy;
- model/tool budget and service availability;
- integrity of answers, actions and audit records;
- personal or regulated information.

Actors include authenticated customers, malicious insiders, compromised dependencies, hostile document authors, external model providers and automated agents. Draw where data crosses client, backend, retrieval store, model provider, tool server and human reviewer. Mark which identity is trusted at each boundary.

Threat modeling asks: what can go wrong, how likely and harmful is it, what control prevents or detects it, and who owns the residual risk?

## 2. Prompt injection

Prompt injection is an attempt to make a model ignore intended instructions or misuse available data/tools. It may arrive directly in user text or indirectly inside a web page, email, PDF, code comment or retrieved chunk.

The model processes instructions and content in one context; textual labels alone do not create a hard security separation. Defenses:
- keep high-value authorization in code;
- grant narrow tools and data access;
- do not put secrets in prompts;
- delimit and label untrusted content;
- validate all tool proposals and output;
- restrict outbound network access;
- require confirmation for consequential writes;
- test direct and indirect injection cases;
- monitor and update threat tests as workflows change.

No single prompt phrase prevents injection reliably. Reduce impact if the model is manipulated.

## 3. Sensitive information disclosure

Risks include prompt logging, retrieval across tenants, accidental inclusion in context, provider retention, training-data memorization, model outputting secret-like text, and overly broad telemetry access.

Controls:
- data classification and minimization;
- authorization before retrieval and cache lookup;
- tenant-keyed caches and indexes;
- provider data-processing/retention review;
- redaction and access controls for traces;
- short retention and deletion procedures;
- secrets manager and key rotation;
- do not expose prompts, chain traces or internal reasoning by default.

A public model API request can leave your infrastructure. Confirm policy, region, retention and contractual terms through current provider documentation before sending protected data.

## 4. Insecure output handling

A model may emit HTML, SQL, shell snippets, URLs, code, Markdown links or structured fields. Treat output as untrusted input to its next destination:
- escape HTML for the rendering context;
- parameterize database queries;
- never eval or execute generated code directly;
- validate URLs and redirect destinations;
- apply schema and business-rule validation;
- sanitize Markdown/HTML if supported;
- set content-security protections appropriate to the application.

“JSON mode” protects syntax/shape, not semantic safety.

## 5. Excessive agency and unsafe tools

A system with too many capabilities or high privileges can turn a prompt injection into a serious action. Apply:
- least privilege per user and task;
- read/write separation;
- allowlisted operations and destinations;
- per-tool schemas, rate limits and budgets;
- human review for high impact;
- dry-run and preview;
- idempotency, rollback and audit;
- independent approval of policy changes.

Do not let model-suggested identity or tenant identifiers replace authenticated context.

## 6. Retrieval and vector-store threats

RAG risks include cross-tenant retrieval, poisoned content, stale permissions, malicious parser files, oversized payloads, source spoofing and citation laundering. Enforce ACL filters before context assembly; retain provenance; validate file types and size; sandbox risky parsers; hash and version sources; delete stale vectors; test permissions after updates.

A citation is not proof that the cited source supports the claim. Validate reference identity and measure support quality.

## 7. Data poisoning and supply chain

Training or retrieval data can be manipulated. Model weights, tokenizer files, plugins, packages and containers can be compromised or untrusted. Controls:
- provenance and checksums;
- review source/maintainer and license;
- dependency pinning and vulnerability scanning;
- artifact signing and access control;
- isolated build pipelines;
- dataset lineage and anomaly review;
- controlled model registry promotion.

Never load a model artifact from an untrusted source into a privileged environment without evaluating its format and execution behavior. Some serialization formats can execute code on load.

## 8. Denial of service and cost abuse

Attackers can submit oversized prompts, long documents, repeated retries, expensive tool chains or high concurrency. Bound:
- request bytes and tokens;
- context and output length;
- tool calls and recursion;
- queue depth and per-user concurrency;
- retry count and wall-clock time;
- batch size and document upload volume.

Apply identity-based quotas, anomaly detection, cancellation and fast overload responses. Cost monitoring needs per-tenant attribution and alerts. Avoid unbounded “retry until success.”

## 9. Model limitations, bias and human impact

Risk is not only a security exploit. Evaluate false positives/negatives by meaningful user segments, language, accessibility and real deployment conditions. Determine whether outputs affect rights, access, employment, health, finance or safety. Use domain expert and legal review where required by the use case and applicable jurisdiction.

Human oversight must be meaningful: reviewer has authority, context, time and a clear path to challenge or correct the result. Document intended use, excluded use, known limits, monitoring and incident response.

## 10. Testing and incident response

Security evaluation should include:
- direct and indirect prompt-injection cases;
- cross-tenant retrieval attempts;
- unauthorized tool proposal;
- malformed structured output;
- sensitive-data canaries in non-production;
- oversized inputs and concurrency load;
- stale/deleted document access;
- provider outage and refusal behavior;
- output rendering injection.

On incident: contain tool/access capabilities, preserve scoped evidence, revoke credentials if needed, assess affected data/users, notify owners under policy, roll back configuration/model/index, and test the fix. Keep response procedures aligned with organization policy and applicable law.

## Questions and answers

**Q: Can a system prompt keep a secret safe?**  
A: No. Treat prompts as potentially observable by the model, logs, provider and outputs. Keep secrets in credential systems and avoid sending them to the model.

**Q: Does a refusal-trained model eliminate harmful tool use?**  
A: No. Model behavior can fail or be manipulated. Enforce permissions and action constraints outside the model.

**Q: Is prompt injection the same as SQL injection?**  
A: No. They are different mechanisms. SQL injection exploits unsafe query construction; prompt injection manipulates model behavior through context. Both require defenses at the correct boundary.

**Q: Is a vector database an authorization system?**  
A: No. It can filter metadata, but trusted application policy must define and enforce which records a caller may retrieve.

## Practice

1. Draw trust boundaries for an enterprise RAG assistant and label all data leaving your network.
2. Threat-model a tool that can create calendar events; include duplicate requests, tenant IDs and prompt injection.
3. Write test cases for a malicious retrieved document and an unauthorized cross-tenant query.
4. Create a data-flow inventory for prompt, retrieved evidence, output, logs and provider telemetry.
5. Define incident response if a model deployment accidentally exposes internal retrieved content.

## Completion check

You can identify assets and trust boundaries, explain prompt injection and output risks, enforce least privilege in code, and create a test/monitor/response plan for AI-specific threats.