# outlook-mcp

MCP server that gives Claude (and other AI agents) full access to Microsoft Outlook — send emails, schedule meetings, classify messages, and more. Uses COM automation via the locally installed Outlook Classic app. No Graph API or app registration needed.

## Prerequisites

- **Windows 11** with **Outlook Classic** installed
- **[uv](https://docs.astral.sh/uv/)** (v0.5+) — Python package manager
- **[Claude Code](https://claude.ai/download)** (v2.1+)

Optional:
- **agency CLI** — enables automatic org chart lookup during setup

## Setup (one command)

Find your full path to `uv.exe` first — don't rely on PATH:

```bash
where uv
# Typically: C:\Users\<you>\.local\bin\uv.exe
```

Register the server with Claude Code:

```bash
claude mcp add --scope user outlook -- \
  C:\Users\<you>\.local\bin\uv.exe \
  --from "git+https://github.com/katundwa_microsoft/outlook-mcp" \
  outlook-mcp-server
```

This writes the config to `~/.claude.json` under `mcpServers`. No git clone needed — uv fetches the package directly.

### **⚠ You MUST restart Claude Code after running this command.**

MCP server config is only read at session startup. If you skip this step, `/mcp` will show "No MCP servers configured" even though the config is correct. This is the most common setup issue.

Close your terminal and start a new `claude` session.

## Verify

In your new Claude Code session:

1. **Check connection**: Run `/mcp` — you should see `outlook` listed as **connected**.
2. **Check health**: Ask Claude to run the `get_diagnostics` tool with dependency checks enabled. All checks should show `ok: true`.
3. **Test a read**: Ask Claude to read your recent inbox — this confirms Outlook COM access works.

If all three pass, you're done. The server auto-initializes on first use (creates `~/.outlook-mcp/` with config and policy files).

### Optional: org-aware policy

Call the `initialize` tool to fetch your org chart from WorkIQ (requires agency CLI). This generates personalized safety rules in `~/.outlook-mcp/policy.md` based on your direct reports and leadership chain.

## Troubleshooting

### `/mcp` shows "No MCP servers configured"

You haven't restarted Claude Code since registering the server. Close terminal, start a new session.

### `/mcp` shows outlook as **errored**

1. **Outlook not running** — start Outlook Classic (the desktop app, not New Outlook) before launching Claude Code.
2. **Wrong uv path** — verify the path in `~/.claude.json` matches your actual `uv.exe` location. Run `where uv` to check.
3. **Python version** — this server requires Python 3.13+. Run `uv python list` to check available versions.

### Server was working but stopped

Outlook Classic must be running for the entire session. If Outlook crashes or gets closed, the server loses COM access. Restart Outlook, then restart Claude Code.

### Config location

The `claude mcp add` command writes to `~/.claude.json` (in your home directory). Look for the `mcpServers.outlook` entry. Don't edit `~/.claude/mcp.json` — it's unreliable in some Claude Code versions.

## Safety

Every write operation (send, reply, forward, schedule) passes through two layers:

1. **Hard guardrails** — code-enforced limits (max recipients, rate limit, blocked addresses). Configured in `~/.outlook-mcp/config.toml`.
2. **AI policy evaluation** — a separate Claude instance reviews each action against your `~/.outlook-mcp/policy.md` rules. If it can't decide, the action is blocked.

No delete tool. No attachment support on send. These are intentional omissions.

## Tools

| Tool | Description |
|------|-------------|
| `initialize` | Check deps, build profile, generate policy |
| `get_diagnostics` | Return runtime diagnostics and recent audit tail |
| `send_email` | Send a new email |
| `reply_to_email` | Reply (or reply-all) to an existing email |
| `forward_email` | Forward an email |
| `categorize_email` | Set categories/tags on an email |
| `move_email` | Move email to a folder |
| `create_meeting` | Send a meeting invitation |
| `cancel_meeting` | Cancel a meeting you organized |
| `read_inbox` | Read recent inbox messages |
| `read_email` | Read a single email with full body |
| `search_emails` | Search by subject/body |
| `list_calendar` | List events in a date range |
| `list_folders` | List all mailbox folders |
| `list_categories` | List available categories |
| `create_folder` | Create a new mail folder |
| `create_category` | Create a new category/tag |

## Development

```bash
uv run pytest -m "not e2e"     # Unit tests (no Outlook needed)
uv run pytest -m e2e           # E2E tests (Outlook must be running)
uv run ruff check .            # Lint
uv run mypy src/ tests/        # Type check
```
