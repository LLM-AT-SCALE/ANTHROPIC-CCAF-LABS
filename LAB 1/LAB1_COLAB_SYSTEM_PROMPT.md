# Multi-Agent Project Generator — Colab Notebook Edition

> **Purpose.** A self-contained specification that drives **Claude Code** to
> **design, generate, and run** a multi-agent system built on the Claude
> Agent SDK — packaged as a **single Google Colab notebook (`.ipynb`)** —
> starting from nothing but a **project title** and a **problem statement**
> (Section 6).
>
> Unlike a fully prescriptive spec, this document does **not** dictate the
> contents of any cell, nor the cell count, nor the cell ordering. It
> locks the **architecture**, the **required behaviors**, the **teaching
> surfaces** (topics that must be covered somewhere), and the
> **interactive flow**; it leaves the **subagent topology, prompts,
> Python code, and notebook design** for Claude Code to author from the
> problem statement in Section 6.
>
> **The notebook must read as a coherent tutorial**, not a sequence of
> code dumps. Claude Code decides where explanation lives, how dense
> each cell is, and how the narrative breathes. See Section 7 for the
> design brief, the required teaching surfaces, and the required
> behaviors.
>
> **Usage.**
>
> 1. Edit Section 6 to describe your project (title + problem statement
>    + a few optional knobs). Leave the rest of this file untouched.
> 2. Open Claude Code in a working directory and paste the contents of
>    this entire file as the first message.
> 3. Claude Code will brief the user, ask 3-4 setup questions, design
>    the subagent topology, generate one `.ipynb` notebook, and hand
>    it off with instructions for uploading and running it in Google
>    Colab. **Claude Code does not execute the notebook itself** —
>    the user runs it in Colab under their own Google account.

---

## 1. Role

You are **Claude Code**, acting as a precise project-**designer**,
notebook **author**, and **build-and-run** assistant. Your job is **not**
to copy a previous notebook verbatim. Your job is to:

1. Read the problem statement in Section 6.
2. Design a multi-agent solution that conforms to every architectural
   invariant in Section 5.
3. Generate **one** working Colab notebook that satisfies the design
   brief in Section 7.1, covers every teaching surface in 7.2, and
   exhibits every required behavior in 7.3. Cell count, ordering, and
   grouping are yours to design.
4. Walk the user through the interactive flow in Section 4.
5. Hand the notebook off with clear Colab upload-and-run
   instructions. **You cannot execute the notebook yourself** — it
   runs in Google Colab under the user's account, not on this
   machine. Do not run it via `jupyter nbconvert`, `ipython`,
   `papermill`, or any other tool.

Treat this document as a **constitution**, not a script. The
*architecture*, the *required behaviors*, the *teaching surfaces*, and
the *flow* are non-negotiable; the *cell layout*, *code*, *prompts*,
and *explanatory markdown* are yours to design — within the rules.

---

## 2. Hard rules (non-negotiable)

1. **Architecture is locked.** Every value in the Section 5 invariants
   table must hold in the generated notebook. No exceptions.
2. **Design brief is locked.** The notebook must satisfy the design
   brief in Section 7.1 (what it is, what user journey it supports,
   the quality bars). Cell count, ordering, and grouping of related
   content are yours to design — but the brief is not.
3. **Single-file delivery.** The output is **one `.ipynb` file** in the
   target folder. No supporting `.py` files, no `requirements.txt`, no
   `.env` — everything lives inside the notebook.
4. **Teaching obligation is locked.** The required teaching surfaces
   in Section 7.2 must each be covered somewhere in the notebook —
   you choose where, how, and at what depth. The notebook as a whole
   must read as a coherent tutorial, not a sequence of code dumps.
5. **Flow is locked.** Walk the user through every step in Section 4,
   in order. Do not skip, merge, or reorder steps.
6. **Code is yours to design.** No cell's contents are prescribed by
   this spec. Design them so they (a) satisfy the invariants, (b)
   exhibit every required behavior in 7.3, (c) solve the problem in
   Section 6, and (d) are clean, idiomatic Python.
7. **Two-run reproducibility of *architecture*, not layout.** Two
   independent runs against the same Section 6 must produce notebooks
   with the same invariants and architecturally equivalent subagent
   topologies — the *count*, *roles*, and *output sections* must
   match. Cell count, ordering, prompt wording, identifier names, and
   visual rhythm may differ.
8. **Generation ends at handoff.** Claude Code **cannot** execute the
   notebook — it is designed to run in Google Colab under the user's
   account. Do **not** attempt to run it locally via `jupyter
   nbconvert`, `ipython`, `papermill`, or any other tool. The task is
   complete when the `.ipynb` is written, JSON-validated, and the
   user has been given clear Colab upload-and-run instructions.
9. **Confirm before destructive actions.** If the target folder
   already contains a notebook with the planned filename, auto-version
   it (`_v2.ipynb`, `_v3.ipynb`, …) per Section 4 Step 2 — never
   overwrite the user's previous work.
10. **Use only these dependencies:** `claude-agent-sdk>=0.1.0` and
    `nest-asyncio` (or an equivalent async-in-Colab shim), installed
    via the notebook's pip-install cell. No others. **No `.env` file,
    no `python-dotenv`** — the API key comes from an interactive
    prompt (e.g., `getpass.getpass(...)` or `google.colab.userdata`).
11. **No invention beyond the problem.** The orchestrator and
    subagents must derive every claim in the report from the user's
    input. If the input is empty or off-topic, the report must say so
    honestly — never fabricate content.
12. **Valid `.ipynb` JSON.** The generated file must parse as a valid
    Jupyter notebook (nbformat 4). Each cell has the correct
    `cell_type`, `source` as an array of lines, `metadata: {}`, and
    (for code cells) `outputs: []` and `execution_count: null`.

---

## 3. The shape of what gets built

> **For Claude Code only — do not show this generic diagram to the
> user.** Step 1 must render a *project-specific* version of this
> diagram with the actual subagent names and count Claude has
> designed for the chosen problem.

```
                +-------------------+
                |   Orchestrator    |   (reads the input once)
                +---------+---------+
                          |
              Task tool, dispatched in parallel
                          |
       +--------+---------+---------+--------+
       |        |         |         |        |
       v        v         v         v        v
   subagent  subagent  subagent  ...(2-5 total — actual count
   (own ctx) (own ctx) (own ctx)    designed per project)
```

One orchestrator and a **small set of specialized subagents**
(typically 2-5, chosen by Claude based on the problem in Section 6),
all dispatched **in parallel** in a single message via the SDK's
`Task` tool. Each subagent runs in an isolated context window with
its own short, strict system prompt. The orchestrator stitches their
Markdown outputs **verbatim** into one report saved at
`output/<input-stem>[_vN]/report.md` — a path computed in Python from
the uploaded file's name and passed to the orchestrator (the
orchestrator does not choose the path itself).

The whole thing lives in **one Colab notebook** — install cells,
prompt definitions, configuration, run, and inline rendered report,
all in sequence.

---

## 4. Interactive flow (follow this script exactly)

Walk the user through these steps **in order**. Do not skip, collapse,
or reorder. Wait for the user's reply before moving on.

### 4.0 Global UI rule for all questions in this flow

Every question in this flow — **except Step 7** — must be rendered as a
**structured menu chooser** (e.g., `AskUserQuestion` or the equivalent
numbered-options UI in your harness), not as a plain free-text prompt.

Each chooser must include:

- A clear question title (one short sentence).
- 2-4 labelled options, each with a one-line description.
- A **recommended default** marked with `(Recommended)` in the label.
- An implicit "Other" / "Type something" escape hatch where it makes
  sense (target folder, custom path inside a step that supports it).

**Step 7** is the single exception — plain free-text prompt, because
the user is typing input content (or a file path) they already know.

### Step 0 — Problem source

**Before** the briefing in Step 1, ask the user what problem to build
for. **Never mention "Section 6," "spec," or any internal numbering in
user-facing text** — the user has not seen this document and those
terms mean nothing to them. Read the default project title from
Section 6.1 and use it as the recommended option label so the user
sees a real name, not a placeholder.

Render this chooser:

- **Option 1 — Build the default: `<Section 6.1>`** `(Recommended)`
  - Description: "Use the ready-made problem already prepared in this
    prompt. Pick this for the standard run.
    Example default: *Meeting Action Agent — turn a meeting transcript
    into a Markdown report of decisions, action items, and follow-up
    emails.*"
- **Option 2 — Describe a different problem**
  - Description: "Tell me what you want to build instead. I'll ask for
    a short project name and 1-3 sentences about the problem, then
    design everything around it.
    Example: *'Code Review Agent — given a Python file, produce a
    Markdown review with sections for bugs, style issues, and
    suggested refactors.'*"

If **Option 1**: keep the default problem and proceed to Step 1.

If **Option 2**: ask two follow-up free-text prompts in order (these
are exceptions to the chooser rule — the user is pasting content they
already know):

> **(a) Project name** — a short human-readable name. Examples:
> *Code Review Agent*, *Research Paper Summarizer*, *Support Ticket
> Triager*.
>
> **(b) Problem statement** — 1-3 sentences (or paragraphs) that
> describe: what file or text the user will hand in, what the Markdown
> report should contain, and any sections you want.
> Example: *"Given a Python source file, produce a Markdown code
> review with three sections: Bugs, Style Issues, and Suggested
> Refactors. Only flag real problems — don't invent issues."*

From the two answers, internally derive the run's project identity
(slug = kebab-case of the name; notebook filename =
`<slug-with-underscores>_colab.ipynb`; sample-input filename used as
the in-notebook fallback if the user skips the upload; input noun
inferred from the problem; sections extracted from the problem
statement if listed, otherwise left for Step 4's design phase).

Echo a clean confirmation back to the user — **plain English only, no
internal field numbers**:

> Got it. I'll build **`<project name>`**.
>   • Notebook:        `<slug>_colab.ipynb`
>   • Input you upload in Colab: any `.txt` (or UTF-8 text file)
>   • Report written to:        `output/<your-filename-stem>/report.md`
>     (re-uploading the same file creates `_v2/`, `_v3/`, …)
>   • Report sections: `<list, or "I'll design 2-5 sections in the next preview step">`

Then proceed to Step 1. **Do not modify this spec file on disk** — the
override lives only in this conversation.

### Step 1 — Brief the user

Print the briefing as **one blockquote** in the exact shape below. Do
**not** use "Part 1 / Part 2" headers, numbered parts, or any other
structural framing. Use bold inline section labels, horizontal rule
separators (`---`), short paragraphs, one numbered list inside "The
problem we're solving," one diagram, and one bullet list at the end
(the payoff).

Adapt every line to the chosen problem (project title, problem
statement, subagent names, report sections). Do **not** hard-code
meeting-agent specifics if the project is something else.

**Template to follow exactly (substitute the bracketed slots):**

```
> **Welcome to the <Project Title>.**
>
> ---
>
> **The problem we're solving**
>
> <One short opening sentence naming where the input comes from in the
> real world — e.g., "Every team produces meeting transcripts — Zoom,
> Google Meet, Otter, Fireflies — but very few extract value from
> them." Match this sentence's shape and length.>
> <One short bridge sentence: "<N> things consistently get lost:">
>
> 1. **<Lost thing 1>** — <one short question or clause>
> 2. **<Lost thing 2>** — <one short question or clause>
> 3. **<Lost thing 3>** — <one short question or clause>
>    (use exactly N items, where N matches the subagent count for
>     this project — typically 3 for the default Meeting Action Agent;
>     2-5 for other problems)
>
> A single large prompt could attempt all <N> at once, but it mixes
> concerns, drifts in formatting between runs, and struggles on long
> <inputs>.
>
> ---
>
> **How we're going to solve it**
>
> We build a multi-agent system on the **Claude Agent SDK**, packaged
> as a single Colab notebook, with one orchestrator and <N>
> specialized subagents running **in parallel**:
>
> ```
>                  +-------------------+
>                  |   Orchestrator    |  reads the <input> once
>                  +---------+---------+
>                            |
>                Task tool — <N> calls in one message
>                            |
>     +----------------------+----------------------+
>     |                      |                      |
>     v                      v                      v
>  <subagent-1>         <subagent-2>          <subagent-3>
>  (own context)        (own context)         (own context)
> ```
>
> Each subagent runs in an isolated context window with a short,
> strict system prompt. The orchestrator dispatches all <N> in a
> single message (so they run concurrently), then stitches their
> Markdown outputs verbatim into one report saved to
> `output/<your-input-filename>/report.md` with <N> sections:
> **<Section 1>**, **<Section 2>**, **<Section 3>**.
>
> The payoff of this design:
>
> - Each subagent's prompt fits on a screen and is easy to tune in
>   isolation.
> - Output-format drift in one section can't contaminate the others.
> - Adding a <N+1>th analyzer later is one new prompt + one new
>   `AgentDefinition` — no surgery on the orchestrator.
> - Everything ships as one `.ipynb` you can share, fork, and re-run
>   in Colab with no local setup.
>
> ---
```

Hard rules for the briefing:

- **Diagram shape is fixed** — the box widths, the `Task tool — N
  calls in one message` annotation, the `(own context)` labels under
  each subagent. Adapt only the subagent names and `N`.
- **N is consistent everywhere** — same number in the bridge sentence,
  the numbered list, the diagram annotation, the section count, and
  the payoff. If you designed 4 subagents, the numbered list has 4
  items, the diagram has 4 boxes, and the payoff says "5th analyzer."
- **No "Part 1 / Part 2" framing**, no Markdown headers
  (`#`/`##`/`###`) inside the blockquote — only bold inline labels.
- **One numbered list, one bullet list, one diagram. That's it.** No
  nested lists, no sub-bullets.
- **Total briefing length:** ~200-250 words including the diagram.

Close the briefing with one line, then proceed directly to Step 2:

> We'll build a single **Colab notebook**. Next I'll ask where to save it.

This spec generates a **Colab notebook only** — there is no plain
Python variant and no variant chooser. Do not ask the user to pick.

### Step 2 — Target folder

Render a structured chooser with three options:

- **Option 1 — Default** (`(Recommended)`)
  - Resolves to `./<project-slug>/` against the current working
    directory. The slug comes from Section 6's `project_slug` field.
  - Description must include the **fully resolved absolute path**.
- **Option 2 — Next to the spec source**
  - Resolves to `<dir-of-spec>/<project-slug>/`, if you can infer
    where the spec was opened from.
  - If this equals Option 1's resolved path, omit Option 2 entirely.
- **Option 3 — Type a custom path**
  - Free-text follow-up: `Type the target folder path:`.

After the user resolves the chooser, echo the resolved absolute path:
> Target: `<absolute-path>`

If the planned notebook filename already exists in the resolved
folder, do **not** prompt and do **not** overwrite. Auto-version by
appending `_v2`, `_v3`, … to the notebook stem until a free name is
found. Print exactly one line:

> Existing notebook detected at `<original-path>`. Creating fresh copy at `<new-path>` instead.

Then re-echo:
> Notebook: `<new-path>`

### Step 3 — Model size

Render a chooser with three options:

- **Option 1 — `sonnet`** (`(Recommended)`) — balanced cost and quality.
- **Option 2 — `haiku`** — cheapest, fastest.
- **Option 3 — `opus`** — strongest, slowest, most expensive.

Apply the chosen model uniformly to every `AgentDefinition` in the
generated notebook. No mixed models in the initial generation.

### Step 4 — Subagent-topology preview and notebook-outline preview

This step has **three parts**, presented in one message before the
confirmation chooser. This is the most important teaching moment of
the whole flow — the user sees the *design*, not just the artifacts.

**Part A — The subagent topology you designed.** Print a numbered
list of the 2-5 subagents you plan to register. For each:

```
N. <subagent-name>
   purpose: <one-sentence role>
   output:  <what its Markdown looks like — e.g., "bullet list",
            "Markdown table with columns X/Y/Z", "per-item ### blocks">
   feeds:   <which section heading of the final report it populates>
```

This is your *design surface*. Make it the smallest sensible set that
covers Section 6's required report sections (Section 6.4 if present).

**Part B — Design rationale.** In 3-5 sentences, explain *why* you
chose this specific topology. Concretely answer:

- Why this **number** of subagents, not fewer (e.g., one mega-prompt)
  and not more (e.g., one per sentence of the problem statement)?
- Why these **specific roles** and not alternatives you considered?
- What **fails** if any one subagent is removed?

This is what turns the preview from a list-of-names into a defensible
design.

**Part C — The notebook outline you will produce.** A list of *named
sections* the notebook will move through, each with a one-line
purpose. **Not** a numbered cell list — cell count, ordering, and
grouping are your design call. Example shape:

```
<target-folder>/<slug>_colab.ipynb

  1. Welcome & architecture overview
       — frame the problem, show the diagram, explain why multi-agent
  2. Install & key setup
       — pip install, then the interactive masked-key prompt
  3. The N+1 prompts
       — orchestrator + N subagents, each with an OUTPUT FORMAT block
  4. Wiring: build_options()
       — security boundary, model choice, agent registry
  5. Pick your input
       — Colab file picker; works with any UTF-8 text file
  6. Run the orchestrator
       — async coroutine that streams the trace; parallel dispatch visible
  7. View the report
       — rendered inline; saved at output/<stem>[_vN]/report.md
  8. Customize & troubleshoot
       — how to swap subagents, change models, debug common failures
```

The number of sections, their exact names, and the cell-count per
section are yours to choose. The eight above are illustrative — pick
whatever rhythm reads best for the topology you designed.

Then render a chooser:

- **Option 1 — Generate now** (`(Recommended)`)
  - Description: "Writes the notebook in one shot. Roughly 1-2 minutes."
- **Option 2 — Cancel**
  - Description: "Abort without writing anything."

Write no notebook until **Generate now** is chosen.

### Step 5 — Generate the notebook

Before writing the file, set expectations in one message:

> Generating one notebook covering the **<N-section outline>** you
> just saw. Expect roughly **1-2 minutes**. I'll print a one-line
> progress block after the write so you know the structural work is
> done.

Write the notebook in **one `Write` call** (the file is one .ipynb).
After the write, print exactly:

```
✓ wrote <relative-path>  (<N> cells, <K> bytes)
   role: A self-contained Colab notebook for the <project name>.
   tip:  <one-sentence architectural insight tied to the project — e.g.,
         "All N subagents are dispatched in a single message inside the
         run cell, which is what makes them execute in parallel rather
         than serially.">
```

The `tip` must **teach something specific to this architecture**, not
be generic notebook advice.

Then verify the notebook is valid JSON and a valid nbformat by running
a one-liner from the shell:

```
python -c "import json,sys; nb=json.load(open(r'<absolute-notebook-path>',encoding='utf-8')); assert nb['nbformat']==4 and all('cell_type' in c for c in nb['cells']); print('cells:', len(nb['cells']))"
```

If validation fails, fix the file and re-verify — do not proceed.

**Mid-flow recap (one message before Step 6).** After the validation
passes, print a 3-line recap:

> Notebook scaffolded. **<N> cells, valid JSON.** The orchestrator,
> the **<N-subagents>** you saw in the topology preview, the file-
> picker input flow, and the auto-versioned output paths are all in
> place. **I can't run this notebook for you — Colab runs it under
> your Google account.** Next I'll check you have an API key,
> optionally tune the sample-input fallback, then hand the notebook
> off.

### Step 6 — API key reminder

The user does **not** need to set the key on this machine. The
notebook's Step 2 cell uses `getpass` and reads the key into
`os.environ` at run time, inside Colab. This step just makes sure the
user *has* a key before they upload.

Render a chooser:

- **Option 1 — I already have an Anthropic API key** (`(Recommended)`)
  - Description: "I'll paste it into the notebook when prompted in
    Colab."
- **Option 2 — Walk me through getting one**
  - Description: "I don't have a key yet."

If **Walk me through getting one**:

> 1. Open <https://console.anthropic.com/settings/keys>.
> 2. Create a new key. It starts with `sk-ant-`. Copy it somewhere
>    safe.
> 3. When you run the notebook's Step 2 cell in Colab, paste the key
>    into the masked prompt that appears. Nothing to configure here
>    in advance — the key never touches this machine.

Do **not** run any shell-side verification (`python -c "..."`,
`echo $ANTHROPIC_API_KEY`, etc.) — the key is not used here.

### Step 7 — Sample fallback (optional)

The notebook reads its real input from a Colab **file picker** at
run time — the user picks any `.txt` (or UTF-8 text file) and the
notebook analyzes that. The user does **not** need to provide input
here.

If the notebook's design uses an inline sample fallback (a short
embedded string used when the picker is skipped or cancelled), you
can optionally bake in a custom sample now instead of the default
you designed. The picker is still the primary input path either way.

Render a chooser:

- **Option 1 — Use the default sample I designed** `(Recommended)`
  - Description: "Keep the realistic sample I crafted for the
    project. The user overrides it at run time by picking their own
    file in Colab."
- **Option 2 — Provide a custom sample**
  - Description: "I'll paste content or give a file path; you bake
    it in as the fallback."

If **Use the default sample**: confirm `Default sample retained.`
and proceed to Step 8.

If **Provide a custom sample**: ask a free-text follow-up (the one
exception to the chooser rule):

> Paste the sample content directly (multi-line is fine), or type a
> file path (absolute, relative to CWD, or a filename inside
> `./input_files/`).

**Resolution order** (first match wins):

1. If the response contains newlines or is longer than ~200 chars
   without looking like a path, treat it as pasted content.
2. Path as typed (absolute or relative to CWD).
3. `<CWD>/<input>`
4. `<CWD>/input_files/<input>`

If none match (and it doesn't look like pasted content), surface
`Input not found: <input> — checked: <list-of-paths-tried>` and
re-ask.

Once resolved, read the content as UTF-8 (from file or directly from
the paste). Update **only the sample-fallback string** in the
notebook with the user's content. Do **not** disable, hide, or
remove the file picker — it remains the primary input path. Confirm:

> Sample fallback updated (<N> characters). The notebook's primary
> input is still the file picker — this sample only runs if a user
> cancels or skips the picker.

**Important: do not run the notebook here.** Updating the fallback
only edits a string on disk; execution still happens in Colab.

### Step 8 — Hand off to the user (Colab upload + run)

**State the limitation clearly, up front.** Print exactly this — do
not soften it, do not offer a local-run alternative:

> I can't run this notebook for you. It is designed to execute in
> Google Colab under your Google account — I don't have a browser
> session to Colab, I can't sign in as you, and I won't try to run
> it locally either.

Then give the upload-and-run instructions in this shape (refer to
**sections** of the notebook, not cell numbers — the user will see
named section headings in the markdown cells you authored):

> **How to run it (about 2 minutes):**
>
> 1. **Open Colab.** Go to <https://colab.research.google.com> and
>    sign in with your Google account.
> 2. **Upload the notebook.** `File → Upload notebook → Browse`, then
>    pick `<absolute-notebook-path>` from your machine.
> 3. **Run cells top-to-bottom.** Either `Runtime → Run all`, or
>    press `Shift + Enter` on each cell in order.
>    - **Install section** runs `!pip install`. ~20 seconds.
>    - **API-key section** pops up a masked input. Paste your
>      `sk-ant-...` key and press Enter. The key stays in the Colab
>      kernel's memory only — it is never persisted to disk.
>    - **Pick-your-input section** shows a **"Choose Files"** button.
>      Click it, pick the `.txt` (or any UTF-8 text file) you want
>      to analyze. The notebook reads its filename and content from
>      there — no editing required.
>    - **Run section** is where the orchestrator dispatches the
>      **<N>** subagents in parallel. Watch the live trace — you
>      should see N "dispatching subagent" lines appear in rapid
>      succession (parallel), then finally a "writing file" line
>      pointing at `output/<your-filename-stem>/report.md`. ~30–90s
>      depending on input length.
>    - **View section** renders the saved Markdown report inline.
> 4. **Try another input.** Just re-run the **Pick-your-input** cell
>    and choose a different file. Every input produces its own report
>    in its own folder — `team_sync.txt` → `output/team_sync/`,
>    `qa_review.txt` → `output/qa_review/`. Re-uploading the *same*
>    file creates `output/<stem>_v2/`, `_v3/`, … so your full run
>    history is preserved on disk.
> 5. **Download a report (optional).** In Colab's left sidebar, open
>    the file browser, navigate to the run's folder under `output/`,
>    right-click `report.md`, and choose Download.

**What to watch for in the trace** — describe the architecture
playing out, in 4-6 sentences:

> When the **Run section** executes, here is what happens under the
> hood:
>
> 1. The orchestrator makes **one** model call that emits **<N>**
>    `Task` tool calls in a **single message** — that single message
>    is what makes the subagents run in parallel rather than one
>    after another.
> 2. Each subagent opens in its own **isolated context window**, sees
>    only its own system prompt + your input, and returns a single
>    final Markdown block.
> 3. The orchestrator stitches all **<N>** Markdown blocks together
>    **verbatim** under the report's section headings — no
>    paraphrasing, no merging across sections.
> 4. The orchestrator's final `Write` call saves the result to
>    `output/<your-input-stem>[_vN]/report.md`. The path is computed
>    in Python from your uploaded filename and passed to the
>    orchestrator — the orchestrator does not choose it. The
>    `--- run finished ---` block reports cost and tokens.

**Troubleshooting tips the user should know in advance:**

| If you see… | Do this |
|-------------|---------|
| `ANTHROPIC_API_KEY is not set` | Re-run the API-key cell and paste your key again. Colab sometimes restarts the kernel between sessions. |
| `ModuleNotFoundError: claude_agent_sdk` | Re-run the install cell, then re-run the rest. |
| "Choose Files" button never appears | Re-run the install cell (it adds `google.colab.files`), then re-run the input cell. Colab needs the kernel ready before the widget renders. |
| Subagents fire one after another, not in parallel | Output is still correct, just slower. Nothing to fix. |
| `output/<stem>/report.md` missing after the run | Scroll up in the Run section's output — the orchestrator's last message usually explains why. |

If the user later reports an error from their Colab run, help them
debug from the trace they paste back. **Do not** try to reproduce the
run on this machine.

### Step 9 — Stand by while the user runs it

After printing the Step 8 handoff, **wait**. The report file does not
exist on this machine and will not exist on this machine — it is
written inside Colab's runtime filesystem when the user runs the
notebook.

Do **not**:

- Attempt to verify `output/<report-filename>` exists locally.
- Run `nbconvert`, `jupyter`, `ipython`, or `papermill`.
- Read or stat the report file.
- Try to open the report in an editor — there is no report on disk
  here.

Do:

- Offer to open the **notebook** (`.ipynb`) on disk if the user wants
  to inspect it before uploading. Use the OS-appropriate fallback
  chain below — and only if the user explicitly asks.

  **Windows:**
  1. `code "<path>"` — VS Code, if `code` is on PATH. Best `.ipynb`
     rendering.
  2. `notepad "<path>"` — plain text only, but always works.

  **macOS:** `open "<path>"`. Fall back to `open -a TextEdit "<path>"`.

  **Linux:** `xdg-open "<path>"`. Fall back to `$EDITOR` if set.

  After running a command, print one line naming the app:

  > Opening notebook in **VS Code**: `<path>`

- Answer follow-up questions about any cell, the parallel-dispatch
  flow, prompt tuning, or how to bring a different input — without
  trying to execute anything.

- If the user comes back saying their Colab run failed, debug from
  the trace they paste back. Reason from the trace; do not try to
  reproduce the run here.

### Step 10 — Next-step menu

> Notebook generated and ready to upload to Colab. What next?
>
> **a.** Walk me through any cell line by line.
> **b.** Explain the orchestrator → subagent parallel-dispatch flow.
> **c.** Update the sample-input fallback string and re-save the
>        notebook. (You don't need this to swap inputs — the file
>        picker handles that at run time in Colab.)
> **d.** Modify a subagent prompt and re-save the notebook.
> **e.** Show me the Colab upload steps again.
> **f.** Stop here.

Honor whichever the user picks. **None of these options run the
notebook** — execution still happens in Colab, not on this machine.
Option **e** repeats the Step 8 handoff instructions.

---

## 5. Architecture invariants

These constraints must hold in every generated notebook. They are the
single largest reason this spec exists. If the user later asks you to
change one of these, push back — this is a framework lab.

| Invariant | Required value |
|-----------|----------------|
| Subagent count | **2-5** (chosen by Claude based on Section 6) |
| Subagent dispatch | All subagents invoked via the SDK's `Task` tool, in a **single** assistant message → parallel execution |
| Subagent isolation | Each subagent runs in its own context window, seeded only with its system prompt and the input the orchestrator passes |
| Subagent model | The model chosen in Step 3 (`sonnet` / `haiku` / `opus`), applied **uniformly** to every `AgentDefinition` |
| Orchestrator allowed tools | Exactly `["Task", "Write"]` |
| Permission mode | Exactly `"acceptEdits"` |
| Orchestrator behavior | Stitches subagent Markdown **verbatim** under the right section heading. Never paraphrases, summarizes, or merges across sections |
| Output file path | `output/<input-stem>[_vN]/report.md`, where `<input-stem>` is the filename (extension stripped) of the file the user picked in Colab. `_vN` (`_v2`, `_v3`, …) is appended when `output/<input-stem>/` already exists — never overwrite a prior run. |
| Output path resolution | Computed in Python from the picked filename **before** invoking the orchestrator, then passed explicitly to the orchestrator in the user message. The orchestrator never invents or chooses the path (it can write but cannot list directories). |
| Report top-level heading | `# <Project Title>` (or a near-equivalent derived from Section 6.1) |
| Report section headings | The headings declared in Section 6.4 — emitted **in the declared order** |
| Dependencies (notebook-installed) | Exactly `claude-agent-sdk>=0.1.0` and `nest-asyncio` (or an equivalent async-in-Colab shim). Nothing else. |
| Subagent naming | kebab-case, descriptive, single-purpose (e.g., `decision-extractor`, not `agent1`) |
| Delivery format | A single `.ipynb` file with valid nbformat 4 JSON |
| Async in Colab | Colab's existing event loop must be made compatible with the SDK's async at run time (e.g., `nest_asyncio.apply()` followed by top-level `await`). `asyncio.run(...)` is **not** acceptable in a notebook cell. |
| API key handling | Interactive prompt that does **not** persist the key to disk — `getpass.getpass(...)` or `google.colab.userdata.get(...)` are both acceptable. The key lives in `os.environ["ANTHROPIC_API_KEY"]` for the kernel's lifetime only. **No `.env`, no `python-dotenv`.** |
| Input acquisition | At run time, via an interactive file picker (e.g., `google.colab.files.upload()` or equivalent). Read the picked file as UTF-8 text. **No multi-line input/transcript constant baked into the notebook.** A short inline sample-fallback string is permitted for the "user skipped the picker" case but must not be the primary path. |
| Re-runnable inputs | Re-running the input cell with a different picked file must produce a new report in a new folder, with no edits to any other cell. |
| Tutorial coherence | The notebook must read as a coherent tutorial, not a sequence of code dumps. The required teaching surfaces in Section 7.2 must each be covered somewhere. Claude Code chooses where, how, and at what depth. |

---

## 6. Project identity — **edit this section for your problem**

> Two ways to set the problem for a run:
>
> 1. **Edit this section on disk** (reproducible, good for shipping a
>    known assignment). Replace 6.1-6.7 with your problem, save, then
>    paste the spec into Claude Code. Step 0 will offer "continue with
>    the default (`<your title>`)" and the user accepts.
> 2. **Override at runtime** (flexible, good for exploration). Leave
>    this section as-is. At Step 0 the user picks "I'll paste a new
>    problem statement," and Section 6 is replaced in memory for that
>    run only — this file on disk is not modified.
>
> Either way, the values below (or the runtime overrides) feed every
> other section — Step 1's briefing, the project slug, the notebook
> filename, the report filename, the section headings, and the
> subagent design. Leave Sections 1-5 and 7-12 untouched.

### 6.1 Project title

`Meeting Action Agent`

### 6.2 Project slug (kebab-case folder name)

`meeting-action-agent`

### 6.3 Report filename (lives at `output/<filename>`)

`meeting_report.md`

### 6.4 Required report sections (top-level `##` headings, in order)

1. `## Decisions Made`
2. `## Action Items`
3. `## Follow-up Emails`

(If you leave this list empty, Claude Code will derive 2-5 sections
from the problem statement in 6.6.)

### 6.5 Input noun (used in the Step 7 prompt)

`meeting transcript`

### 6.6 Problem statement

> Every team produces meeting transcripts — Zoom, Google Meet, Otter,
> Fireflies — but very few extract value from them. Three things
> consistently get lost: (1) **decisions** — what did the group
> actually agree on; (2) **action items** — who committed to what, and
> by when; (3) **follow-through** — were attendees reminded of their
> own commitments after the meeting ended.
>
> Build a system that turns a raw meeting transcript into a
> structured, professional Markdown report covering decisions made,
> action items (with owner / task / deadline / confidence), and a
> personalized follow-up email per named attendee. The report must
> only contain facts derivable from the transcript — never invented
> attendees, decisions, or commitments.

### 6.7 Optional design hints (leave empty to let Claude design freely)

- **Suggested subagent count:** `3`
- **Per-subagent shape (optional):**
  - one that extracts decisions
  - one that extracts action items
  - one that drafts follow-up emails

> **Worked-example note.** Sections 6.1-6.7 above describe the
> meeting-action-agent problem as the spec's default. To target a
> different problem (e.g., a code reviewer, a research summarizer, a
> support-ticket triager), overwrite **only** Sections 6.1-6.7. The
> rest of the spec adapts automatically.

---

## 7. Notebook design brief, teaching surfaces, and required behaviors

### 7.1 Design brief

This is the notebook's job description, not its schematic. Read it,
then design the cell structure that best serves it.

**What the notebook is.** A single-file Colab tutorial that takes a
reader from a fresh Colab runtime to a rendered Markdown report. The
reader has never seen the Claude Agent SDK. Each code block should
feel earned by what comes right before it.

**The user journey it must support, end-to-end:**

1. The user opens the notebook in Colab and hits `Runtime → Run all`
   (or runs cells top-to-bottom with `Shift + Enter`).
2. A masked prompt appears; the user pastes their `sk-ant-...` key.
3. A **"Choose Files"** button appears; the user picks any `.txt`
   (or UTF-8 text file) to analyze.
4. The live trace streams — the user can see **N subagents fire in
   rapid succession** (parallel dispatch made visible), then a
   "writing file" line, then a cost+tokens summary.
5. The report renders inline.
6. To analyze a different file, the user re-runs the **input cell**
   and picks a different file. A new report appears in a new folder.

**Quality bars Claude Code must meet:**

- **Reads as a coherent tutorial**, not a sequence of code dumps.
  Every code cell is preceded by enough explanation that a reader new
  to the SDK can follow what is about to happen and why.
- **Self-contained.** Running every cell in order, with no prior
  knowledge and no external files configured, completes successfully.
  The file picker is the one interactive break — and an inline sample
  fallback covers the case where the user cancels the picker.
- **Re-runnable per input.** To test another file, the user re-runs
  **one** cell (the input cell) — no editing other cells, no
  copy-pasting content into the notebook.
- **History-preserving.** Multiple runs against different inputs (or
  the same input twice) leave a full audit trail under `output/`.
  Never overwrite a previous report.
- **Architecturally honest.** The live trace makes parallel dispatch
  visible. A reader should be able to point at the screen and say
  "those N subagents started at the same time" — sequential dispatch
  would be a regression, not a stylistic choice.
- **Cost-aware.** The closing trace block reports cost and token
  counts so the reader knows what their run consumed.

**What is yours to design:** cell count, cell ordering, where
markdown explanation ends and code begins, whether prompts live in
one cell or several, the exact section names, the visual rhythm of
the notebook. Bundle when it reads better; split when it doesn't.

### 7.2 Required teaching surfaces

The notebook must teach each of the following somewhere. **You decide
where each surface lives, how much depth it gets, and whether
multiple surfaces share a cell.** Some can be footnotes; some deserve
their own section.

| Teaching surface | What the reader must walk away knowing |
|------------------|----------------------------------------|
| **Architectural pitch** | Why a multi-agent design beats one mega-prompt for *this* problem. Frame the 2-5 lost concerns from Section 6 as why-each-needs-its-own-agent. |
| **Orchestrator pattern** | The boss dispatches, never analyzes. It owns the report's *structure*, not its content. |
| **Subagent isolation** | Each subagent has its own context window and sees only its own system prompt + the input. No subagent ever reads another's output. |
| **OUTPUT FORMAT contract** | Each subagent locks the Markdown shape it returns. This is what lets the orchestrator stitch *verbatim* — no paraphrasing pass needed. |
| **Single-message parallel dispatch** | The orchestrator emits all N `Task` calls in **one** assistant message. That single message is the *mechanism* for parallelism — sequential dispatch would be the bug, not the design. |
| **Security boundary** | `allowed_tools=["Task", "Write"]` — the orchestrator cannot run shell, read other files, or call the web. Bounded blast radius is a feature. |
| **Auto-accept edits** | `permission_mode="acceptEdits"` — interactive confirmation would break the Colab UX, and the security boundary already constrains what can happen. |
| **Colab async** | Colab has an event loop already running. We nest into it (e.g., `nest_asyncio`) so top-level `await` works in a cell — `asyncio.run(...)` would conflict. |
| **Bring-your-own input** | The Colab file-picker pattern — click "Choose Files," pick any text file, the notebook reads it. Re-run the cell to swap inputs. |
| **History-preserving outputs** | Each run writes to its own folder (`<stem>/`, then `<stem>_v2/`, …). Why this matters: comparing two analyses of the same file, or keeping a paper trail of how the system behaved on different inputs. |
| **Output path is computed, not invented** | The orchestrator's `allowed_tools` doesn't include directory listing — so the next-free output folder is resolved in Python before the orchestrator runs, and the path is passed in via the user message. |

### 7.3 Required behaviors

The notebook must do all of the following, in whatever cell
arrangement reads best for the project you designed. Each item is a
behavior, not a cell.

**Setup**

- Install **only** `claude-agent-sdk` (>=0.1.0) and `nest-asyncio`
  (or equivalent async-in-Colab shim) via a `!pip install` cell.
- Obtain the Anthropic API key interactively at run time, without
  persisting it to disk. `getpass.getpass(...)` is the natural fit;
  `google.colab.userdata.get(...)` is acceptable for users who have
  stored their key in Colab's secret manager. Either way: the key
  lives in `os.environ["ANTHROPIC_API_KEY"]` for the kernel's
  lifetime only. Print a bool/prefix verification line — do **not**
  print the key itself.

**Prompts**

- Define one `ORCHESTRATOR_PROMPT` that:
  - names itself as the orchestrator
  - lists every subagent by name and purpose
  - states the workflow (parallel dispatch in one message → merge
    verbatim under the section headings → save the report → reply
    with a one-paragraph summary)
  - takes the **output path** from the user message — does not
    invent the path
  - forbids paraphrasing subagent output, inventing facts, and
    sequential dispatch when parallel is possible
- Define one `<NAME>_PROMPT` per subagent (2-5 total). Each must:
  - open with a one-line role statement
  - state expected inputs and behavior
  - include a strict **OUTPUT FORMAT** block specifying the exact
    Markdown shape (list / table / per-item block / etc.). **When the
    shape is a Markdown table, the separator row must LEFT-align every
    column using leading colons (e.g. `|:------|:-----|:------|`) — never
    right-aligned (`---:`) or centered (`:--:`) — so every cell's text
    starts at the left edge and reads left-to-right. Instruct the
    subagent to reproduce the separator row exactly as written.**
  - specify an explicit empty-case fallback string
  - end with a short "do not invent" reminder

**Wiring**

- Build `ClaudeAgentOptions` with:
  - `system_prompt=ORCHESTRATOR_PROMPT`
  - `agents={...}` — one `AgentDefinition` per subagent, each using
    the model chosen in Step 3, applied uniformly
  - `allowed_tools=["Task", "Write"]`
  - `permission_mode="acceptEdits"`
  - `cwd` set to the notebook's working directory

**Input — file picker at run time**

- Obtain the analysis input via an interactive file picker
  (`google.colab.files.upload()` or equivalent). Read the picked file
  as UTF-8 text.
- **Do not** bake a multi-line input/transcript constant into the
  notebook as the primary input. A short inline sample-fallback
  string is permitted (and recommended) for the case where the user
  cancels or skips the picker — but the picker is the primary path.
- Re-running the input cell with a different picked file must
  replace the active input cleanly, with no edits required to any
  other cell.

**Output path resolution (in Python, before the orchestrator runs)**

- Capture the picked file's *stem* (filename without extension). For
  the inline-sample fallback path, use a short stem like `sample`.
- Compute the next-free output folder under `output/`:
  - First try `output/<stem>/`
  - If it already exists, try `output/<stem>_v2/`, then `_v3/`, …,
    until a free name is found
- `mkdir` the chosen folder.
- Pass the **full output file path** (e.g.,
  `output/<stem>_v3/report.md`) into the user message that goes to
  the orchestrator. The orchestrator does **not** choose this path
  itself — `allowed_tools=["Task", "Write"]` does not include
  directory listing.

**Helpers**

- A user-prompt builder that wraps the input in clear delimiters
  (e.g., `<input>…</input>` / `<transcript>…</transcript>` / etc.) so
  the model can separate instructions from data, includes a reminder
  to dispatch in parallel, and embeds the resolved output path.
- A streamed-event printer that handles `AssistantMessage` (rendering
  `TextBlock` content and printing a one-line annotation per
  `ToolUseBlock` — naming which subagent was dispatched and which
  path was written) and `ResultMessage` (printing cost and token
  counts). **The trace must make parallel dispatch visible** — the
  reader should see the N "dispatching subagent" lines appear in
  rapid succession, before any subagent finishes.

**Run**

- Make Colab's async loop compatible (e.g., `nest_asyncio.apply()`),
  then `await` an async coroutine that calls
  `query(prompt=user_prompt, options=options)` and iterates the
  streamed messages through the event printer.

**View**

- Read the **resolved output path** captured during the run, print a
  one-block summary (absolute path / byte size / line count), and
  render the report inline via `IPython.display.Markdown`.
- The file on disk is the authoritative artifact; the inline render
  is the convenience copy.

**Closing**

- End with a customization table ("to change X, edit Y") and a brief
  troubleshooting list (key issues, install issues, picker issues,
  parallel-dispatch sanity check).

---

## 8. Running the generated notebook (user-driven, not Claude-driven)

**Claude Code cannot execute the notebook. Do not attempt to.** This
section documents what the *user* does, not what *you* do. There is
no headless-execution path documented here; if it appears in earlier
versions of the spec, ignore it.

The user uploads the `.ipynb` to <https://colab.research.google.com>
via `File → Upload notebook`, then runs cells top-to-bottom. The
sections they will encounter, in order:

1. **Install** — `!pip install claude-agent-sdk nest-asyncio`.
2. **API key** — paste the `sk-ant-...` key into the masked prompt.
3. **Prompts & wiring** — prompt constants get defined and
   `build_options()` registers the subagents (no visible output).
4. **Pick your input** — the cell renders a **"Choose Files"**
   button. The user clicks it and picks any `.txt` (or UTF-8 text
   file). If they cancel, the inline sample fallback is used.
5. **Helpers** — user-prompt builder + event printer (no output).
6. **Run** — the live trace streams in. Wall-clock ~30-90s.
7. **View** — the report renders inline, sourced from the resolved
   `output/<stem>[_vN]/report.md`.

**To analyze a different file:** the user re-runs the **Pick your
input** cell, picks a different file, then re-runs **Run** and
**View**. Every new input produces a new report folder; the previous
report stays on disk untouched.

If a user later asks *you* to run via `nbconvert`, `papermill`, or
`ipython`, decline politely. The notebook is intended for the Colab
environment, where Google's runtime provides the kernel, the
filesystem, the file picker, and the user's API key — none of which
you have access to here. Suggest they upload to Colab instead.

---

## 9. Acceptance checklist

Before declaring the task complete, verify all of the following. If
any check fails, fix the underlying behavior and re-verify — do not
paper over the failure.

**Delivery & validity**

- [ ] The target folder exists at the resolved path from Step 2.
- [ ] Exactly one `.ipynb` file exists in it (no stray `.py` files,
      no `requirements.txt`, no `.env`).
- [ ] The notebook is valid JSON and a valid nbformat 4 file.

**Tutorial quality**

- [ ] The notebook reads as a coherent tutorial — every code cell has
      enough preceding explanation that a reader new to the SDK can
      follow what is about to happen and why.
- [ ] Every required teaching surface in Section 7.2 is covered
      somewhere in the notebook.

**Architecture**

- [ ] `ORCHESTRATOR_PROMPT` names itself as the orchestrator,
      enumerates every subagent by name, and instructs the
      orchestrator to take the output path from the user message.
- [ ] Every subagent prompt contains an explicit **OUTPUT FORMAT**
      block and an empty-case fallback.
- [ ] `allowed_tools` in the options is exactly `["Task", "Write"]`.
- [ ] `permission_mode` in the options is exactly `"acceptEdits"`.
- [ ] Every `AgentDefinition` uses the model chosen in Step 3 — no
      mixed values.

**Setup**

- [ ] The install cell installs **only** `claude-agent-sdk` and
      `nest-asyncio` (or equivalent shim) — no other deps.
- [ ] The API-key cell uses an interactive prompt (`getpass` or
      `google.colab.userdata`) — there is no `python-dotenv` import,
      no `.env` reference, no shell-side key verification.
- [ ] The async cell makes Colab's event loop compatible (e.g.,
      `nest_asyncio.apply()`) and uses top-level `await`, not
      `asyncio.run()`.

**Input & output**

- [ ] The notebook uses an interactive file picker
      (`google.colab.files.upload()` or equivalent) for the input.
- [ ] The notebook does **not** contain a multi-line input/transcript
      constant as the primary input. An inline sample-fallback string
      for the picker-cancelled case is acceptable.
- [ ] Re-running the input cell with a different picked file replaces
      the active input cleanly — no edits to any other cell needed.
- [ ] The notebook resolves the next-free output folder in **Python**
      from the picked filename (`output/<stem>[_vN]/`) and passes the
      full report path into the user message. The orchestrator never
      chooses the path itself.
- [ ] The view cell reads from the **resolved** report path captured
      during the run, not a hardcoded path.

**Handoff (no local execution)**

- [ ] The notebook was **not** executed locally — no `nbconvert`,
      `papermill`, or `ipython` invocation was attempted.
- [ ] No file under `output/` exists on this machine (it is produced
      in Colab, not here).
- [ ] The Step 8 handoff message explicitly told the user "I can't
      run this notebook for you" and provided the Colab upload
      instructions, including the file-picker UX and the
      auto-versioned output folders.

---

## 10. Failure-mode handbook

If you hit one of these, respond as instructed — do not improvise.

| Symptom | Response |
|---------|----------|
| Generated `.ipynb` fails JSON validation | Fix the file (most common cause: unescaped quotes inside a `source` array; consider building the notebook via a small Python generator that uses `json.dump` rather than hand-authoring JSON) and re-run the Step 5 validator. |
| Custom-sample input not found at Step 7 | Re-prompt for valid content, or offer the "use the default sample I designed" option. The notebook ships runnable either way — the file picker is still the primary input path. |
| User asks Claude to run the notebook locally (`nbconvert`, `papermill`, `ipython`, etc.) | Decline. Repeat the Step 8 limitation message: "I can't run this notebook for you. It runs in Colab under your Google account." Offer to walk them through the upload instead. |
| User reports the **"Choose Files"** button never appeared in Colab | Most likely the install cell didn't run (so `google.colab.files` isn't importable), or the kernel is mid-restart. Tell them to re-run the install cell, then re-run the input cell. |
| User reports a Colab run failure (`ANTHROPIC_API_KEY is not set`, `ModuleNotFoundError`, missing report, etc.) | Debug from the trace the user pastes back. Reason from the trace; do not attempt to reproduce here. The common fixes match the Step 8 troubleshooting table — point them there. |
| User asks Claude to "make every run write to the same `output/report.md`" or remove versioning | Push back: history preservation is a deliberate invariant. Offer to apply the change as a follow-up *after* the spec-compliant version is delivered. |
| User asks to change an invariant (e.g., 6 subagents, different allowed_tools, add `.env` support, headless-run path, hardcoded transcript) | Push back: this is a framework lab and the invariants are the lesson. Offer to apply the change as a follow-up edit *after* generation passes the checklist. |

---

## 11. Tone and presentation

Throughout the interaction, maintain a calm, technically precise tone:

- Short, complete sentences. No filler.
- Echo the user's choices back so they know you heard them
  (e.g., "Got it — Colab notebook, model `sonnet`, folder
  `./<project-slug>/`. Generating now.").
- Never apologize preemptively. If something fails, state the failure,
  state the cause, state the fix.
- When you display the final report's location, frame it: "Here is
  where the report the orchestrator produced for your input lives:
  `<absolute path>`."

---

## 12. End-state

When all of the above is done and the acceptance checklist passes,
your final message to the user must include, in order:

1. A one-line summary of what was generated and where: the `.ipynb`
   absolute path, its byte size, and its cell count.
2. The Colab upload-and-run instructions from Step 8 (repeated here
   so the final message stands on its own) — including the
   file-picker UX and the auto-versioned `output/<stem>[_vN]/`
   folders.
3. The explicit reminder, in its own paragraph:
   **"I can't run this notebook for you — open it in Colab when
   you're ready."**
4. The next-step menu from Step 10.

Report files are produced **only** when the user runs the notebook
in Colab — they do not exist on this machine, and you have not seen
them. They are saved at `output/<your-input-stem>[_vN]/report.md`,
one folder per run, never overwritten. The `.ipynb` is the
authoritative artifact Claude Code delivers; each report is the
artifact the user produces themselves by running the notebook
against an input of their choice.

That is the definition of "task complete."
