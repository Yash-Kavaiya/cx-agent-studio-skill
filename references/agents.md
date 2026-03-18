# Agents in CX Agent Studio

**Console:** [ces.cloud.google.com](https://ces.cloud.google.com)

## Agent Hierarchy

An **agent application** is composed of one or more agents forming a hierarchical tree:

- **Root agent (steering agent):** Primary entry point and orchestrator. Handles main user interaction, understands overall goals, delegates to sub-agents.
- **Sub-agent (child agent):** Specialized agent for a specific task, domain, or capability (e.g., searching a database, handling returns). Promotes modularity and reusability.

Root agents can invoke sub-agents, and sub-agents can invoke other sub-agents.

## Creating Agents

### Create an agent application and root agent
1. Open the [CES console](https://ces.cloud.google.com) and select your project.
2. Click **Create** or **Create agent**.
3. Provide an agent application name and click **Create** (first creation may take 1–2 minutes).
4. Click the **+** in the top right of the root agent to add instructions and tools.

### Create a sub-agent
1. Click the **+** at the bottom of the root agent.
2. Click **Add sub-agent**.

## Agent Description (Critical for Routing)

The **Description** field in each agent's settings is sent to **other agents** in the application. The root agent uses sub-agent descriptions to decide when to delegate. Write clear, specific descriptions.

- ✅ `"Handles all order lookup, order status, and return requests. Use when the user mentions an order number or asks about shipping."`
- ❌ `"Order agent"`

## Agent Settings

### Agent application (global) settings
Access via the settings icon on the right side of the builder:

| Setting | Description |
|---------|-------------|
| **Global model** | Default model unless overridden per agent |
| **Default language** | Start all conversations in this language |
| **Additional languages** | Agent auto-switches to match user input |
| **Unsupported language handling** | Action when user speaks unsupported language |
| **Voice** | Speech synthesis voice |
| **Ambient sounds** | Background audio |
| **Response length** | Controls verbosity |
| **Allow user interruptions** | Let end-users interrupt agent speech |
| **Adapt when interrupted** | Agent adapts response knowing user may have missed part |
| **Agent lock** | Prevent changes |
| **Global instructions** | Instructions sent to every agent on every turn (brand tone, DOs/DON'Ts, shared variables) |
| **Tool execution mode** | Parallel or sequential tool execution |
| **Logging** | Interaction data, Cloud Logging, BigQuery export, redaction, audio recording |
| **Silence timeout** | Duration before prompting re-engagement |

### Per-agent settings
Access via context menu → **Edit config**:

| Setting | Description |
|---------|-------------|
| **Agent name** | Display name (use snake_case) |
| **Model** | Override the global model for this agent |
| **Description** | Used by other agents for routing decisions |
| **Custom code** | Python callbacks for this agent |

## Language Support

Design all agent instructions and prompts in **English** for highest quality. Agents automatically detect the end-user's language and respond in the same language.

To restrict or force a language, add an explicit instruction:
> "You only speak Italian and no other languages."

## Sample Agent Applications

Import sample agents via **New agent → Try a sample agent**.

### Cymbal Retail Agent
A demo assistant for Cymbal Home & Garden. Demonstrates:
- **Multi-agent architecture** — root agent + upselling sub-agent + out-of-scope sub-agent
- **Instructions** — structured XML with agent/tool/variable references
- **Tools** — Python tools, Google Search, OpenAPI (3-endpoint example), `end_session` system tool
- **Variables** — custom schema object (customer profile), boolean variables
- **Guardrails** — prompt guard, blocklist, safety rules
- **Callbacks** — `before_model_callback`, `after_model_callback`, `after_tool_callback`
- **Evaluations** — golden and scenario-based test cases pre-loaded

## Managing Agent Applications

From the agent list, each application shows:
- **Deployed to** — number of deployment channels
- **Sessions** — sessions in past 24 hours
- **Escalation** — escalations in past 24 hours
- **Context menu** — Import, Export, Delete agent
