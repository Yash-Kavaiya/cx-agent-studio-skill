# Instructions in CX Agent Studio

Agent instructions provide detailed natural-language guidance for the model — its goal, persona, tool/sub-agent usage, and behavior.

## Syntax

| Reference | Format |
|-----------|--------|
| Variable | `{variable_name}` |
| Tool | `{@TOOL: tool_name}` |
| Sub-Agent | `{@AGENT: Agent Name}` |

The instructions editor highlights recognized references as colored chips. Shortcuts:
- Type `@` → context menu with Agent / Tool / Variable options
- Type `{` → context menu with available variables

## Plain Language Example

```
CURRENT CUSTOMER: {username}

You are the main Weather Agent coordinating multiple agents.
Your primary responsibility is to provide weather information.
Use {@TOOL: get_weather} ONLY for specific weather requests (e.g., 'weather in London').

If you know the user's name, always greet them by their name.

You have specialized sub-agents:
 1. Greeting Agent: Handles simple greetings like 'Hi', 'Hello'.
 2. Farewell Agent: Handles simple farewells like 'Bye', 'See you'.

Analyze the user's query.
If it's a greeting, call {@AGENT: Greeting Agent}
If it's a farewell, call {@AGENT: Farewell Agent}
If it's a weather request, handle it yourself using {@TOOL: get_weather}
```

## Restructure Instructions (XML Format)

Click **Restructure instructions** to auto-format into the recommended XML structure. XML formatting improves model reliability.

### XML Tag Reference

| Tag | Purpose |
|-----|---------|
| `<role>` | Agent's core function or responsibility |
| `<persona>` | Personality, tone, behavioral guidelines |
| `<primary_goal>` | Within `<persona>` — agent's main objective |
| `<constraints>` | Rules and limitations the agent must follow |
| `<taskflow>` | Conversational flows as a series of subtasks |
| `<subtask>` | A specific part of the conversation flow |
| `<step>` | An individual step within a subtask |
| `<trigger>` | Condition or user input that initiates a step |
| `<action>` | What the agent should do when triggered |
| `<examples>` | Inline few-shot examples |

### XML Example

```xml
CURRENT CUSTOMER: {username}

<role>The main Weather Agent coordinating multiple agents.</role>

<persona>
  <primary_goal>To provide weather information.</primary_goal>
  General guidelines: Follow constraints and task flow precisely.
</persona>

<constraints>
  1. Use {@TOOL: get_weather} ONLY for specific weather requests (e.g., 'weather in London').
  2. If the user's name is known, always greet them by name.
</constraints>

<taskflow>
  <subtask name="Query Analysis and Routing">
    <step name="Handle Greeting">
      <trigger>User query is a simple greeting (e.g., 'Hi', 'Hello').</trigger>
      <action>Call {@AGENT: Greeting Agent}.</action>
    </step>
    <step name="Handle Weather Request">
      <trigger>User query is a specific weather request (e.g., 'weather in London').</trigger>
      <action>Use {@TOOL: get_weather} and provide the result.</action>
    </step>
  </subtask>
</taskflow>

<examples>
</examples>
```

## Inline Few-Shot Examples

Use `<examples>` to guide behavior for complex formatting or nuanced logic. **Use sparingly** — too many examples causes overfitting.

### When to Use
- Fixing consistent failures where the model misunderstands instructions
- Complex non-standard output formatting
- Nuanced decision-making that if-then instructions can't capture

### When NOT to Use
- Start with clear instructions first; only add examples if they fail
- Don't enumerate every possible query
- Don't add many examples — it reduces generalization ability

### Components

| Component | Tag | Description |
|-----------|-----|-------------|
| User input | `[user]` | End-user's message |
| Model response | `[model]` | Agent's text response |
| Tool call | ` ```tool_code ``` ` | Tool invocation with arguments |
| Tool output | ` ```tool_outputs ``` ` | Data returned from tool |

### Format

```
<examples>
  EXAMPLE 1:
  Begin example
  [user]
  What's the weather in London?
  [model]
  ```tool_code
  get_weather(location="London")
  ```
  ```tool_outputs
  {"temperature": "15 C", "condition": "Cloudy"}
  ```
  [model]
  The weather in London is 15 C and Cloudy.
  End example
</examples>
```

## Formatting Agent Responses

### Chunking and whitespace
- Never write dense paragraphs — users scan, they don't read
- Limit text blocks to 1–2 sentences maximum
- Insert a line break between every distinct idea

### Strategic bolding
- Bold the most important data points so they stand out
- Always bold: **Product Names**, **Prices**, **Dates**, **Order Numbers**, **Deadlines**

### Lists over text
- If mentioning more than two items or steps, use a list
- Use `-` bullets for options, `1.` numbered lists for instructions

## Global Instructions

Defined in advanced agent application settings. Sent to **every agent** on every turn, on top of agent-specific instructions.

Use for: brand tone, DOs/DON'Ts, globally shared variables, customer profile context.

## Refine Instructions

Select any portion of instructions text → click **Refine** → enter requirements to use AI to improve the selected content.

## Language
Always write instructions in **English** for highest quality. Agents automatically detect and respond in the user's language unless instructed otherwise.
