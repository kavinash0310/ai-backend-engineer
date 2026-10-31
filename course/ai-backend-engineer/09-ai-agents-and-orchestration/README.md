# AI Agents and Orchestration

An agent is a software system in which a model selects or sequences actions based on observations under an application-defined control loop. A chatbot with a single model response is not automatically an agent.

## Required curriculum

Tool schemas and authorization; bounded agent loops; planning/execution; explicit workflow state; memory and provenance; durable checkpoints; retries/idempotency; human review; LangGraph concepts; multi-agent trade-offs; MCP architecture and resources/prompts/tools; sandboxing; observability; evaluation and stop conditions.

Start with deterministic workflows. Give an agent only allowlisted, least-privilege tools, enforce all permissions in application code, bound steps/time/cost, and test refusal/failure paths. Compare autonomy against ordinary application logic before adopting it.


## Detailed lesson

- [Agents, tools and MCP](./01-agents-tools-and-mcp.md) — bounded control loops, tool design, state, recovery, approval and protocol limits.
- Official [MCP specification](https://modelcontextprotocol.io/specification/latest).
