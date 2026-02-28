# outlook-mcp

MCP server exposing Microsoft Outlook operations (read, send, calendar, classify) via COM automation. Provides the write layer that WorkIQ MCP lacks.

## Why This Exists

[Lumen](https://github.com/katundwa_microsoft/lumen) is an investigation agent that monitors email for bug reports and delivers root-cause analysis. It uses WorkIQ MCP for Outlook access, but WorkIQ:
- **Read-only** — can't send replies, schedule meetings, or tag triaged emails
- **Unreliable** — frequent timeouts (2-min cap per spawn)

This MCP server fills both gaps: reliable reads + full write support. Usable from any MCP client (Claude Code, Lumen, Claude Desktop).

## Tech Stack

- **Python 3.13** on Windows 11
- **FastMCP** (`mcp` package) — MCP server framework
- **pywin32** (`win32com.client`) — Outlook COM automation
- **Outlook Classic** (Win32 desktop app) — COM not supported in "new Outlook"
- **Transport**: stdio (each consumer spawns its own instance)

## Architecture

```
Claude Code ─┐
Lumen ───────┼── stdio ──→ outlook-mcp (Python) ── COM ──→ Outlook.exe
Claude Desktop┘
```

- Stateless: per-call COM init/teardown (~10ms overhead, resilient to Outlook restarts)
- No agent logic here — the LLM client decides what to do; this server just executes
- See `docs/design-decisions.md` for full tradeoff analysis

## Tools (15 total)

**Setup:**
`initialize` — checks dependencies, probes Outlook, builds user profile, generates policy
`get_diagnostics` — returns runtime diagnostics (logging/config state + recent audit tail)

**P0 — Write (WorkIQ can't do these):**
`send_email`, `reply_to_email`, `forward_email`, `categorize_email`, `move_email`, `create_meeting`, `cancel_meeting`

**P1 — Read (WorkIQ does these but unreliably):**
`read_inbox`, `read_email`, `search_emails`, `list_calendar`

**P2 — Metadata:**
`list_folders`, `list_categories`

## First Run

Call the `initialize` tool on first connection. It:
1. Checks dependencies (Python, pywin32, Outlook, Claude CLI, agency CLI)
2. Creates `~/.outlook-mcp/` with default `config.toml`
3. Probes Outlook COM for the current user's name and email
4. If agency CLI is available, queries WorkIQ for org chart (direct reports, leadership chain)
5. Generates `~/.outlook-mcp/policy.md` with user profile + safety rules

If agency is unavailable or WorkIQ times out, the policy file is generated with what's available (name + email from Outlook) and TODO markers for org data. Edit `~/.outlook-mcp/policy.md` manually to fill in the rest.

## Safety

Write tools pass through two layers before reaching Outlook:

1. **Hard guardrails** (`guards.py`) — instant, code-enforced: recipient cap, rate limit, blocked recipients, audit log
2. **AI policy evaluation** (`policy.py`) — separate Claude CLI instance evaluates against `~/.outlook-mcp/policy.md` rules. Fail-closed.

Configured via `~/.outlook-mcp/config.toml` (limits) and `~/.outlook-mcp/policy.md` (natural language rules).

Intentional omissions: no delete tool, no attachment support on send.

## Key Constraints

- Outlook Classic must be running
- COM is STA — all calls on the thread that called `CoInitialize()`
- Outlook Classic supported until at least 2029

## Project Structure

```
email-assistant/
├── CLAUDE.md                 # This file
├── policy.md                 # Example policy template (reference only)
├── docs/
│   ├── research.md           # COM approach analysis
│   └── design-decisions.md   # Architecture decisions with tradeoffs
├── src/
│   └── outlook_mcp/
│       ├── __init__.py
│       ├── server.py         # FastMCP server + 15 tool definitions
│       ├── outlook.py        # COM automation layer (13 Outlook operations)
│       ├── setup.py          # Initialization: dep checks, profile, policy gen
│       ├── guards.py         # Hard guardrails (recipient cap, rate limit, blocklist)
│       ├── policy.py         # AI policy evaluator (Claude CLI)
│       └── config.py         # Config loading from ~/.outlook-mcp/
├── tests/
└── pyproject.toml
```

## Dev Commands

```bash
uv run pytest -m "not e2e"          # Unit tests (no Outlook needed)
uv run pytest -m e2e                # E2E tests (Outlook must be running)
uv run pytest                       # All tests
uv run ruff check src/ tests/       # Lint
uv run mypy src/ tests/             # Type check
```

## Configuration

Registered in user-level config (`~/.claude.json`) so it's available in every Claude Code session:

```json
{
  "mcpServers": {
    "outlook": {
      "command": "uv",
      "args": ["--directory", "<path-to-email-assistant>", "run", "outlook-mcp-server"]
    }
  }
}
```
