# AI agents, tools and orchestration

## 1. What people mean by agent

“Agent” is used inconsistently. In application engineering, a practical agent is a model-driven control loop that observes input/state, chooses among constrained actions, receives tool results, and continues until a stop condition or budget is reached.

A normal LLM call maps a prompt to a response. An agentic workflow may call search, retrieve records, invoke a calculator or request approval. The extra autonomy increases capability and risk. Many business tasks work better as a deterministic workflow with one or two model steps than as an open-ended agent.

## 2. A bounded control loop

~~~mermaid
flowchart TD
  S[Authenticated request and state] --> P[Policy and allowed actions]
  P --> M[Model selects response or tool proposal]
  M --> V{Validate action}
  V -->|Answer| F[Validate and return]
  V -->|Tool call| X[Authorize and execute tool]
  X --> R[Tool result, treated as untrusted data]
  R --> B{Budget and stop condition}
  B -->|Continue| M
  B -->|Stop or limit| F
~~~

The model proposes a tool call; trusted code validates and executes it. Never give the model direct shell, unrestricted SQL, arbitrary URLs, or a broad cloud credential because the prompt says to be careful.

## 3. Tool design

A tool should have:
- one narrow purpose;
- typed input schema and validation;
- explicit authorization and tenant scope;
- bounded result size and execution time;
- well-defined errors;
- idempotency for side effects;
- audit and tracing;
- tests and a human approval path where impact warrants it.

A read-only search tool is safer than a “do anything” browser or database tool. For writes, separate proposal from commit: show a preview, verify permissions, ask for approval when required by product policy, and make the change idempotent.

Tool output is untrusted. A web page, document or API response can contain prompt injection or malformed data. Normalize and bound output, keep source provenance, and do not let tool content redefine system policy.

## 4. Planning, state and stopping

Some workflows ask the model to plan multiple steps. Keep the plan inside application-defined constraints:
- maximum steps/tool calls;
- maximum wall time and tokens;
- allowed tool set per user/task;
- per-tool budgets;
- explicit stopping rules;
- cancellation propagation;
- retry limits;
- fallback or escalation.

Persist important workflow state in application storage rather than relying on a long prompt alone. Store version, status, prior actions, results, approvals, deadlines and idempotency keys. Do not store hidden reasoning as an audit substitute; record observable actions and decisions with sufficient privacy controls.

## 5. Error handling and recovery

A tool can time out, return partial data, reject authorization, or produce stale/ambiguous results. The orchestrator must define:
- whether to retry;
- whether the action may have already happened;
- whether to ask the user;
- whether another source may be used;
- whether to stop and report incomplete work.

Do not repeat a non-idempotent write after a timeout without checking state. A generated plan is not a transaction. Use domain-level workflows and compensating actions for multi-step changes.

## 6. MCP: a protocol, not an authorization policy

The Model Context Protocol (MCP) is a standard protocol for connecting AI applications to context and tools through servers. It defines concepts such as tools, resources and prompts and transport/lifecycle details. MCP does not grant application authorization by itself. The host, server and downstream systems still need identity, consent, access controls, input validation, output handling and audit.

When connecting to an MCP server:
- verify server identity and provenance;
- inspect tools and required permissions;
- apply least privilege and user/tenant policy;
- treat server results as untrusted;
- review data transmitted and retained;
- bound tool calls and payload sizes;
- handle version and compatibility;
- defend against confused-deputy behavior.

For a backend, decide whether the product needs an MCP client/server boundary or a simpler internal API. Do not add a protocol merely because it is fashionable.

## 7. Function calling and structured outputs

Function calling lets a model emit a structured proposal associated with a tool schema. Structured output modes can constrain formatting. Neither makes the content trustworthy:
- validate every field, range and identifier;
- reauthorize against trusted identity;
- reject unknown operations;
- validate referenced records belong to the caller;
- sanitize output for its next context;
- handle malformed or refused output explicitly.

A schema verifies shape, not business correctness or permission.

## 8. Human-in-the-loop design

Use human approval when the action has high impact, is difficult to reverse, affects money or access, or policy requires it. The approval should show the actual action and consequence, not a vague “allow AI?” message. Bind approval to a specific versioned proposal; if the action changes, require a new decision.

Human review is not a cure for bad interface design. Reviewers need context, time, ability to reject, and clear responsibility. Measure override rates and review burden.

## Questions and answers

**Q: Is every application using an LLM an agent?**  
A: No. A single model request or deterministic RAG pipeline is not necessarily agentic. The defining practical feature is model-selected actions in a control loop.

**Q: Can I trust a tool call because it matches a JSON schema?**  
A: No. A valid shape can still name the wrong record, exceed a user's permission or cause a harmful action. Apply domain validation and authorization.

**Q: Why not let the model decide when to stop?**  
A: The model may loop, spend excessive budget or stop before a required check. The application needs hard limits and authoritative stop conditions.

**Q: What does MCP provide?**  
A: A common protocol shape for exposing and consuming tools/resources/prompts. It does not automatically solve authentication, authorization, consent or safe execution.

## Practice

1. Design a read-only “search tickets” tool with tenant boundaries and a bounded result schema.
2. Add a maximum tool-call count and an explicit stop condition to an agent loop.
3. Simulate a timeout after a write may have succeeded; define idempotent recovery.
4. Place prompt injection in a tool result and document which controls prevent it from authorizing another action.
5. Compare a deterministic workflow with an agent for an expense approval task.

## Completion check

You can distinguish model proposals from trusted execution, design bounded tool schemas, explain MCP's scope, and identify when a fixed workflow or human approval is more appropriate.