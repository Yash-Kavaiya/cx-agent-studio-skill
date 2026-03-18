# Variables in CX Agent Studio

Variables store and retrieve runtime conversation data, enabling agents to remember information across turns for more contextual interactions.

## Variable Data Fields

| Field | Description |
|-------|-------------|
| **Name** | Variable name in snake_case |
| **Type** | Data type (see below) |
| **Default value** | Initial value |
| **Description** | Optional human-readable description |

## Variable Types

| Type | Description | Example |
|------|-------------|---------|
| **Text** | String values | `"John Smith"` |
| **Number** | Numeric values | `42`, `3.14` |
| **Yes/No** | Boolean values | `true`, `false` |
| **Custom Object** | You provide a JSON schema | `{"email": "...", "loyalty_points": 0}` |
| **List** | Comma-delimited list | `"item1,item2,item3"` |

## Referencing Variables in Instructions

Use `{variable_name}` braces syntax in agent instructions:

```
CURRENT CUSTOMER: {username}
Your account tier is: {account_tier}
Cart items: {cart_items}
```

### How Variable References Actually Work

When you reference `{variable_name}` in instructions, it does **not** do a direct text substitution. Instead:

1. The system **tracks** the variable while the agent is active.
2. When the variable's value is **updated** (by a tool or callback), the new value is **appended to conversation history**.
3. The model sees the updated value in conversation history and uses it to generate accurate responses.

## Updating Variable Values

**The agent itself cannot update variables directly.** Only **tools** and **callbacks** can:

### Python Tools — `context.state`
```python
# Read
value = context.state.get("variable_name", default_value)

# Write
context.state["variable_name"] = new_value

# With Pydantic model
context.state["customer_profile"] = profile.model_dump()
```

### Callbacks — `callback_context.variables`
```python
# Read
value = callback_context.variables.get("variable_name", default_value)

# Write  
callback_context.variables["variable_name"] = new_value
```

> ⚠️ **Important:** Python tools use `context.state` and callbacks use `callback_context.variables`. These access the same underlying ADK session state.

## Session Persistence

Variables persist across the **entire session** — every turn, every agent. They are reset when a new session starts.

## Passing Variables to OpenAPI Tools

Use `x-ces-session-context` in your OpenAPI schema to inject variables without the model needing to recall and pass them:

```yaml
parameters:
  - name: customer_id
    in: query
    description: Customer ID from session
    required: true
    schema:
      type: string
    x-ces-session-context: $context.variables.customer_id
  - name: X-SESSION
    in: header
    schema:
      type: string
    x-ces-session-context: $context.session_id
```

## Dynamic Prompt Variables Pattern

You can use variables to create dynamic prompts that change agent behavior at runtime:

**Variables:**
```
current_instructions = "You are Gemini and you work for Google."
lawyer_instructions  = "You are a lawyer and tell dad-joke style jokes with a lawyer edge."
pirate_instructions  = "You are a pirate and tell jokes in pirate style."
username             = "Unknown"
```

**Instructions:**
```
The current user is: {username}
Follow the current instruction set below exactly.
{current_instructions}
```

**Callback to switch persona:**
```python
def before_model_callback(callback_context, llm_request):
    username = callback_context.variables.get("username")
    if username == "Jenn":
        new_instructions = callback_context.variables.get("pirate_instructions")
        callback_context.variables["current_instructions"] = new_instructions
    elif username == "Gary":
        new_instructions = callback_context.variables.get("lawyer_instructions")
        callback_context.variables["current_instructions"] = new_instructions
    return None
```

## Variable Scoping

- Variables are **session-scoped** — shared across all agents in the session
- Changes made by any tool or callback are immediately visible to all subsequent agent turns
- Variables do **not** persist between sessions

## Common Variable Patterns

### Boolean flag (track first turn)
```python
# In callback
if callback_context.variables.get("first_turn", True):
    callback_context.variables["first_turn"] = False
    # Do first-turn logic
```

### Counter
```python
def after_agent_callback(callback_context):
    if callback_context.agent_name == "Routing Agent":
        counter = callback_context.variables.get("counter", 0)
        callback_context.variables["counter"] = int(counter) + 1
```

### Custom object (customer profile)
```python
# Define schema as Custom Object variable type in console
# Access in Python tool:
profile = context.state.get("customer_profile", {})
profile["loyalty_points"] = profile.get("loyalty_points", 0) + 10
context.state["customer_profile"] = profile
```

## ADK Documentation

Variables use [ADK context state](https://google.github.io/adk-docs/context/#managing-state) under the hood.
