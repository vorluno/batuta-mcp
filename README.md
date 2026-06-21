<div align="center">

<img src="./assets/banner.jpg" alt="batuta-mcp — disjoint plans and git worktrees for parallel agents" width="100%" />

# batuta-mcp

**Split a task into plans with disjoint file boundaries, then scaffold a git worktree per plan — so parallel agents never step on each other.**

A stateless MCP server that turns a brain dump into conflict-free parallel work.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)
[![Bun](https://img.shields.io/badge/Bun-1.3+-fbf0df?logo=bun&logoColor=black)](https://bun.sh)
[![Model Context Protocol](https://img.shields.io/badge/MCP-compatible-6E56CF)](https://modelcontextprotocol.io)
[![Built with Claude Code](https://img.shields.io/badge/Built_with-Claude_Code-D97757)](https://claude.com/claude-code)

</div>

---

## Why

Running several coding agents in parallel is fast — until two of them edit the same file and silently clobber each other's work. The fix isn't live coordination; it's **separation**: give each agent a set of files that **don't overlap**, and the conflict can't happen by design.

**batuta-mcp** is the brain that does that split. You hand it a brain dump; it returns plans with **disjoint file boundaries**, and scaffolds an isolated git worktree for each one.

## How it works

It's a **stateless** MCP server (stdio). No database, no daemon, no web UI — just three tools that compose:

1. **Decompose** a brain dump into 2–5 plans whose `fileBoundaries` don't overlap (auto-corrects once if they do).
2. **Check** that the boundaries are truly disjoint.
3. **Scaffold** a `git worktree` per plan, returning a ready-to-paste prompt for each — open them in separate terminals/tabs and let one agent work each, conflict-free.

The "muscle" (running the agents, the terminals) stays in your editor; batuta-mcp is just the planning brain.

## Features

- 🧠 **Disjoint decomposition** — plans are generated so no file appears in two plans.
- ♻️ **Auto-correction** — if the model returns overlapping boundaries, it retries once to separate them.
- 🌳 **Worktree scaffolding** — one isolated `git worktree` per plan, with a ready-to-paste prompt.
- 🔒 **Safe by design** — pre-flight checks (valid git repo, no overlaps, no path traversal) before touching disk; `dryRun` previews without writing.
- 🪶 **Stateless** — nothing persisted; uses your logged-in `claude` CLI (no API key needed).

## Requirements

- [Bun](https://bun.sh) 1.3+
- `git` 2.x (for `scaffold_worktrees`)
- The [`claude`](https://claude.com/claude-code) CLI, logged in (for `decompose_into_plans`)
- [Claude Code](https://claude.com/claude-code) or any MCP client

## Installation

```bash
git clone https://github.com/vorluno/batuta-mcp.git
cd batuta-mcp
bun install

claude mcp add batuta -- bun run /absolute/path/to/batuta-mcp/src/index.ts
```

## Configuration

Any MCP client works. The `mcpServers` entry:

```json
{
  "mcpServers": {
    "batuta": {
      "command": "bun",
      "args": ["run", "/absolute/path/to/batuta-mcp/src/index.ts"]
    }
  }
}
```

For **Claude Code**, `claude mcp add` (above) writes this for you. For **Warp** or **Cursor**, paste the snippet into their MCP settings.

## Tools

| Tool | Description |
|------|-------------|
| `decompose_into_plans` | `{ brainDump, projectHint?, repoPath? }` → `{ plans, overlapsResolved, attempts }`. Splits the work into plans with disjoint boundaries (auto-corrects overlaps once). |
| `check_boundary_overlaps` | `{ plans }` → `{ overlaps, ok }`. Pure check: do any plans share files? |
| `scaffold_worktrees` | `{ repoPath, plans, dryRun? }` → `{ results }`. Pre-flight, then `git worktree add` per plan + a ready-to-paste prompt. |

## Typical flow

1. `decompose_into_plans` → plans with disjoint boundaries.
2. `check_boundary_overlaps` → confirm `ok: true` (or adjust).
3. `scaffold_worktrees` → creates the worktrees; open each in its own terminal/tab and paste its `suggestedPrompt`.

## Development

```bash
bun test          # full suite
bunx tsc --noEmit # type-check
```

Built test-first across 10 TDD tasks with per-task and whole-branch review.

## License

[MIT](./LICENSE) © 2026 Vorluno

---

<div align="center">

Built by **[Vorluno](https://vorluno.dev)** — a software studio from Panamá 🇵🇦

Part of the [`mcp-s`](https://github.com/vorluno/mcp-s) family of MCP servers.
Looking for live coordination between sessions instead of separation? See [`agora`](https://github.com/vorluno/agora-mcp).

</div>
