# CX Agent Studio Skill

> An expert agent skill for building, optimizing, and integrating with **Google Cloud CX Agent Studio** (Gemini Enterprise for Customer Experience).

[![Install via ClawHub](https://img.shields.io/badge/ClawHub-Install-blue?style=flat-square)](https://clawhub.ai/Yash-Kavaiya/cx-agent-studio)
[![skills.sh](https://img.shields.io/badge/skills.sh-compatible-green?style=flat-square)](https://skills.sh)
[![Google Cloud](https://img.shields.io/badge/Google%20Cloud-CX%20Agent%20Studio-orange?style=flat-square&logo=googlecloud)](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps)

---

## 🚀 Install

**ClawHub (OpenClaw):**
```bash
npx clawhub install cx-agent-studio
```

**skills.sh (Claude Code, Cursor, Gemini CLI, GitHub Copilot):**
```bash
npx skills add Yash-Kavaiya/cx-agent-studio-skill
```

---

## 🧠 What This Skill Does

Gives your AI coding agent deep knowledge of CX Agent Studio — the minimal-code conversational agent builder built on the Agent Development Kit (ADK), and the evolution of Dialogflow CX.

Once installed, your agent can:
- Guide you through **building** agents, sub-agents, and multi-agent architectures
- Write and **format instructions** (XML structure, few-shot examples, global instructions)
- Generate **Python tool code** with proper `context` object usage, `ces_requests`, and docstrings
- Write all 6 **callback types** with real working code samples
- Configure **variables**, **guardrails**, and **OpenAPI tools**
- Apply **best practices** for session handling, tool chaining, error recovery, voice/audio
- Explain **evaluation** metrics, test case types, and AI issue discovery
- Reference **REST/RPC API** endpoints and **authentication** flows

---

## 📚 Skill Coverage

| Topic | Coverage |
|-------|----------|
| **Agents** | Root agent, sub-agents, multi-agent architecture, routing via descriptions, settings |
| **Instructions** | Plain language, XML restructuring, few-shot examples, global instructions, response formatting |
| **Tools** | Python, OpenAPI, Client function, MCP, Data store, Google Search, Salesforce, ServiceNow, System, Widget |
| **Variables** | Types, scoping, `context.state`, `callback_context.variables`, dynamic prompts, `x-ces-session-context` |
| **Callbacks** | All 6 types with code, custom payloads, session patterns, error handling, partial responses |
| **Guardrails** | Prompt guard, blocklist, safety levels, custom code rules |
| **Best Practices** | Tool chaining, context engineering, session handling, voice/audio, UI, collaboration |
| **Evaluations** | Golden + scenario tests, metrics, personas, AI issue discovery, scheduled runs |
| **APIs** | REST v1/v1beta, RPC gRPC, client libraries (Python, Go, Java, Node.js, C#, PHP, Ruby) |
| **Authentication** | ADC, service accounts, OAuth, API keys, gcloud CLI, Secret Manager |

---

## ⚡ Callback Execution Flow

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

---

## 📁 Skill Structure

```
cx-agent-studio-skill/
├── SKILL.md                        ← Agent instructions + reference routing table
└── references/
    ├── agents.md                   ← Agent hierarchy, settings, routing, sample agents
    ├── instructions.md             ← XML format, few-shot, global instructions, formatting
    ├── tools.md                    ← All tool types, Python tools, OpenAPI, ces_requests
    ├── variables.md                ← Types, scoping, update patterns, x-ces-session-context
    ├── callbacks.md                ← All 6 callback types, custom payloads, error patterns
    ├── guardrails.md               ← Prompt guard, blocklist, safety, custom rules
    ├── best-practices.md           ← Session patterns, tool chaining, voice/audio, UI
    └── evaluation.md               ← Test types, metrics, personas, REST API
```

---

## 🛠️ Example Trigger Phrases

After installing, try asking your AI agent:

- *"Write a before_model_callback to add a legal disclaimer on session start"*
- *"Create a Python tool that wraps our CRM API and returns only relevant fields"*
- *"How do I set up a multi-agent architecture for a retail assistant?"*
- *"Write an after_model_callback to add escalation custom payload"*
- *"What's the best way to chain tool calls deterministically?"*
- *"How do I pass session variables to an OpenAPI tool?"*
- *"Create scenario evaluation test cases for my booking agent"*
- *"Show me how to use partial responses for real-time UI updates"*
- *"How do I authenticate to the CES API using Application Default Credentials?"*

---

## 📋 REST API Quick Reference

**Service endpoint:** `https://ces.googleapis.com`

**Key resources (v1beta):**

| Resource | Operations |
|----------|-----------|
| `apps` | create, get, list, patch, delete, export, import, executeTool, runEvaluation |
| `apps.agents` | create, get, list, patch, delete |
| `apps.tools` | create, get, list, patch, delete |
| `apps.sessions` | runSession, generateChatToken |
| `apps.guardrails` | create, get, list, patch, delete |
| `apps.evaluations` | create, get, list, patch, delete, uploadAudio |
| `apps.versions` | create, get, list, restore, delete |
| `apps.conversations` | get, list, delete, generateEvaluation |

**RPC services:** `AgentService`, `SessionService`, `ToolService`, `EvaluationService`, `WidgetService`

---

## 📦 Client Libraries

Available for: **Python**, **Go**, **Java**, **Node.js**, **C#**, **PHP**, **Ruby**

Download from: `https://storage.googleapis.com/gassets-api-ai/ces-client-libraries/`

Authentication via [Application Default Credentials (ADC)](https://cloud.google.com/docs/authentication/application-default-credentials).

---

## 🔗 Official Documentation

- [CX Agent Studio Overview](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps)
- [Console](https://ces.cloud.google.com)
- [Python Runtime Reference](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps/reference/python)
- [REST API v1beta](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps/reference/rest/v1beta-overview)
- [Best Practices](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps/best-practices)
- [ADK Documentation](https://google.github.io/adk-docs/)

---

## 🤝 Contributing

PRs welcome! Add patterns, code samples, or update references as the platform evolves.

1. Fork the repo
2. Update the relevant `references/*.md` file
3. Open a PR with a brief description of what changed

---

## 📄 License

MIT

---

*Built for [Google Cloud CX Agent Studio](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps) — last synced from official docs March 2026.*
