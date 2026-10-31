# AI tools, durable workflows, and streaming

## A model tool call is a request, not an authorization

A model may propose a structured tool call, such as search_documents(query) or create_ticket(title, body). The backend must validate the schema, authenticate the user, authorize that action, constrain arguments, execute the tool, and return an observation. The model does not directly own credentials or permissions.

Use narrow tools with clear contracts. A search tool should search only resources the current user can access. A send-email tool should require confirmation and validate recipients. Do not expose arbitrary SQL, shell, filesystem, or unrestricted HTTP tools to a model.

## Agents and bounded workflows

An agent repeatedly chooses an action, observes a result, and continues. More steps mean more chances for latency, cost, and failure. Set limits on steps, time, tokens, and tool calls. Define a stopping condition, escalation path, and recovery behavior.

For known business processes, a deterministic state machine is often more reliable than an open-ended loop. A support workflow may be: classify request, search approved policy, draft answer, request human review, send after approval. The model can help at selected steps without controlling the whole workflow.

## Durable background jobs

A job can outlive an HTTP request. Persist its ID, owner, state, input reference, attempt count, and timestamps. Queue the job and return a status endpoint. Workers should use idempotency keys or conditional state transitions so redelivery does not duplicate side effects. Keep expensive model calls outside database transactions.

A durable workflow records enough state to resume after process restart. For multi-step work, store completed outputs and retries. A cancellation request should be checked between steps. Human approval is a state transition, not merely text in a prompt.

## Streaming

Server-Sent Events stream server-to-client events over HTTP and are often suitable for token progress. WebSockets support two-way communication but require connection lifecycle management. Choose the simplest transport that fits.

Define event types such as started, token, source, completed, and error. Handle client disconnects, cancellation, partial output, and reconnect semantics. Do not stream internal chain-of-thought or hidden prompts; stream the product's user-facing answer and status only. A provider can fail after some tokens have already been delivered, so the client must know whether the response completed.

Questions: Can a model decide whether a user may call a tool? No; the backend checks identity and permission. Why are duplicate jobs normal? Queues and workers can retry after uncertainty or crash. Why bound agent steps? To limit loops, cost, and impact.

Practice: design a document-search agent with one read-only search tool and one draft tool. Specify schemas, authorization, step limit, human approval, and failure states. Test a malicious document that asks the agent to call an unrelated tool.

Further reading: [FastAPI background tasks](https://fastapi.tiangolo.com/tutorial/background-tasks/) and [OWASP LLM Top 10](https://owasp.org/www-project-top-10-for-large-language-model-applications/).

## Durable state and exactly-once side effects

Model-driven workflows should have explicit states: draft, awaiting approval, approved, completed, failed, cancelled. The database owns state; a model can suggest a transition, but cannot skip policy. Use conditional updates or locks so two workers cannot both execute an action.

A worker might send a message successfully and then crash before recording success. On retry, it may send twice. Give every side effect an operation ID and use downstream idempotency if supported. Otherwise reconcile the downstream state before repeating. Keep external calls outside long database transactions.

A durable job stores owner, input reference, state, attempt count, timestamps, and outputs needed to resume. Cancellation should be checked between steps. A live token stream can be lost when a client disconnects; persist job status and provide a polling path if the result must be recoverable.

Practice: build a draft-reply workflow that needs human approval before sending. Test simultaneous approvals, duplicate worker delivery, cancellation, and a send timeout with unknown outcome.
