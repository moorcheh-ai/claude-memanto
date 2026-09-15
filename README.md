<p align="center">
  <img src="assets/logo.svg" alt="MEMANTO" width="72" height="72" align="middle">
  &nbsp;&nbsp;<b>×</b>&nbsp;&nbsp;
  <img src="assets/claude-code.svg" alt="Claude Code" width="72" height="72" align="middle">
</p>

<h1 align="center">MEMANTO × Claude Code</h1>

<p align="center">
  <strong>Persistent memory for Claude Code — store decisions, recall them semantically,<br>
  and answer questions grounded in everything the agent has learned.</strong>
</p>

<p align="center">
  <a href="https://docs.memanto.ai">Documentation</a> ·
  <a href="https://console.moorcheh.ai">Console</a> ·
  <a href="https://docs.memanto.ai/cli/overview">CLI Guide</a> ·
  <a href="https://docs.memanto.ai/integrations/claude-code">Claude Code Integration</a>
</p>

---

Claude forgets everything when a session ends. MEMANTO gives it long-term memory that
survives restarts: decisions, preferences, corrections, and open commitments stay available
across sessions, projects, and compactions.

MEMANTO is a memory companion agent built on [Moorcheh](https://moorcheh.ai), a semantic
database with zero-indexing latency. It provides:

- **Persistent memory** across sessions and restarts
- **Semantic and temporal recall** — search by meaning, or ask what was true last Tuesday
- **13 memory types** — facts, decisions, preferences, goals, errors, and more
- **Trust scoring** via confidence levels and provenance tracking
- **Conflict detection** so contradictory memories get caught instead of quietly poisoning recall
- **Expiry policy** so memory stays useful instead of only growing

## Install

```
/plugin marketplace add moorcheh-ai/claude-memanto
/plugin install memanto@moorcheh-ai
```

Then set up the CLI and an agent — or run `/memanto:quickstart` and let Claude walk you
through all of it:

```bash
pip install memanto              # Python 3.10–3.12
memanto                          # interactive setup — writes your key to ~/.memanto/.env
memanto agent create my-project  # creates and activates an agent for this project
memanto status                   # confirm it is wired up
```

Get a free API key at [console.moorcheh.ai](https://console.moorcheh.ai), or export it
yourself with `export MOORCHEH_API_KEY="..."`. You can also run
[on-prem](https://docs.memanto.ai/on-prem/quickstart) with no key at all.

### Local development

```bash
git clone https://github.com/moorcheh-ai/claude-memanto.git
claude --plugin-dir ./claude-memanto
```

## What you get

**Claude reaches for memory unprompted.** The `memanto` skill loads automatically when you
state a decision worth keeping or ask what was decided earlier, and it forbids answering
"I don't have context on that" without checking memory first.

> "Remember that we decided to use React for the frontend"
>
> "What do you remember about our authentication approach?"
>
> "What are my pending commitments?"
>
> "That's wrong — we switched to DynamoDB last month"

**`MEMORY.md` stays current on its own.** A `SessionStart` hook runs `memanto memory sync`
whenever a session starts or resumes, so Claude has full context from your first message.
A `PreCompact` hook re-syncs before compaction, so context about to be summarized away is
written to memory first — the one moment context is most likely to be lost.

**Memory operations read as English, not shell.** A `PostToolUse` hook replaces the raw
command in chat with what actually happened:

```
👾 Memanto · stored a decision (confidence 0.95)
👾 Memanto · recalled 7 memories
👾 Memanto · MEMORY.md synced — 42 memories
```

It stays silent on anything it cannot describe confidently, and never runs for non-MEMANTO
commands.

**A status line showing what memory is doing.**

```
👾 Memanto · my-project · 42 memories · +3 this session · synced 2m ago
```

Installed on the plugin's first session — it never overwrites a `statusLine` you already
have, and `/memanto:statusline remove` turns it off. It degrades honestly rather than lying:
`not configured`, `no active agent`, `session expired`, or `stale 2d ago` when `MEMORY.md`
has drifted.

**A `memory-scout` subagent** for deep background. It fans out several recalls at once —
task terms, standing conventions, past decisions, known traps, open commitments, recent
changes — and returns a short sourced brief instead of a memory dump. It also surfaces
contradictions rather than silently picking a winner. Ask for it by name, or let Claude
delegate before a refactor:

> "Use memory-scout to get background before we touch the auth module"

**Twelve commands** for when you want to drive it explicitly:

| Command | What it does |
|---|---|
| `/memanto:quickstart` | Set up MEMANTO for this project, end to end |
| `/memanto:remember` | Store something, with type/confidence/provenance chosen for you |
| `/memanto:recall` | Search by meaning, type, tag, or point in time |
| `/memanto:answer` | Answer a question grounded in memory (RAG) |
| `/memanto:capture` | Review this session and store what is worth keeping |
| `/memanto:forget` | Correct, expire, or delete a memory |
| `/memanto:upload` | Ingest a PDF/DOCX/XLSX/CSV/MD document into memory |
| `/memanto:conflicts` | Find and resolve contradictory memories |
| `/memanto:sync` | Refresh `MEMORY.md` (or export an OKF bundle) |
| `/memanto:session` | Show or switch the active agent |
| `/memanto:status` | Health check — config, session, agents, what is stored |
| `/memanto:statusline` | Install, preview, or remove the status line |

## Skills

<details>
<summary><strong>memanto</strong> — core memory operations</summary>

Storing and retrieving memory: `remember`, `recall`, `answer`, `edit`/`forget`/`expire`, file
upload, conflict resolution, expiry policy, provider migration, and `MEMORY.md` sync. Loads
automatically when relevant.

</details>

<details>
<summary><strong>memanto-cookbooks</strong> — application blueprints</summary>

End-to-end guides for building on MEMANTO: persistent agent memory, session continuity, memory
export and audit, daily summary automation, memory-powered RAG, conflict resolution, and
migrating from Mem0/Letta/Supermemory.

</details>

## What this plugin touches

Everything below happens on your machine — nothing is written outside your project and
`~/.memanto` / `~/.claude` unless you ask for it.

| Component | Effect |
|---|---|
| `SessionStart` hook | Runs `memanto memory sync`, refreshing `MEMORY.md` in the project |
| `PreCompact` hook | Runs `memanto memory sync` before context is compacted |
| `PostToolUse` hook | Summarizes `memanto` Bash calls in the transcript; never writes anything |
| Status line | Added to `~/.claude/settings.json` once, only if you have no `statusLine` set |
| `memanto` CLI | Sends memory content to the Moorcheh API under your own API key |

The skill and every command declare `allowed-tools: Bash(memanto:*)`, so they cannot run
arbitrary shell commands on your behalf. The plugin ships no MCP server, and its only network
access is the `memanto` CLI talking to Moorcheh.

### Optional: capture memory automatically at session end

`/memanto:capture` is deliberately manual, because storing memories costs API calls and writes
data you may not want. If you would rather it happen on its own, add a `Stop` hook to your own
settings — this plugin will not do it for you:

```jsonc
// .claude/settings.json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "memanto daily-summary 2>/dev/null || true"
      }]
    }]
  }
}
```

## Requirements

- Claude Code
- Python 3.10–3.12
- A [Moorcheh](https://console.moorcheh.ai) account and API key — or run
  [on-prem](https://docs.memanto.ai/on-prem/quickstart) with no key at all

## Other agents

This repository is the Claude Code plugin. MEMANTO also works with Cursor, Codex CLI, Copilot,
Windsurf, Gemini CLI, and any tool that reads the
[Agent Skills](https://agentskills.io/specification) format — see
[moorcheh-ai/memanto-agent-skills](https://github.com/moorcheh-ai/memanto-agent-skills) or run
`memanto connect list`.

## Resources

- [MEMANTO Documentation](https://docs.memanto.ai)
- [CLI Reference](https://docs.memanto.ai/cli/overview)
- [Claude Code Integration](https://docs.memanto.ai/integrations/claude-code)
- [Memory Types Reference](https://docs.memanto.ai/reference/memory-types)
- [API Reference](https://docs.memanto.ai/api-reference/authentication)
- [MCP Server](https://docs.memanto.ai/integrations/mcp)
- [Self-Hosting (On-Prem)](https://docs.memanto.ai/on-prem/quickstart)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
