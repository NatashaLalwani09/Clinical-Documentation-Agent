# Clinical Documentation Agent

An AI agent that processes doctor–patient transcripts and generates structured clinical documentation — automatically.

## What it does

Given a raw conversation transcript, the agent:

1. **Parses** the dialogue into structured speaker turns
2. **Generates** a SOAP note (Subjective, Objective, Assessment, Plan)
3. **Maps** diagnoses to ICD-10-CM billing codes
4. **Validates** those codes against an official ~75,000-entry database
5. **Extracts** follow-up actions (medications, labs, imaging, referrals)

The agent also uses **judgment** — for non-clinical conversations (e.g., appointment scheduling), it skips the clinical tools entirely.

## What is an agent?

A regular LLM call is a single round-trip: you send a prompt, you get a response. An **agent** is different — it has access to **tools** (Python functions) and autonomously decides which tools to call, in what order, and when it has enough information to give a final answer. This loop is called the **agentic loop**.

```
User message
    │
    ▼
 LLM call ──► Tool call? ──Yes──► Execute tool ──► Append result to messages ──┐
                 │                                                              │
                No                                                   ◄──────────┘
                 │
                 ▼
          Final answer ✓
```

## Tech stack

| Library | Role |
|---|---|
| **LangChain** | Tool binding, message formatting, LLM interface |
| **GPT-4o** (`langchain_openai`) | Underlying model |
| **Pydantic** | Structured output validation |
| **ACI-Bench dataset** | Real doctor–patient transcripts for testing |

## Project structure

```
March - Agents/
├── clinical_agent.ipynb   # Main notebook — walk through from setup to output
└── Data/
    ├── icd10cm_codes_2026.txt   # Official ICD-10-CM code database (~75,000 codes)
    └── train.csv                # ACI-Bench clinical transcripts
```

## Setup

### Prerequisites

- Python 3.10+
- Jupyter Notebook or JupyterLab
- An OpenAI API key

### Install dependencies

```bash
pip install langchain langchain-openai pydantic pandas
```

### Configure your API key

The notebook imports `OPENAI_API_KEY` from a local `chatgpt_key.py` file. Create that file:

```python
# chatgpt_key.py
OPENAI_API_KEY = "sk-..."
```

Update the `sys.path.append(...)` line in the notebook to point to wherever you store this file.

### Run

Open `clinical_agent.ipynb` and run all cells top to bottom.

## Agent tools

| Tool | When called |
|---|---|
| `parse_transcript` | First, always — structures the raw dialogue |
| `generate_soap` | If the encounter is clinical |
| `code_diagnoses` | If the SOAP assessment contains diagnoses |
| `validate_icd10` | After coding — checks codes against the official database |
| `extract_actions` | If the plan contains actionable follow-up items |

## Example output

**Clinical transcript** (heart failure, hypertension, depression):

```
── Iteration 1 → parse_transcript
── Iteration 2 → generate_soap
── Iteration 3 → code_diagnoses
── Iteration 4 → validate_icd10
── Iteration 5 → extract_actions
── Iteration 6
Done: 5 tool calls

ICD-10 CODES
I50.9      Heart failure, unspecified               (high) ✓ validated ★ PRIMARY
I10        Essential (primary) hypertension         (high) ✓ validated
F32.1      Major depressive disorder, single episode (medium) ✓ validated
```

**Administrative transcript** (appointment rescheduling):

```
── Iteration 1 → parse_transcript
── Iteration 2
Done: 1 tool call
```

The agent correctly skips all clinical tools for a non-clinical conversation.

## Data

- **ICD-10-CM codes** — 2026 edition from the US Centers for Medicare & Medicaid Services (CMS)
- **ACI-Bench** — a benchmark dataset of real doctor–patient dialogues for clinical NLP research
