# LAB 1 — Meeting Action Agent

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/LLM-AI-INDIA/ANTHROPIC-CCAF-LABS/blob/main/LAB%201/meeting_action_agent_colab.ipynb)

A multi-agent system built on the **Claude Agent SDK** that turns a raw meeting
transcript into a structured Markdown report with three sections:

- **Decisions Made**
- **Action Items** (owner, task, deadline, confidence)
- **Follow-up Emails** (one personalized email per attendee)

## Architecture

One orchestrator dispatches three specialized subagents **in parallel**, each
running in its own isolated context window:

```
                +-------------------+
                |   Orchestrator    |   (reads the transcript once)
                +---------+---------+
                          |
              Task tool, dispatched in parallel
                          |
   +----------------------+----------------------+
   |                      |                      |
   v                      v                      v
+--------------------+ +-----------------------+ +-------------------+
| decision-extractor | | action-item-extractor | | follow-up-drafter |
|  (own context)     | |   (own context)       | |   (own context)   |
+--------------------+ +-----------------------+ +-------------------+
```

Each subagent stays focused on its single job. The orchestrator never sees
their internal reasoning — only their final Markdown output — and the subagents
never see each other's work.

## How to run

### Option A — Google Colab (no install)

1. Open `meeting_action_agent_colab.ipynb` in Colab.
2. Get a free Anthropic API key at https://console.anthropic.com/settings/keys
   (must start with `sk-ant-`).
3. Run the cells from top to bottom. The second cell will prompt you to paste
   your key (input is hidden, never saved to the notebook).
4. The final report renders inline at the bottom of the notebook.

### Option B — Local Python

```bash
pip install claude-agent-sdk python-dotenv
export ANTHROPIC_API_KEY=sk-ant-...
python main.py samples/meeting_transcript.txt
```

## What you'll see when it runs

```
Analyzing transcript...

  -> dispatching subagent: decision-extractor
  -> dispatching subagent: action-item-extractor
  -> dispatching subagent: follow-up-drafter
  -> writing file: output/meeting_report.md

--- run finished ---
cost: $0.XXXX USD
tokens (input/output): ... / ...
```

## What's in this folder

| File | Purpose |
|------|---------|
| `meeting_action_agent_colab.ipynb` | Self-contained Colab notebook — open and run |
| `README.md` | This file |

## Try your own transcript

Replace the `TRANSCRIPT` variable in the notebook with your own meeting text and
re-run the analysis cell. Anything in plain English works — no formatting
required.

## Customization

| To change… | Edit… |
|------------|-------|
| Make it cheaper | Change `model="sonnet"` to `model="haiku"` in `build_options()` |
| Make it stronger | Change to `model="opus"` |
| Add a new subagent | Add another `AgentDefinition` + a new prompt + mention it in `ORCHESTRATOR_PROMPT` |
| Change the report format | Edit the workflow section of `ORCHESTRATOR_PROMPT` |
