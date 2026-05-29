# LAB 2 — Bookstore MCP Server

A guided system prompt that turns **Claude Code** into a build-and-walk assistant for a fully functional **Model Context Protocol (MCP)** server. Paste it once and you walk away with a Python project that exposes a local SQLite bookstore to **Claude Desktop**, **Claude Code**, or the **MCP Inspector** — set up, dependency-installed, seeded, smoke-tested, and (optionally) auto-registered with Claude Desktop.

The lab is designed as a teaching reference: every primitive (tool / resource / prompt) is implemented with the `FastMCP` decorator API, every design decision is explained inline in the generated `HOW_WE_BUILT_IT.md`, and the entire flow runs offline with **no API key required**.

---

## What's in this folder

| File | Purpose |
|------|---------|
| [`BOOKSTORE_MCP_SYSTEM_PROMPT.md`](./BOOKSTORE_MCP_SYSTEM_PROMPT.md) | The system prompt you paste into Claude Code. Locks the architecture, schema, and primitive surface; leaves implementation to Claude. |
| `README.md` | This file. |

---

## What you'll build

A self-contained Python project that demonstrates the three core MCP primitives against a local SQLite database:

- **4 Tools** — `run_query`, `top_selling_books`, `revenue_by_author`, `low_stock_books`
- **2 Resources** — `bookstore://tables`, `bookstore://schema/{table}`
- **1 Prompt** — `data_analyst` (a reusable template that primes the model to read the schema, run targeted SQL, and reply in plain English)

### Architecture

```
         Claude Desktop / Claude Code / MCP Inspector
                            |
                stdio (JSON-RPC, default MCP transport)
                            |
                  +---------v---------+
                  |   bookstore MCP   |
                  |       server      |
                  |    (server.py)    |
                  +---------+---------+
                            |
              +-------------+-------------+----------------+
              |             |             |                |
              v             v             v                v
           4 tools     2 resources     1 prompt     SQLite (read-only)
                                                     bookstore.db
```

The server runs as a Python process started by the client. It speaks JSON-RPC over the client's stdin/stdout and opens `bookstore.db` in strict read-only mode (`?mode=ro`) — so even a malicious SQL string cannot mutate the data.

---

## Prerequisites

1. **Claude Code** installed locally — <https://claude.com/claude-code>
2. **Python 3.10 or newer** — <https://www.python.org/downloads/>
3. Optional: **Claude Desktop** if you want to wire the server into the Desktop client at the end.

No API key. No paid services. No internet dependency at runtime (only the one-time `pip install` needs network access).

---

## How to use it

1. **Open Claude Code** in any working directory.
2. **Copy the entire contents** of [`BOOKSTORE_MCP_SYSTEM_PROMPT.md`](./BOOKSTORE_MCP_SYSTEM_PROMPT.md) and paste it as your first message.
3. Claude Code will walk you through an interactive **8-step flow** (~4–6 minutes):

   | Step | What happens |
   |------|--------------|
   | 1 | Welcome briefing — what MCP is, what you're about to build, the architecture diagram |
   | 2 | Pick a target folder (auto-versioned to `-v2`, `-v3`, … if the folder already exists, so prior runs are never overwritten) |
   | 3 | Review the file-tree preview and confirm generation |
   | 4 | Claude writes 10 files in order, surfacing per-file progress |
   | 5 | Creates a project-local `.venv` and installs `mcp[cli]` into it |
   | 6 | Seeds the SQLite database (5 authors, 11 books, 22 sales) |
   | 7 | Runs an 8-behavior smoke test |
   | 8 | Optionally auto-registers the server with Claude Desktop (merges into your existing config — preserves any other MCP servers) |

4. When the flow ends, you'll have three ways to use the server:
   - **MCP Inspector** — `mcp dev server.py` opens a local web UI with tabs for Tools, Resources, and Prompts
   - **Claude Code** — the `.mcp.json` in the folder registers the server automatically the next time you open that folder in a Claude Code session
   - **Claude Desktop** — registered via Step 8 (or paste the printed snippet manually); fully restart Desktop from the system tray to load it

---

## The 10 generated files

```
<target-folder>/
├── requirements.txt           # one runtime dep: mcp[cli]>=1.0.0
├── .gitignore                 # excludes .venv/, __pycache__/, bookstore.db
├── .mcp.json                  # project-scoped MCP registration (Claude Code reads this)
├── seed_db.py                 # builds bookstore.db with sample data
├── server.py                  # the MCP server (FastMCP decorators)
├── samples/
│   ├── _smoke_test.py         # direct-call verification of all primitives
│   └── example_queries.md     # 8–12 questions to try in Claude
├── README.md                  # setup + usage for the generated project
├── HOW_WE_BUILT_IT.md         # architecture deep-dive + design rationale
└── CLAUDE.md                  # session instructions for future Claude Code work in this folder
```

`bookstore.db` and `.venv/` are created by Steps 5–6 and are git-ignored.

---

## Key design decisions (taught by the generated project)

- **Storage-layer read-only enforcement.** The server opens SQLite with `sqlite3.connect("file:...?mode=ro", uri=True)`. No SQL-keyword string filtering — that approach is fragile and gives false confidence. SQLite itself rejects writes at the storage layer.
- **Project-local venv with absolute paths.** `.mcp.json` and the Claude Desktop registration both point at `<target-folder>/.venv/<python>`, an interpreter that's **guaranteed to exist on disk** before any client tries to spawn it. This avoids the `spawn … ENOENT` failures you get when configs reference a system Python that isn't where you expect.
- **Auto-versioned folders + server names.** A second build creates `bookstore-mcp-server-v2/` with server name `bookstore-v2`, so multiple versions can register simultaneously in the same Claude client without collision.
- **Windows MSIX awareness.** On Windows, Claude Desktop ships in two flavors (Microsoft Store / traditional installer) with **two different config locations**. The Store version sandboxes `%APPDATA%`, so writing to the traditional path is a silent no-op. The flow detects the install flavor and writes to the correct path.
- **Errors as dicts, not exceptions.** Every tool returns `{"error": "..."}` on failure rather than raising across the MCP boundary, so the client always gets structured output.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `Could not find Python 3.10+ on PATH` | Install Python 3.10+ from <https://www.python.org/downloads/> and re-run. |
| `pip install` fails with a network error | The one-time install needs internet to fetch `mcp[cli]`. Connect and re-run. |
| On Debian/Ubuntu, `python -m venv .venv` fails | Install `python3-venv` (`sudo apt install python3-venv`). |
| Registered with Claude Desktop, restarted, but the server doesn't appear | On Windows, you likely wrote to the wrong config path. If `C:\Users\<you>\AppData\Local\Packages\Claude_*\` exists, the Store-installed app reads from `<that-dir>\LocalCache\Roaming\Claude\` — **not** `%APPDATA%\Claude\`. Also confirm you did a full system-tray Quit, not just a window close. |
| Smoke test fails on the `INSERT`/`DROP` rejection check | The connection wasn't opened in read-only mode. Confirm `?mode=ro` appears literally in `server.py`. |
| `mcp dev server.py` says module not found | You're not in the project venv. Activate it (`.venv\Scripts\activate` on Windows, `source .venv/bin/activate` on macOS/Linux). |

---

## Re-running the lab

Re-pasting the prompt is safe. Step 2 detects an existing folder and creates a fresh `-v2`, `-v3`, … copy beside it — your previous work is never touched. You can diff a fresh copy against one you've edited to see what drifted.
