# Tools in CX Agent Studio

Tools connect agents to external systems or inline code — to fetch, update, format, or analyze information.

## Creating a Tool

1. Click the **tool icon** on the right side of the agent builder.
2. Select a tool type.
3. Provide required information. Use **snake_case** for tool names.
4. Tool descriptions are sent to agent models — write them carefully.

## Adding a Tool to an Agent

1. Hover over an agent → click **Add tool** → select a tool.
2. Describe how the tool should be used in the agent's instructions.

## Testing a Tool

Open the tool panel → click **Test Tool** → provide input → click **Submit**.
Then test end-to-end via the **Preview agent** simulator.

---

## Tool Types

### 🔧 Python Code Tools
Inline Python code with full control over logic. Can call other tools, access `context` object, make HTTP requests.
→ See [Python Tools detail](#python-code-tools) below.

### 🌐 OpenAPI Tools
Connect to an external API using an OpenAPI 3.0 schema. The agent calls the API on your behalf.
- **Limitation:** Each OpenAPI tool can only have one operation/function.
- Supports authentication: Service Agent ID token, Service Account, OAuth, API key (via Secret Manager).
- Can inject session context variables using `x-ces-session-context`.
→ See [OpenAPI Tools detail](#openapi-tools) below.

### 📱 Client Function Tools
Executed on the **client side** (not by the agent). The conversation is blocked server-side until the client provides a response.

Use for: UI/UX actions, local device data, private APIs only reachable from the client.

```json
// Session response — agent signals client to call a function:
"toolCalls": [{ "id": "<id>", "tool": "<resource>", "args": {"zip_code": "92031"}, "displayName": "get_nearest_store" }]

// Client responds with tool output:
"toolResponses": [{ "displayName": "get_nearest_store", "id": "<id>", "tool": "<resource>", "response": {"name": "Store A", "address": "..."} }]
```

### 🗄️ Data Store Tools
AI-generated responses from website content or uploaded documents.

- **Website data store** — search from specific websites
- **Cloud storage data store** — RAG over unstructured documents / FAQs
- **File search** — upload files directly as a RAG knowledge base

### 🔍 Google Search Tools
Ground agent responses with live Google Search results.

### 🔌 Integration Connector Tools
Use pre-configured Connections (e.g., SAP, databases) via Integration Connectors.

### 🤖 MCP Tools
Connect to a Model Context Protocol (MCP) server.

### 💼 Salesforce / ServiceNow Tools
Pre-built connectors to Salesforce and ServiceNow instances.

### ⚙️ System Tools
Built-in tools for common tasks (e.g., `end_session`, `customize_response`).

### 🖼️ Widget Tools
Rich UI interaction tools for chat interfaces (chips, buttons, rich content).

---

## Python Code Tools (Detail)

### Name and Docstring
- Tool name and main function name must **match exactly** in snake_case.
- The function's **docstring is the tool description** sent to the model. Treat it as an extension of prompting.

```python
def get_order_status(order_id: str) -> dict:
    """
    Retrieves the current status and estimated delivery date for a given order.
    Use this when the user asks about their order, shipping, or delivery status.
    
    Args:
        order_id: The unique order identifier (e.g., 'ORD-12345').
    
    Returns:
        A dictionary with 'status', 'estimated_delivery', and 'tracking_url'.
    """
    # implementation
```

### The `context` Object (ToolContext)
Globally available — no import needed, no parameter needed.

| Attribute | Type | Description |
|-----------|------|-------------|
| `context.state` | dict | Session state — read and write variables |
| `context.user_content` | Content | Most recent user message with `.parts` and `.role` |
| `context.events` | list | Full conversation history |
| `context.session_id` | str | Unique session identifier |
| `context.invocation_id` | str | Unique ID for current turn |
| `context.agent_name` | str | Name of the agent executing the tool |
| `context.function_call_id` | str | Unique ID for this specific tool call |

### Reading/Writing Session Variables

```python
# Read with default
profile = context.state.get("customer_profile", {})

# Write
context.state["customer_profile"] = {"email": "user@example.com", "loyalty_points": 100}
```

### Calling Other Tools from Python

```python
# Call an OpenAPI tool from within a Python tool
# Syntax: tools.<tool_name>_<endpoint_name>({args})
result = tools.crm_service_get_cart_information({})

# Chain multiple tools
def python_wrapper(arg_a: str, arg_b: str) -> dict:
    """Makes sequential API calls."""
    res1 = tools.tool_1({"arg_a": arg_a, "arg_b": arg_b})
    res2 = tools.tool_2(res1.json())
    res3 = tools.tool_3(res2.json())
    return res3.json()
```

### Making HTTP Requests

Use `ces_requests` (inspired by `requests` library, available without import):

```python
def get_random_fact() -> str | None:
    """Fetches a random fact from a public API."""
    url = "https://uselessfacts.jsph.pl/api/v2/facts/random"
    try:
        res = ces_requests.get(url=url)
        # For POST: ces_requests.post(url=url, json=body, headers=headers)
        res.raise_for_status()
        return res.json().get("text")
    except:
        return None
```

### Full Example: Get and Update Variables with Pydantic

```python
from pydantic import BaseModel, Field
from typing import Optional, Dict, Any

class CustomerProfile(BaseModel):
    email: Optional[str] = None
    loyalty_points: int = Field(default=0, ge=0)
    plan: str = "Standard"

def update_customer_loyalty_points(points_to_add: int) -> Dict[str, Any]:
    """
    Adds loyalty points to the customer's profile and returns the updated profile.
    
    Args:
        points_to_add: Number of loyalty points to add.
    
    Returns:
        Dictionary with updated customer profile information.
    """
    current_data = context.state.get("customer_profile", {})
    profile = CustomerProfile(**current_data)
    profile.loyalty_points += points_to_add
    print(f"Updated loyalty points to: {profile.loyalty_points}")
    context.state["customer_profile"] = profile.model_dump()
    return context.state["customer_profile"]
```

---

## OpenAPI Tools (Detail)

### Schema Example

```yaml
openapi: 3.0.0
info:
  title: Simple Pets API
  version: 1.0.0
servers:
  - url: 'https://api.pet-service-example.com/v1'
paths:
  /pets/{petId}:
    get:
      summary: Return a pet by ID.
      operationId: getPet
      parameters:
        - in: path
          name: petId
          required: true
          schema:
            type: integer
```

### Injecting Session Context Variables

Use `x-ces-session-context` to inject session data without model involvement:

| Value | Description |
|-------|-------------|
| `$context.session_id` | Session ID |
| `$context.project_id` | GCP project ID |
| `$context.app_id` | Agent application ID |
| `$context.variables` | All session variables as object |
| `$context.variables.variable_name` | Specific variable value |

```yaml
parameters:
  - name: X-SESSION
    in: header
    required: true
    schema:
      type: string
    x-ces-session-context: $context.session_id
```

### Authentication Options

| Method | Description |
|--------|-------------|
| **Service Agent ID token** | Auto-generated CES service agent token in `Authorization` header |
| **Service Account** | SA email → access token in `Authorization` header |
| **OAuth** | OAuth config for Google or other services |
| **API key** | Key name + location (header/query) + Secret Manager secret version |

### Security Best Practices
- Deploy agent applications in a **dedicated project** separate from critical resources
- Implement **VPC Service Controls** to limit service account scope
- Follow **least privilege** — only grant minimum required IAM roles

---

## Tool Best Practices

### Names and Descriptions
- Use semantically meaningful snake_case names
- Write high-quality, descriptive docstrings/descriptions
- Remove unused tools from agent nodes
- Use distinct, non-similar names across tools
- Use flattened parameter structures (not nested)
- Avoid abbreviations: use `phone_number` not `pnum`, `first_name` not `fn`

### Wrap APIs with Python Tools (Context Engineering)
Don't expose raw OpenAPI responses to the agent — filter to only relevant fields:

```python
def schedule_appointment(date: str, service_type: str) -> dict:
    """Schedules a service appointment and returns confirmation details."""
    res = complicated_external_api_call(date=date, service=service_type)
    # Return only what the agent needs
    return {
        "appointment_time": res.json()["appointment_time"],
        "appointment_location": res.json()["appointment_location"],
        "confirmation_id": res.json()["confirmation_id"],
    }
```

### Chain Tool Calls in Python (Not in Instructions)
❌ **Bad:** Instruct the agent to call tool_1 → tool_2 → tool_3 (4 model calls, error-prone)

✅ **Good:** One Python wrapper that internally calls the chain:
```python
def process_order(order_id: str) -> dict:
    """Validates, processes, and confirms an order end-to-end."""
    validation = tools.validate_order({"order_id": order_id})
    processing = tools.process_payment(validation.json())
    confirmation = tools.send_confirmation(processing.json())
    return confirmation.json()
```
