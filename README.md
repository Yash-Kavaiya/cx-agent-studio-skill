# CX Agent Studio Skill

A comprehensive OpenClaw skill for navigating, building, and optimizing Google Customer Experience Agent Studio (CX Agent Studio) conversational agents.

## 🚀 Published Skill
This skill is officially published and available to install via the ClawHub Registry:
**[View on Skills.sh](https://skills.sh/skills/cx-agent-studio)**

### Installation
To install this skill directly using OpenClaw, run:
```bash
npx clawhub install cx-agent-studio
```

## Overview
Customer Experience Agent Studio (CX Agent Studio) is a minimal code conversational agent builder built on the Agent Development Kit (ADK). This skill provides OpenClaw with deep context and best practices for:
- Quick access to building guides for AI-Augmented agents.
- Best practices and references for **Instructions** (XML-structured prompts, inline few-shot examples).
- Overviews for **Tools** (Python API wrappers, MCP integrations, Data Stores).
- Advanced references for Python **Callbacks** (`before_model_callback`, `custom_payloads`, etc.).
- Guidelines on **Guardrails**, **Variables**, and **Agent Flows**.

## Structure
- `SKILL.md` - The primary entry point containing core instructions and commands for the AI.
- `references/` - In-depth markdown guides detailing specific CX Agent Studio topics.