---
name: cx-agent-studio
description: Guide and instructions for using Google Customer Experience Agent Studio (CX Agent Studio). Use when creating conversational agents, writing or structuring instructions with XML tags, setting up few-shot examples, or building evaluation test cases (Golden or Scenario).
---

# CX Agent Studio

Customer Experience Agent Studio (CX Agent Studio) is a minimal code conversational agent builder built on the Agent Development Kit (ADK), representing the evolution of Dialogflow CX.

## Core Capabilities

- **AI-Augmented Building**: Generate agents using Gemini with a 1-2 sentence goal.
- **Bi-directional Streaming**: Ultra-low latency voice interactions.
- **Asynchronous Tool Calling**: Maintains natural conversation flow during backend calls.

## Quick Actions

### 1. Generating an Agent with AI
To generate an agent automatically:
- Provide a clear 1-2 sentence goal.
- Optionally provide up to 5 knowledge documents (under 8MB total) like FAQs or tool catalogs.
*Note: Only works for the root agent and empty agents.*

### 2. Writing Agent Instructions
Agent instructions guide the model's behavior, persona, and tool/agent usage.
- **Syntax References**:
  - Variables: `{variable_name}`
  - Tools: `{@TOOL: tool_name}`
  - Sub-Agents: `{@AGENT: Agent Name}`
- **For complex instructions or recommended XML formatting**, read: `references/instructions.md`

### 3. Agent Evaluation
Evaluation ensures agent performance via automated test cases.
- **Scenario Test Cases**: AI-generated simulated user conversations based on a user goal.
- **Golden Test Cases**: Specific, ideal conversation paths for regression testing.
- **For detailed evaluation metrics, personas, and test case creation**, read: `references/evaluation.md`
