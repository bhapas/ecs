# 0054: LLM Tool Call Fields

- Stage: **Proposal**
- Date: **TBD**
- Target maturity: **alpha**

## Summary

This RFC proposes a new top-level `llm_tool.*` field set to capture tool (function) calls made by Large Language Models during agentic workflows. As AI agents increasingly invoke external tools — APIs, code interpreters, retrieval systems, and MCP servers — there is no standardized way to observe, audit, and alert on these interactions in Elastic. This field set enables security teams to monitor tool usage, detect policy violations, and trace multi-step agent reasoning chains.

## Usage

Organizations deploying LLM-based agents (e.g., customer support bots, code assistants, autonomous workflows) need visibility into what tools agents invoke, what parameters they pass, whether execution succeeded, and whether safety guardrails were respected.

**Security use case:** A detection rule alerts when `llm_tool.safety.classification == "blocked"` AND `llm_tool.result.status != "denied"` — indicating a guardrail disagreement where a tool was flagged but still executed.

**Observability use case:** Dashboard panels show p95 `llm_tool.result.duration_ms` by `llm_tool.name`, revealing slow tool backends. Alerts fire when `llm_tool.chain.step` exceeds `llm_tool.chain.max_steps * 0.8` — an agent nearing its loop limit.

**Compliance use case:** Audit logs filter on `llm_tool.approval.required == true AND llm_tool.approval.status == "auto_approved"` to find tool calls that bypassed human review in regulated environments.

## Fields

All proposed fields are defined in [`schemas/llm_tool.yml`](../../schemas/llm_tool.yml) and summarized in [`rfcs/text/0054/llm_tool.yml`](0054/llm_tool.yml).

Key field groups:

| Group | Fields | Purpose |
|-------|--------|---------|
| Identity | `llm_tool.id`, `llm_tool.name`, `llm_tool.type` | Identify which tool was called |
| Input | `llm_tool.parameters` (flattened) | Capture heterogeneous tool inputs |
| Output | `llm_tool.result.*` | Execution outcome, errors, timing |
| Safety | `llm_tool.safety.*` | Guardrail classifications and policy |
| Chain | `llm_tool.chain.*` | Position in multi-step reasoning |
| Approval | `llm_tool.approval.*` | Human-in-the-loop decisions |

### Justification for `flattened` type on `llm_tool.parameters`

Tool parameters are heterogeneous — each tool defines its own schema. A `get_weather` tool takes `{"location": "...", "days": 5}` while a `run_sql` tool takes `{"query": "SELECT ..."}`. Defining explicit leaf fields is not feasible because the parameter shapes are unbounded and tool-specific. The `flattened` type allows indexing leaf values for search while accommodating arbitrary schemas.

## Source data

### OpenAI Function Calling Response

```json
{
  "@timestamp": "2026-04-30T10:15:30.000Z",
  "gen_ai.operation.name": "chat",
  "gen_ai.model.id": "gpt-4o",
  "llm_tool.id": "call_abc123def456",
  "llm_tool.name": "get_weather_forecast",
  "llm_tool.type": "function",
  "llm_tool.parameters": {"location": "San Francisco", "days": 5, "units": "celsius"},
  "llm_tool.result.status": "success",
  "llm_tool.result.content": "Current temperature: 18°C, partly cloudy. 5-day high: 22°C.",
  "llm_tool.result.duration_ms": 245,
  "llm_tool.chain.id": "chain_session_001",
  "llm_tool.chain.step": 2,
  "llm_tool.safety.classification": "safe"
}
```

### Anthropic MCP Tool Use (Blocked)

```json
{
  "@timestamp": "2026-04-30T10:20:00.000Z",
  "gen_ai.operation.name": "chat",
  "gen_ai.model.id": "claude-opus-4-6",
  "llm_tool.id": "toolu_01A2B3C4D5",
  "llm_tool.name": "filesystem_read",
  "llm_tool.type": "mcp",
  "llm_tool.description": "Read a file from the local filesystem",
  "llm_tool.parameters": {"path": "/etc/shadow"},
  "llm_tool.result.status": "denied",
  "llm_tool.result.error.message": "Access denied: path outside sandbox boundary",
  "llm_tool.result.error.type": "permission_denied",
  "llm_tool.result.duration_ms": 2,
  "llm_tool.safety.classification": "blocked",
  "llm_tool.safety.policy_id": "prod-tool-policy-v3",
  "llm_tool.safety.reason": "Tool attempts to access filesystem outside sandbox.",
  "llm_tool.chain.id": "chain_mcp_session_042",
  "llm_tool.chain.step": 5,
  "llm_tool.chain.max_steps": 10,
  "llm_tool.approval.required": true,
  "llm_tool.approval.status": "rejected",
  "llm_tool.approval.reviewer": "security-team@example.com"
}
```

### Code Interpreter with Timeout

```json
{
  "@timestamp": "2026-04-30T10:25:00.000Z",
  "gen_ai.operation.name": "chat",
  "gen_ai.model.id": "gpt-4o",
  "llm_tool.id": "call_xyz789",
  "llm_tool.name": "python_executor",
  "llm_tool.type": "code_interpreter",
  "llm_tool.parameters": {"code": "import time; time.sleep(300); print('done')"},
  "llm_tool.result.status": "timeout",
  "llm_tool.result.error.message": "Execution exceeded 60s time limit",
  "llm_tool.result.error.type": "timeout",
  "llm_tool.result.duration_ms": 60000,
  "llm_tool.result.tokens_used": 0,
  "llm_tool.chain.id": "chain_code_007",
  "llm_tool.chain.step": 1,
  "llm_tool.chain.max_steps": 5,
  "llm_tool.safety.classification": "suspicious",
  "llm_tool.safety.reason": "Long-running code execution pattern detected"
}
```

## Scope of impact

**Ingestion mechanisms:** Beats/Elastic Agent integrations for LLM providers (OpenAI, Anthropic, Azure OpenAI) and agent frameworks (LangChain, CrewAI, AutoGen) would emit these fields. Custom ingest pipelines parsing LLM API response logs would map tool_calls arrays to `llm_tool.*` events.

**Usage mechanisms (Kibana/Security):** Detection rules can alert on safety violations, unauthorized tool use, and chain runaway scenarios. Dashboards can visualize tool call patterns, latency distributions, and approval workflows. The Security app could surface tool-call timelines alongside existing process and network events.

**ECS project:** New field set YAML, generated artifacts, and field reference documentation. Alignment with OpenTelemetry GenAI semantic conventions for tool interactions (currently in development in OTel semconv).

## Concerns

**Concern 1: Overlap with gen_ai.* fields**
Resolution: `gen_ai.*` captures model-level request/response metadata (tokens, model ID, operation). `llm_tool.*` captures individual tool invocations within a single model interaction. They are complementary — a single `gen_ai` operation may produce multiple `llm_tool` events. The relationship is 1:N.

**Concern 2: `flattened` type for parameters limits aggregation**
Resolution: `flattened` is intentional — tool parameters are unbounded and tool-specific. Users needing aggregation on specific parameter values can use runtime fields or ingest-time extraction to promoted fields. The alternative (nested object per tool) would explode mapping complexity.

**Concern 3: High cardinality on llm_tool.name**
Resolution: Tool names are bounded by an organization's tool registry (typically 10-100 tools per agent system). This is comparable to `event.action` cardinality. Index template settings can apply `eager_global_ordinals` if needed.

**Concern 4: OTel alignment unclear**
Resolution: OTel GenAI semconv is actively developing tool-call attributes (gen_ai.tool.call.id, gen_ai.tool.name). We align naming where possible and will add `otel:` mappings once the OTel spec stabilizes. The `alpha` maturity allows us to adjust before GA.

## People

* @bhapas | author
* TBD | subject matter expert (GenAI observability)
* TBD | security detection engineering

## References

- [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [OpenAI Function Calling API](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [Model Context Protocol Specification](https://modelcontextprotocol.io/)

### RFC Pull Requests

* Proposal: https://github.com/elastic/ecs/pull/TBD
