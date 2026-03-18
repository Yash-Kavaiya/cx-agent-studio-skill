---
name: cx-agent-studio
description: >
  Guide and expert reference for Google Cloud CX Agent Studio (Gemini Enterprise for CX).
  Use when the user asks about: building conversational agents, Dialogflow CX, CX Agent Studio,
  agent instructions, XML prompt formatting, few-shot examples, global instructions,
  Python tools, OpenAPI tools, client function tools, MCP tools, data store tools,
  tool chaining, wrapping APIs, context engineering, session variables, variable types,
  callbacks (before_model_callback, after_model_callback, before_tool_callback,
  after_tool_callback, before_agent_callback, after_agent_callback), custom payloads,
  guardrails, prompt guard, blocklist, safety levels, evaluation, golden test cases,
  scenario test cases, evaluation metrics, hallucination detection, tool correctness,
  best practices, session handling, partial responses, deterministic greetings,
  agent escalation, voice audio patterns, error handling, dynamic prompts,
  multi-agent architecture, root agent, sub-agent, agent routing, agent description,
  agent settings, CES API, REST API, RPC API, client libraries, authentication,
  ADC credentials, service account, x-ces-session-context, ces_requests, ToolContext,
  CallbackContext, LlmRequest, LlmResponse, Content, Part, FunctionCall, or
  any request to build, debug, optimize, or integrate with CX Agent Studio agents.
---

# CX Agent Studio

Customer Experience Agent Studio (CX Agent Studio) is a minimal-code conversational agent builder built on the Agent Development Kit (ADK), representing the evolution of Dialogflow CX.

**Console:** [ces.cloud.google.com](https://ces.cloud.google.com)

## Core Capabilities

- **AI-Augmented Building** — Generate agents from a 1–2 sentence goal using Gemini
- **Bi-directional Streaming** — Ultra-low latency voice interactions
- **Asynchronous Tool Calling** — Maintains natural conversation flow during backend calls
- **Multi-Agent Architecture** — Root and sub-agents for modular, scalable design
- **Evaluations** — Golden and scenario-based automated testing

---

## Quick Actions

### 1. Build an Agent Application
- Open [ces.cloud.google.com](https://ces.cloud.google.com) → **Create agent**
- Add instructions, tools, variables, guardrails, and callbacks
- **Read** `references/agents.md` for full agent settings and architecture

### 2. Write Agent Instructions
- Use natural language; click **Restructure instructions** to format as XML
- Reference syntax: `{variable_name}`, `{@TOOL: tool_name}`, `{@AGENT: Agent Name}`
- **Read** `references/instructions.md` for XML format, few-shot examples, and formatting best practices

### 3. Add Tools
- Python code tools, OpenAPI, Client function, MCP, Data store, Google Search, and more
- **Read** `references/tools.md` for all tool types, `context` object, `ces_requests`, and code samples

### 4. Use Variables
- Store/retrieve runtime data across turns
- Updated by tools (`context.state`) or callbacks (`callback_context.variables`)
- **Read** `references/variables.md` for types, updating patterns, and `x-ces-session-context`

### 5. Add Guardrails
- Prompt guard, blocklist, safety levels, and custom code rules
- **Read** `references/guardrails.md` for configuration options and code-based rule examples

### 6. Add Callbacks
- Python hooks at 6 execution points in the conversation lifecycle
- Custom payloads, deterministic responses, error handling, audit logging
- **Read** `references/callbacks.md` for all callback types with code samples

### 7. Evaluate Agent Quality
- Golden test cases (regression) and scenario test cases (AI-simulated users)
- **Read** `references/evaluation.md` for metrics, personas, and REST API

### 8. Apply Best Practices
- **Read** `references/best-practices.md` for patterns on session handling, tool chaining, voice/audio, UI integration, and error handling

---

## Reference Files

| File | Read When |
|------|-----------|
| `references/agents.md` | User asks about agent architecture, multi-agent setup, agent settings, routing, sub-agents |
| `references/instructions.md` | User asks about writing instructions, XML format, few-shot examples, global instructions |
| `references/tools.md` | User asks about any tool type, Python tools, OpenAPI, `context` object, `ces_requests`, tool chaining |
| `references/variables.md` | User asks about variables, session state, `context.state`, `callback_context.variables`, dynamic prompts |
| `references/guardrails.md` | User asks about guardrails, prompt guard, blocklist, safety, custom rules |
| `references/callbacks.md` | User asks about any callback type, custom payloads, deterministic responses, session patterns |
| `references/best-practices.md` | User asks about best practices, patterns, session handling, voice/audio, UI, error handling |
| `references/evaluation.md` | User asks about testing, evaluations, golden tests, scenario tests, metrics, personas |

---

## Callback Execution Flow

```
User message arrives
  │
  ├─ before_agent_callback    ← Setup / validation / skip agent
  │
  ├─ before_model_callback    ← Input guardrails / context injection / skip LLM
  │    │
  │    ├─ [LLM processes]
  │    │
  │    └─ after_model_callback ← Output formatting / PII redaction / payloads
  │
  ├─ before_tool_callback     ← Validate args / mock response / rate limit
  │    │
  │    ├─ [Tool executes]
  │    │
  │    └─ after_tool_callback  ← Transform response / save to variables / audit
  │
  └─ after_agent_callback     ← Cleanup / counters / override response
```

## Important Notes

- Always write instructions in **English** for highest quality
- Agent descriptions are used for routing — write them specifically
- Use `context.state` in Python tools and `callback_context.variables` in callbacks (same underlying state)
- Variables cannot be updated by the agent directly — only by tools and callbacks
- Custom payloads (JSON) are **not visible to the LLM** — only in the final API response
- Callback function names are **fixed** — the runtime finds them by exact name
- Multiple callbacks of the same type execute in definition order
- `partial = True` on `LlmResponse` forces the agent to continue processing after that response
