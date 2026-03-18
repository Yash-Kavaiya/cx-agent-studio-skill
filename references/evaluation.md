# CX Agent Studio Evaluation Reference

Evaluations automate testing, catch regressions, and measure agent quality through test cases.

---

## Test Case Types

### 1. Scenario Test Cases
- AI simulates user conversations based on a high-level **user goal** (e.g., "Securely book a specific room at a chosen hotel")
- Explores edge cases without manually writing every conversation path
- Successful scenarios can be saved as **Golden Conversations**

**Scenario Expectations:**
- Message expected from end-user or agent
- Tool call with expected inputs/outputs
- Conditions: Must have, Must not have, After tool call, Variable value

### 2. Golden Test Cases
- Specific, ideal conversation paths for **regression testing**
- Evaluates whether agent behavior and tool calls match the expected path

**Creation methods:**
- Simulator (record a conversation)
- Conversation history
- From scratch
- Simulated scenario
- CSV batch upload

**Golden Expectations:**
- **Message** — agent response matches expectation semantically
- **Tool Call** — agent calls specific tool with expected arguments and response
- **Agent Handoff** — conversation transfers to human or another bot

---

## Evaluation Metrics

| Metric | Test Type | Description |
|--------|-----------|-------------|
| **Tool Correctness** | Golden + Scenario | % of expected parameters matched. Unexpected calls fail golden tests but don't affect this score |
| **User Goal Satisfaction** | Scenario | Binary (0=no, 1=yes). Did simulated user believe their goal was achieved? |
| **Hallucinations** | Golden + Scenario | Identifies claims not justified by context (variables, history, tools, instructions). Only computed for turns with tool calls |
| **Semantic Match** | Golden | How well observed agent utterance matches expected (0=contradictory → 4=consistent) |
| **Scenario Expectations** | Scenario | Satisfactory behavior met (0=no, 1=yes). Includes tool call and agent response expectations |
| **Task Completion** | Scenario | Joint measure of user goal achievement and correct agent behavior |

---

## AI Issue Discovery

When running **3 or more evaluations**, enable **Find issues with AI** to:
- Generate a downloadable `loss_report`
- Interact with a Gemini helper via "Ask Gemini" to explain issues and suggest fixes

---

## Personas

Simulated user personas customize scenario tests with age, location, motivations, and behavior patterns — ensuring the agent interacts appropriately with expected user types.

- Created under **"Persona management"**
- Applied by selecting the persona when starting an evaluation run

**Example persona configuration:**
```
Name: Tech-savvy millennial
Age: 28
Location: San Francisco, CA
Behavior: Prefers quick answers, skips pleasantries, uses abbreviations
Goal: Efficiently resolve issues without transfers
```

---

## Importing Evaluations

Import evaluation test cases via:
- **Console:** Agent app → Import evaluations
- **REST API:** `POST /v1beta/{parent=.../apps/*}:importEvaluations`
- **CSV Batch Upload:** For golden test cases

---

## Scheduled Evaluation Runs

Automate regular evaluation runs:
- **REST:** `POST /v1beta/{parent=.../apps/*}/scheduledEvaluationRuns`
- Useful for CI/CD pipelines — run evals on every deployment

---

## Workflow: Creating and Running Evaluations

```
1. Define test cases (Golden or Scenario)
   │
   ├── Golden: ideal conversation paths, specific tool calls
   └── Scenario: user goal, expectations, persona
   
2. Run evaluation
   │
   ├── Automated: Scheduled evaluation runs
   └── Manual: Console → Run evaluation

3. Review results
   │
   ├── Check per-metric scores
   ├── Review failed turns in detail
   └── Enable "Find issues with AI" (3+ evals)

4. Iterate
   │
   ├── Fix instructions / tools based on failures
   ├── Create new version snapshot when passing
   └── Re-run evals to validate
```

---

## REST API Reference

| Operation | Endpoint |
|-----------|----------|
| Create evaluation | `POST /v1beta/{parent=.../apps/*}/evaluations` |
| Run evaluation | `POST /v1beta/{app=.../apps/*}:runEvaluation` |
| List evaluations | `GET /v1beta/{parent=.../apps/*}/evaluations` |
| Get evaluation | `GET /v1beta/{name=.../evaluations/*}` |
| List results | `GET /v1beta/{parent=.../evaluations/*}/results` |
| Generate from conversation | `POST /v1beta/{conversation=.../conversations/*}:generateEvaluation` |
| Upload audio for golden eval | `POST /v1beta/{name=.../evaluations/*}:uploadEvaluationAudio` |
| Create evaluation dataset | `POST /v1beta/{parent=.../apps/*}/evaluationDatasets` |
| Create scheduled run | `POST /v1beta/{parent=.../apps/*}/scheduledEvaluationRuns` |

---

## Sample Agent Evaluations (Cymbal Retail)

When you import the Cymbal retail sample agent, many golden and scenario-based evaluations are pre-loaded and start executing immediately. Use them to understand what good evaluations look like.
