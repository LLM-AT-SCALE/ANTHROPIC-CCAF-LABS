# LAB 1 — Multi-Agent Project Generator (Colab Edition)

A self-contained system prompt that turns **Claude Code** into a project designer and notebook author. You hand it a problem statement; it designs a multi-agent solution built on the **Claude Agent SDK** and ships you a single Google Colab notebook (`.ipynb`) that runs the system end-to-end.

The default problem is a **Meeting Action Agent** — paste in a meeting transcript, get back a Markdown report with decisions, action items, and follow-up emails — but the spec is generic, so you can override it with any text-in / structured-Markdown-out task (code reviewer, paper summarizer, support ticket triager, etc.).

---

## What's in this folder

| File | What it is |
|------|------------|
| [`LAB1_COLAB_SYSTEM_PROMPT.md`](./LAB1_COLAB_SYSTEM_PROMPT.md) | The system prompt you paste into Claude Code. Locks the architecture; leaves cell design to Claude. |
| `README.md` | This file. |

---

## How it works (the full loop)

```
   You                Claude Code            Google Colab
    |                      |                      |
    |--paste prompt------->|                      |
    |                      |--asks 3-4 setup Qs   |
    |<---------------------|                      |
    |                      |--designs subagents   |
    |                      |--writes .ipynb       |
    |<--hands off .ipynb---|                      |
    |                                             |
    |--upload .ipynb----------------------------->|
    |--paste API key + pick input file----------->|
    |<--rendered Markdown report------------------|
```

Claude Code **does not run the notebook**. It only generates it. Execution happens in Colab, under your Google account, with your Anthropic API key.

---

## Prerequisites

1. **Claude Code** installed locally — <https://claude.com/claude-code>
2. An **Anthropic API key** — get one at <https://console.anthropic.com/settings/keys>. It will start with `sk-ant-...`.
3. A **Google account** for Colab.

---

## How to use it

### Option A — Generate your own notebook with Claude Code (recommended)

1. **Open Claude Code** in any working directory.
2. **Copy the entire contents** of [`LAB1_COLAB_SYSTEM_PROMPT.md`](./LAB1_COLAB_SYSTEM_PROMPT.md) and paste it as your first message.
3. Claude Code will walk you through a short interactive flow:
   - Pick the default problem (Meeting Action Agent) **or** describe a different one.
   - Pick a target folder.
   - Pick a model (`sonnet` / `haiku` / `opus`).
   - Review the subagent topology it designed and approve.
4. Claude Code writes one `.ipynb` file to your chosen folder and hands it off.
5. **Upload that `.ipynb` to Google Colab** — `File → Upload notebook` at <https://colab.research.google.com>.
6. Run cells top-to-bottom:
   - Install cell — installs `claude-agent-sdk` and `nest-asyncio`.
   - API-key cell — paste your `sk-ant-...` key into the masked prompt.
   - Pick-your-input cell — click **"Choose Files"** and pick any `.txt` (or UTF-8 text) file.
   - Run cell — watch the live trace stream as the subagents fire **in parallel**.
   - View cell — the Markdown report renders inline. The same file is saved on disk at `output/<your-input-stem>/report.md`.

To analyze a different file, re-run the input cell only. Every new input gets its own folder; previous reports are never overwritten.

### Option B — Try the pre-built sample notebook

A ready-to-run notebook is published here:

**Colab notebook:** <https://colab.research.google.com/drive/1mTP0RUljvwo64IowACFL6EcsNuHl52cS#scrollTo=fBwgH6raD4eq>

Open the link, click **`File → Save a copy in Drive`** so you have your own editable copy, then run cells top-to-bottom as described above.

---

## Architecture (what the generated notebook does)

```
              +-------------------+
              |   Orchestrator    |   reads your input once
              +---------+---------+
                        |
              Task tool — N calls in ONE message
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
   subagent-1      subagent-2      subagent-3   ...(2-5 total)
   (own context)   (own context)   (own context)
```

- **One orchestrator** reads the input, dispatches subagents, stitches their outputs verbatim into a Markdown report, and saves it.
- **2-5 specialized subagents** run in **parallel** (all dispatched in a single assistant message via the SDK's `Task` tool). Each runs in an isolated context window with its own short, strict system prompt and a locked OUTPUT FORMAT contract.
- **Bounded blast radius:** `allowed_tools=["Task", "Write"]` — the orchestrator cannot shell out, read other files, or call the web.
- **History-preserving outputs:** re-running with the same input produces `output/<stem>_v2/`, `_v3/`, … so you never lose a prior run.

---

## Customizing the problem

Open [`LAB1_COLAB_SYSTEM_PROMPT.md`](./LAB1_COLAB_SYSTEM_PROMPT.md) and edit **Section 6** only (project title, slug, report filename, section headings, problem statement). Leave Sections 1-5 and 7-12 untouched — they encode the architectural invariants the lab is teaching.

You can also override Section 6 at runtime: leave the file as-is and tell Claude Code "I want to build something different" at the first prompt; it will ask for a name + problem statement and design around that for the session only.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `ANTHROPIC_API_KEY is not set` in Colab | Re-run the API-key cell. Colab sometimes restarts the kernel between sessions. |
| `ModuleNotFoundError: claude_agent_sdk` | Re-run the install cell, then re-run everything below it. |
| "Choose Files" button never appears | Re-run the install cell (it brings in `google.colab.files`), then re-run the input cell. |
| Subagents fire sequentially instead of in parallel | Output is still correct, just slower. The orchestrator should emit all N `Task` calls in one message — if it's splitting them, re-run the run cell. |
| `output/<stem>/report.md` missing after a run | Scroll up in the Run cell output — the orchestrator's last message usually says why. |
