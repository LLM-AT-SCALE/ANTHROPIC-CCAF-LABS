# Support Agent v2 — Stress-Tested AI Customer Support

An AI-powered customer support agent built with the **Anthropic Claude API**, featuring fault injection and reliability stress testing. The agent handles real customer workflows (order lookups, refunds, shipment tracking) while being tested against simulated API errors, tool failures, and context overflow conditions.

---

## Features

- **Conversational support agent** — handles multi-turn customer conversations using Claude
- **Three integrated tools** — order lookup, refund processing, and shipment tracking
- **Fault injection engine** — simulates API errors, tool failures, and context overflow
- **Automatic recovery** — retry logic, context summarization, and graceful fallback responses
- **Stress test suite** — 5 scenarios × 9 fault configs with JSON reporting
- **Interactive mode** — chat with the agent directly in the terminal

---

## Project Structure

```
support_agent_v2/
├── agent.py              # Core SupportAgent class with retry and recovery logic
├── tools.py              # Tool definitions and mock order/tracking data
├── fault_injector.py     # FaultConfig — injects API errors and tool faults
├── context_manager.py    # Tracks conversation history, detects overflow, summarizes
├── logger.py             # StressLogger — structured event logging with color output
├── exceptions.py         # Custom exceptions: ContextOverflowError, ToolTimeoutError, etc.
└── run_stress_tests.py   # CLI test runner with scenario and fault filters
```

---

## Setup

### 1. Install dependencies

```bash
pip install anthropic httpx
```

### 2. Set your API key

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Or enter it when prompted in the Colab notebook.

---

## Running the Tests

### Full suite (45 tests: 5 scenarios × 9 fault configs)

```bash
python run_stress_tests.py
```

### Recommended for stress-test documentation (9 tests)

```bash
python run_stress_tests.py --scenario order_lookup --verbose --report my_report.json
```

### Single scenario + single fault

```bash
python run_stress_tests.py --scenario order_lookup --fault api_500
```

### Interactive mode (chat with the agent)

```bash
python run_stress_tests.py --interactive --fault-profile chaos
```

---

## Scenarios

| Key | Description |
|-----|-------------|
| `order_lookup` | Customer asks for the status of a single order |
| `refund_request` | Customer wants a full refund for a damaged item |
| `shipping_track` | Customer asks where a slow shipment is right now |
| `multi_tool_chain` | Agent looks up an order and processes a refund if it was late |
| `long_conversation` | Ten-turn conversation across multiple orders, refunds, and a summary email |

---

## Fault Configs

| Key | What it simulates |
|-----|-------------------|
| `clean` | No faults — baseline run |
| `api_429` | Rate limit error on every API call |
| `api_500` | Internal server error on every API call |
| `api_529` | Overload error on every API call |
| `tool_timeout` | Every tool call times out |
| `tool_malformed` | Every tool returns malformed/partial data |
| `tool_cascade` | Cascading tool failure — downstream tools blocked |
| `ctx_overflow` | Conversation history exceeds context window |
| `chaos` | Random mix of all fault types |

---

## Recovery Behaviour

The agent recovers from faults automatically:

- **API 429 / 529** — exponential backoff with up to 3 retries
- **API 500** — retries then escalates with a ticket ID if all attempts fail
- **Tool timeout** — returns a user-friendly message and continues the conversation
- **Tool malformed data** — extracts available partial fields and informs the user
- **Tool cascade failure** — gracefully reports what could not be completed
- **Context overflow** — summarizes the oldest messages and continues without losing state

---

## Output

Test results are saved as a JSON report:

```json
{
  "generated_at": "2026-05-29T...",
  "total_runs": 9,
  "passed": 9,
  "failed": 0,
  "total_errors_injected": 34,
  "total_recoveries": 31,
  "results": [ ... ]
}
```

Each result includes the scenario, fault type, errors injected, recovery actions taken, and the full conversation transcript.

---

## Tools

### `order_lookup`
Looks up an order by ID. Returns status, items, total, shipping address, and estimated delivery.

### `refund_tool`
Processes a refund for an order. Returns a confirmation ID and 3–5 business day timeline.

### `shipping_tracker`
Tracks a shipment by order ID. Returns carrier, current location, and delivery events.

---

## Requirements

- Python 3.10+
- `anthropic` SDK
- `httpx`
- Anthropic API key (`ANTHROPIC_API_KEY`)
