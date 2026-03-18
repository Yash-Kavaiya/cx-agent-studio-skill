# Best Practices and Patterns in CX Agent Studio

Source: [Google Cloud Docs](https://docs.cloud.google.com/customer-engagement-ai/conversational-agents/ps/best-practices)

---

## General

### Start Simple
Build simple use cases first, validate them, then scale to complex ones.

### Specific Instructions
- Unambiguous, well-organized, grouped by topic
- Easy for a human to follow
- Use **Restructure instructions** to format into XML for higher reliability

---

## Tools

### Wrap APIs with Python Tools (Context Engineering)
External OpenAPI specs often return 100+ fields when you only need 3. Exposing all of them wastes tokens and confuses the model.

✅ **Good:** Filter to only what the agent needs:
```python
def schedule_appointment(date: str, service_type: str) -> dict:
    """Schedules a service appointment. Returns only relevant confirmation fields."""
    res = complicated_external_api_call(date=date, service=service_type)
    return {
        "appointment_time": res.json()["appointment_time"],
        "appointment_location": res.json()["appointment_location"],
        "confirmation_id": res.json()["confirmation_id"],
    }
```

### Use Tools and Callbacks for Deterministic Behavior
- **Callbacks** are the best option — they run outside the agent's purview entirely
- **Tool internals** are deterministic, but the agent's decision to call them is not (hallucination risk)

### Chain Tool Calls in Code, Not Instructions

❌ **Bad pattern** (4 model calls, hallucination-prone):
```
Instructions: "First call tool_1, then pass result to tool_2, then call tool_3..."
```

✅ **Good pattern** (1 model call, deterministic):
```python
def process_full_order(order_id: str) -> dict:
    """Validates, processes, and confirms an order in one call."""
    res1 = tools.validate_order({"order_id": order_id})
    res2 = tools.process_payment(res1.json())
    res3 = tools.send_confirmation(res2.json())
    return res3.json()
```

Alternatively, use `after_tool_callback` to chain remaining tools after the first.

### Clear and Distinct Tool Definitions
- Different tools must have noticeably distinct names
- Remove unused tools from agent nodes
- Use snake_case, descriptive names: `phone_number` not `pnum`, `first_name` not `fn`
- Use **flat** parameter structures — nested structures increase hallucination probability

---

## Development Workflow

### Collaboration
- **Third-party version control:** Use import/export to sync with Git. Define clear owners, review processes, and merge criteria (e.g., eval results must pass).
- **Built-in version control (snapshots):** Create snapshots at milestones (after passing evals, before new feature work).

### Use Versions Often
- Create a version after every 10–15 major changes
- Use semantic names: `v1.0.0`, `pre-prod-instruction-changes`, `prod-ready-for-testing`
- Add descriptions like commit messages
- Versions are immutable — you can always roll back

### End-to-End Testing
Include integration testing with external systems in your development process.

---

## Evaluations
Use evaluations to keep agents reliable. Set expectations on agents and the APIs they call. See `evaluation.md`.

---

## Session Handling Patterns

### Deterministic Greeting on Session Start
Save model calls, tokens, and latency with a static greeting:
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            return LlmResponse.from_parts(parts=[
                Part.from_text(text="Hello, how can I help you today?")
            ])
    return None
```

### Prefix Message While Model Processes (Reduce Perceived Latency)
Send a quick branded greeting while the model works in parallel:
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            response = LlmResponse.from_parts([
                Part.from_text("Hello, I'm Gemini, your personal AI assistant.")
            ])
            response.partial = True  # Continue processing after this response
            return response
    return None
```

### Enforce Mandatory Content (Legal Disclaimers)
```python
DISCLAIMER = "THIS CONVERSATION MAY BE RECORDED FOR LEGAL PURPOSES."

def after_model_callback(callback_context, llm_response):
    if callback_context.variables.get("first_turn"):
        callback_context.variables["first_turn"] = False
        for part in callback_context.get_last_agent_output():
            if part.text and DISCLAIMER in part.text:
                return None  # Agent already included it
        # Force it if agent missed it
        return LlmResponse.from_parts(parts=[
            Part.from_text(DISCLAIMER),
            *llm_response.content.parts
        ])
    return None
```

### Post-Call Logging on Session End
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

### Custom No-Input Response
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text and "no user activity detected" in part.text:
            return LlmResponse.from_parts(parts=[
                Part.from_text(text="Hi, are you still there?")
            ])
    return None
```

---

## Client-Side Integration Patterns

### Use Custom Payloads to Drive UI
Transform plain text options into interactive elements (chips, buttons):
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

### Real-Time UI Updates with Partial Responses
Send a JSON payload when a tool completes, before the model generates the final text:
```python
import json

def before_model_callback(callback_context, llm_request):
    if llm_request.contents[-1].parts[-1].has_function_response('update_order'):
        order_state = llm_request.contents[-1].parts[-1].function_response.response['result']['order_state']
        response = LlmResponse.from_parts([Part.from_json(data=json.dumps(order_state))])
        response.partial = True  # Keep processing for final text response
        return response
    return None
```

### Displaying Markdown and HTML
The simulator supports Markdown and HTML. Use agent instructions to direct formatting:
```xml
<role>
  You are a Markdown Display Assistant. Generate HTML-formatted content including
  images (<img>), videos (<video>), and hyperlinks (<a>) based on user requests.
</role>
```

---

## Voice and Audio Channel Patterns

### Play Prerecorded Audio on Session Start (Non-Interruptable)
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            return LlmResponse.from_parts(parts=[
                Part.from_json(data='{"audioUri": "gs://bucket/greeting.wav", "interruptable": false}')
            ])
    return None
```

### Play Hold Music During Slow Tools (No Barge-In)
```python
def after_model_callback(callback_context, llm_response):
    for index, part in enumerate(llm_response.content.parts):
        if part.has_function_call("slow_tool"):
            music = Part.from_json(data='{"audioUri": "gs://bucket/hold-music.mp3", "cancellable": true}')
            return LlmResponse.from_parts(
                parts=llm_response.content.parts[:index] + [music] + llm_response.content.parts[index:]
            )
    return None
```

### Play Music During Async Tool (Allow Barge-In)
```python
def before_model_callback(callback_context, llm_request):
    for part in llm_request.contents[-1].parts:
        if part.has_function_response("async_tool"):
            return LlmResponse.from_parts(parts=[
                Part.from_text("I'm submitting your order, it may take a while."),
                Part.from_json(data='{"audioUri": "gs://bucket/music.mp3", "cancellable": true}')
            ])
    return None
```

### Disable Barge-In for Critical Responses
```python
def before_model_callback(callback_context, llm_request):
    for part in callback_context.get_last_user_input():
        if part.text == "<event>session start</event>":
            return LlmResponse.from_parts(parts=[
                Part.from_customized_response(
                    content="Hello, I'm Gemini. Please listen to the following: <LEGAL_DISCLAIMER>",
                    disable_barge_in=True
                ),
                Part.from_text("How can I help you today?")
            ])
    return None
```

**Notes on audio:**
- Supported formats: Linear16, mulaw, alaw
- If audio file is in a different GCP project, grant `storage.objects.get` to `service-<project_number>@gcp-sa-ces.iam.gserviceaccount.com`
- Use `interruptable: false` to prevent user interruption
- Use `cancellable: true` to stop music when agent generates a new response

---

## Error Handling Patterns

### Transfer to Another Agent on Tool Failure
```python
def before_model_callback(callback_context, llm_request):
    for part in llm_request.contents[-1].parts:
        if (part.has_function_response('authentication') and
                'error' in part.function_response.response['result']):
            return LlmResponse.from_parts(parts=[
                Part.from_text('Sorry, something went wrong. Let me transfer you.'),
                Part.from_agent_transfer(agent='escalation agent')
            ])
    return None
```

### Gracefully Terminate on Critical Failure
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

---

## Context and Variables Patterns

### Pass Variables to OpenAPI Tools via `x-ces-session-context`
```yaml
parameters:
  - name: session_id
    in: path
    required: true
    schema:
      type: string
    x-ces-session-context: $context.session_id
  - name: test_variable
    in: query
    required: true
    schema:
      type: string
    x-ces-session-context: $context.variables.test_variable
```

Available context values: `$context.project_id`, `$context.project_number`, `$context.location`, `$context.app_id`, `$context.session_id`, `$context.variables`, `$context.variables.<name>`

### Dynamic Prompts Using Variables + Callbacks
```python
# Variables: current_instructions, lawyer_instructions, pirate_instructions, username
# Instructions: {current_instructions}

def before_model_callback(callback_context, llm_request):
    username = callback_context.variables.get("username")
    if username == "Jenn":
        callback_context.variables["current_instructions"] = callback_context.variables.get("pirate_instructions")
    elif username == "Gary":
        callback_context.variables["current_instructions"] = callback_context.variables.get("lawyer_instructions")
    return None
```
