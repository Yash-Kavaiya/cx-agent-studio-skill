# Guardrails in CX Agent Studio

Guardrails protect agent applications with checks and balances for both model **input** and model **output**. Each agent application has default guardrails you can customize.

Access via the **guardrails button** on the right side of the agent builder.

---

## 1. Prompt Guard

Protects against prompt-based attacks like "ignore your instructions and...".

| Setting | Options |
|---------|---------|
| **Enable prompt guard** | Enable / Disable |
| **Outcomes** | Say exactly, Handoff to an agent, Generate a response, **Custom** (provide a custom security prompt for screening) |

**When to use:** Always enable for public-facing agents. Use "Custom" outcome to provide a fallback response aligned with your brand voice.

---

## 2. Blocklist

Prevents specific words or phrases in user input or agent responses.

| Setting | Options |
|---------|---------|
| **Matching method** | Whole words, Any mention, Regex pattern |
| **Block words and phrases** | Comma-separated list of blocked terms |
| **Blocked content from** | On user input, On agent response, or Both |
| **Outcomes** | Say exactly, Handoff to an agent, Generate a response |
| **Details** | Optional name and description |

**When to use:**
- Block competitor names from agent responses
- Block profanity or sensitive terms from user input
- Block regulatory-sensitive phrases

---

## 3. Safety

Enforces Responsible AI practices.

| Safety Level | Behavior |
|-------------|----------|
| **Relaxed** | Prioritize flexible generation and low latency. Never engage with illicit/harmful prompts. Always block explicitly harmful content. |
| **Balanced** | Prioritize safe and natural interactions. Always stop unsafe content. Never engage with harmful prompts. |
| **Strict** | Prioritize deep harm filtering. Never allow sensitive content. Always push back against harmful prompts. |

**Outcomes:** Say exactly, Handoff to an agent, Generate a response, **Custom** (individually adjust or disable specific safety levels per category)

**When to use:**
- Use **Balanced** for most customer service agents
- Use **Strict** for agents serving minors or highly regulated domains
- Use **Relaxed** for creative or internal tools where flexibility is needed

---

## 4. Rules

Build custom guardrails using natural language or Python code.

| Behavior | Description |
|----------|-------------|
| **Natural language** | Provide plain-text instructions for the rule |
| **Code** | Provide Python code as an `after_model_callback` — gives full programmatic control |

**Outcomes:** Say exactly, Handoff to an agent, Generate a response

**Details:** Optional name and description for the rule.

### Code-Based Rule Example (PII Redaction)

```python
import re

def after_model_callback(
    callback_context: CallbackContext,
    llm_response: LlmResponse
) -> Optional[LlmResponse]:
    """
    Redacts SSN and credit card numbers from model responses.
    """
    if not llm_response.content or not llm_response.content.parts:
        return None
    
    new_parts = []
    modified = False
    
    for part in llm_response.content.parts:
        if part.text:
            text = part.text
            # Redact SSN patterns (XXX-XX-XXXX)
            text = re.sub(r'\b\d{3}-\d{2}-\d{4}\b', '[SSN REDACTED]', text)
            # Redact credit card patterns (16 digits)
            text = re.sub(r'\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b', '[CC REDACTED]', text)
            if text != part.text:
                modified = True
            new_parts.append(Part(text=text))
        else:
            new_parts.append(part)
    
    if modified:
        return LlmResponse(content=Content(parts=new_parts, role="model"))
    return None
```

### Code-Based Rule Example (Topic Blocking)

```python
BLOCKED_TOPICS = ["competitor_name", "legal_advice", "medical_diagnosis"]

def after_model_callback(
    callback_context: CallbackContext,
    llm_response: LlmResponse
) -> Optional[LlmResponse]:
    """Blocks responses containing prohibited topics."""
    for part in llm_response.content.parts:
        if part.text:
            text_lower = part.text.lower()
            for topic in BLOCKED_TOPICS:
                if topic in text_lower:
                    return LlmResponse(content=Content(parts=[
                        Part(text="I'm not able to help with that topic. Can I assist you with something else?")
                    ], role="model"))
    return None
```

---

## Guardrail Decision Guide

| Need | Use |
|------|-----|
| Block prompt injection attacks | Prompt Guard |
| Block specific words/phrases | Blocklist |
| General content safety filtering | Safety |
| Custom logic (PII, topics, patterns) | Rules → Code (after_model_callback) |
| Brand-aligned fallback messaging | Prompt Guard (Custom outcome) |
| Transfer to human on violation | Any guardrail → Handoff outcome |
