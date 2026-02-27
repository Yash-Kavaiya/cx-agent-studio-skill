# CX Agent Studio Instructions Reference

Instructions provide natural language guidance to the conversational agent on its goal, persona, and sub-agents/tools to use.

## Reference Syntax

Always use proper syntax so references are highlighted as chips in the UI:
- **Variable**: `{variable_name}` (snake_case)
- **Tool**: `{@TOOL: tool_name}`
- **Sub-Agent**: `{@AGENT: Agent Name}`

*Note: Always use English for designing instructions. The agent will automatically detect and respond in the end-user's language unless restricted.*

## XML Instruction Structure

To improve model adherence to instructions, use the recommended XML tag structure:

| Tag | Description |
| --- | --- |
| `<role>` | Defines the agent's core function. |
| `<persona>` | Describes personality, tone, and behavioral guidelines. |
| `<primary_goal>` | Inside `<persona>`, specifies main objective. |
| `<constraints>` | Lists rules or limitations. |
| `<taskflow>` | Outlines conversational flows as a series of `<subtask>`s. |
| `<subtask>` | A specific part of the flow containing `<step>`s. |
| `<step>` | Contains a `<trigger>` and an `<action>`. |
| `<trigger>` | Condition or user input that initiates the step. |
| `<action>` | What the agent should do when triggered. |
| `<examples>` | Few-shot examples container. |

### Example XML Instruction

```xml
CURRENT CUSTOMER: {username}

<role>The main Weather Agent coordinating multiple agents.</role>

<persona>
 <primary_goal>To provide weather information.</primary_goal>
 How to handle prohibited topics and violations: Respond appropriately.
</persona>

<constraints>
 1. Use {@TOOL: get_weather} ONLY for specific weather requests.
 2. Always greet them by name.
</constraints>

<taskflow>
 <subtask name="Query Analysis and Routing">
 <step name="Handle Greeting">
 <trigger>User query is a simple greeting.</trigger>
 <action>Call {@AGENT: Greeting Agent}.</action>
 </step>
 <step name="Handle Weather Request">
 <trigger>User query is a specific weather request.</trigger>
 <action>Use {@TOOL: get_weather} to retrieve weather information.</action>
 </step>
 </subtask>
</taskflow>
```

## Inline Few-Shot Examples

Use few-shot examples inside `<examples>` to guide the model's behavior, especially for complex formatting or nuanced logic. Add examples only if instructions alone fail.

**Components**:
- `[user]`: User input
- `[model]`: Agent response
- `tool_code`: Input arguments to the tool
- `tool_outputs`: Data returned by the tool

**Format**:
```xml
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

## Text Formatting Guidelines

Instruct the agent to format text for readability:
- **Chunking**: Limit blocks to 1-2 sentences. Use line breaks.
- **Bolding**: Always bold Product Names, Prices, Dates, Order Numbers, and Deadlines.
- **Lists**: Convert 3+ items into bulleted (-) or numbered (1.) lists.

## Global Instructions

Defined in advanced settings, these apply to all agents. Best for brand tone, shared DOs and DON'Ts, and global variables.