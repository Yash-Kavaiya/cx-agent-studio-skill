# Callbacks in CX Agent Studio

Callbacks are advanced Python hooks that let you observe, customize, and control agent behavior at specific points in the execution lifecycle.

Add callbacks via: **Agent settings → Add code → Select callback type → Provide Python code → Save**.

Multiple callbacks of the same type execute in the order defined.

---

## Python Runtime

All callback code runs in the CX Agent Studio Python runtime with access to injected classes. See the [Python runtime reference](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps/reference/python).

**Available imports:**
- `import json`
- `import re`
- `import random`
- `from datetime import datetime`
- Standard Python built-ins

**Injected classes (no import needed):**
`CallbackContext`, `LlmRequest`, `LlmResponse`, `Content`, `Part`, `FunctionCall`, `Tool`, `Blob`

---

## Execution Flow

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

## Callback Types

### 1. `before_agent_callback`

| | |
|---|---|
| **Execution** | Before the agent is invoked |
| **Purpose** | Setup resources, validate session state, skip agent invocation |
| **Arguments** | `CallbackContext` |
| **Return** | `Optional[Content]` — if set, agent is skipped and this Content is used |

```python
import random

def before_agent_callback(
    callback_context: CallbackContext
) -> Optional[Content]:
    """Setup username with random suffix before agent runs."""
    username = callback_context.variables.get("username", None)
    if not username:
        final_name = "Default Name"
    else:
        final_name = f"{username} {random.randint(1, 10)}"
    callback_context.variables["username"] = final_name
    # Return None to let agent run normally
    return None
```

---

### 2. `after_agent_callback`

| | |
|---|---|
| **Execution** | After the agent completes |
| **Purpose** | Cleanup, post-execution validation, modify final state, update response |
| **Arguments** | `CallbackContext` |
| **Return** | `Optional[Content]` — if set, replaces the agent's output |

```python
def after_agent_callback(
    callback_context: CallbackContext
) -> Optional[Content]:
    """Increment invocation counter for tracking agent usage."""
    if callback_context.agent_name == "Routing Agent":
        counter = callback_context.variables.get("counter", 0)
        callback_context.variables["counter"] = int(counter) + 1
    return None
```

---

### 3. `before_model_callback`

| | |
|---|---|
| **Execution** | Before the LLM request is sent |
| **Purpose** | Inspect/modify model request, input guardrails, skip LLM call, inject context |
| **Arguments** | `CallbackContext`, `LlmRequest` |
| **Return** | `Optional[LlmResponse]` — if set, LLM call is skipped and this response is used |

```python
def before_model_callback(
    callback_context: CallbackContext,
    llm_request: LlmRequest
) -> Optional[LlmResponse]:
    """
    Intercept the LLM call to force a specific function call.
    Returning LlmResponse skips the actual LLM — useful for guardrails,
    input validation, or serving cached/deterministic responses.
    """
    # Modify session state
    callback_context.variables['foo'] = 'baz'
    
    # Skip LLM and return a deterministic response
    return LlmResponse(
        content=Content(
            parts=[Part(function_call=FunctionCall(name="function_name", args={"arg": "value"}))],
            role="model"
        )
    )
```

**Session start greeting pattern:**
```python
def before_model_callback(
    callback_context: CallbackContext,
    llm_request: LlmRequest
) -> Optional[LlmResponse]:
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            return LlmResponse.from_parts(parts=[
                Part.from_text(text="Hello, how can I help you today?")
            ])
    return None
```

**Partial response (prefix while model works):**
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            response = LlmResponse.from_parts([Part.from_text("Hello, I'm Gemini, your personal AI assistant.")])
            response.partial = True  # Forces agent to continue processing after this response
            return response
    return None
```

---

### 4. `after_model_callback`

| | |
|---|---|
| **Execution** | After LLM response is received, before agent processes it |
| **Purpose** | Reformat responses, censor PII, parse structured data to variables, add payloads |
| **Arguments** | `CallbackContext`, `LlmResponse` |
| **Return** | `Optional[LlmResponse]` — if set, replaces the model's response |

```python
def after_model_callback(
    callback_context: CallbackContext,
    llm_response: LlmResponse
) -> Optional[LlmResponse]:
    """
    Approve the LLM response as-is by returning None.
    Return a new LlmResponse to replace/modify the output.
    """
    return None  # Pass through unchanged
```

**Enforce mandatory content (disclaimer):**
```python
DISCLAIMER = "THIS CONVERSATION MAY BE RECORDED FOR LEGAL PURPOSES."

def after_model_callback(callback_context, llm_response):
    if callback_context.variables.get("first_turn"):
        callback_context.variables["first_turn"] = False
        for part in callback_context.get_last_agent_output():
            if part.text and DISCLAIMER in part.text:
                return None  # Already included
        # Force the disclaimer
        return LlmResponse.from_parts(parts=[
            Part.from_text(DISCLAIMER),
            *llm_response.content.parts
        ])
    return None
```

---

### 5. `before_tool_callback`

| | |
|---|---|
| **Execution** | Before a tool call executes |
| **Purpose** | Inspect/modify tool arguments, authorization checks, mocking/caching |
| **Arguments** | `Tool`, `dict[str, Any]` (tool inputs), `CallbackContext` |
| **Return** | `Optional[dict[str, Any]]` — if set, tool is skipped and this dict is returned to the model |

```python
def before_tool_callback(
    tool: Tool,
    input: dict[str, Any],
    callback_context: CallbackContext
) -> Optional[dict[str, Any]]:
    """
    Validate and modify tool inputs before execution.
    Returning a dict skips the actual tool — useful for mocking or rate limiting.
    """
    callback_context.variables['foo'] = 'baz'
    input['input_arg'] = 'updated_val1'
    input['additional_arg'] = 'updated_val2'
    
    # Skip tool and return mocked result
    return {"result": "ok"}
```

---

### 6. `after_tool_callback`

| | |
|---|---|
| **Execution** | After a tool completes |
| **Purpose** | Modify tool response, save results to variables, audit logging |
| **Arguments** | `Tool`, `dict[str, Any]` (tool inputs), `CallbackContext`, `dict` (tool response) |
| **Return** | `Optional[dict]` — if set, overrides the tool response sent to the model |

```python
# Previous tool `get_user_info` returned: {"username": "Patrick", "fave_food": ["pizza"]}

def after_tool_callback(
    tool: Tool,
    input: dict[str, Any],
    callback_context: CallbackContext,
    tool_response: dict
) -> Optional[dict]:
    """Enrich tool response with additional data."""
    if tool.name == "get_user_info":
        tool_response["username"] = "Gary"
        tool_response["pet"] = "dog"
        return tool_response  # Override the response
    return None  # Pass through for other tools
```

---

## Custom Payloads (`custom_payloads`)

Supplementary JSON data included in agent responses — **not visible to the LLM**, only in the final API response.

**Use cases:**
- Agent escalation / human handoff routing
- Rich content widgets (chips, buttons, image carousels)
- Real-time UI updates

**Set using:** `before_model_callback` or `after_model_callback`

**Syntax:** `Part.from_json(data=json_string)` with mime_type `application/json`

### After-model payload example:
```python
import json

def after_model_callback(callback_context, llm_response):
    """Adds a custom payload to every text response."""
    if llm_response.content.parts[0].text is not None:
        payload_dict = {"custom_payload_key": "custom_payload_value"}
        payload_json = json.dumps(payload_dict)
        
        new_parts = []
        new_parts.append(Part(text=llm_response.content.parts[0].text))  # Keep original text
        new_parts.append(Part.from_json(data=payload_json))             # Append payload
        
        return LlmResponse(content=Content(parts=new_parts))
    return None
```

### Escalation payload example:
```python
import json

def has_escalate(llm_request):
    for content in llm_request.contents:
        for part in content.parts:
            if part.function_call and part.function_call.name == 'escalate':
                return True
    return False

def before_model_callback(callback_context, llm_request):
    if not has_escalate(llm_request):
        return None
    payload = {"escalate": "user requested escalation", "queue": "tier2_support"}
    return LlmResponse(content=Content(parts=[
        Part(text="Let me connect you with a specialist."),
        Part.from_json(data=json.dumps(payload))
    ]))
```

### UI chip payload example:
```python
import json

def after_model_callback(callback_context, llm_response):
    prefix = 'Available options are:'
    payload = {}
    for part in llm_response.content.parts:
        if part.text and part.text.startswith(prefix):
            payload['chips'] = [opt.strip() for opt in part.text[len(prefix):].split(',')]
            break
    
    if payload:
        new_parts = list(llm_response.content.parts)
        new_parts.append(Part.from_json(data=json.dumps(payload)))
        return LlmResponse.from_parts(parts=new_parts)
    return None
```

---

## Error Handling Patterns

### Transfer to another agent on tool failure:
```python
def before_model_callback(callback_context, llm_request):
    for part in llm_request.contents[-1].parts:
        if (part.has_function_response('authentication') and
                'error' in part.function_response.response['result']):
            return LlmResponse.from_parts(parts=[
                Part.from_text('Sorry, something went wrong. Let me transfer you to another agent.'),
                Part.from_agent_transfer(agent='escalation agent')
            ])
    return None
```

### Graceful session termination on failure:
```python
def before_model_callback(callback_context, llm_request):
    for part in llm_request.contents[-1].parts:
        if (part.has_function_response('authentication') and
                'error' in part.function_response.response['result']):
            return LlmResponse.from_parts(parts=[
                Part.from_text('Sorry, something went wrong. Please call back later.'),
                Part.from_end_session(reason='Failure during user authentication.')
            ])
    return None
```

### Post-session logging:
```python
def after_model_callback(callback_context, llm_response):
    for index, part in enumerate(llm_response.content.parts):
        if part.has_function_call('end_session'):
            tool_call = Part.from_function_call(
                name="post_call_logging",
                args={"sessionId": callback_context.session_id}
            )
            return LlmResponse.from_parts(
                parts=llm_response.content.parts[:index] + [tool_call] + llm_response.content.parts[index:]
            )
    return None
```

---

## `CallbackContext` Reference

| Attribute / Method | Description |
|-------------------|-------------|
| `callback_context.variables` | Session variables dict (read/write) |
| `callback_context.agent_name` | Name of the currently executing agent |
| `callback_context.session_id` | Unique session identifier |
| `callback_context.get_last_user_input()` | Parts from most recent user message |
| `callback_context.get_last_agent_output()` | Parts from most recent agent response |
| `callback_context.get_variable(name, default)` | Get a variable with default |
| `callback_context.set_variable(name, value)` | Set a variable |
