# Contributing to MEMANTO for Claude Code

Thank you for your interest in contributing!

## How to Contribute

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-change`)
3. Add or improve files following the structure below
4. Validate (see below) and submit a pull request

## Repository Structure

```
.claude-plugin/
├── plugin.json        # Claude Code plugin manifest
└── marketplace.json   # Marketplace entry, so this repo can be added directly
agents/                # Subagents — one .md per agent
commands/              # Slash commands — one .md per command
hooks/
├── hooks.json         # SessionStart, PreCompact, PostToolUse
├── session_start.py   # MEMORY.md refresh + one-time status line install
└── notify.py          # PostToolUse notice renderer
skills/<skill-name>/
├── SKILL.md           # Skill definition (required, needs YAML frontmatter)
└── references/*.md    # Detailed reference guides
statusline.py          # Status line renderer
assets/logo.svg        # Plugin logo
```

This repository is the Claude Code plugin only. The cross-agent distribution — Cursor,
Codex CLI, Copilot, Windsurf, and the raw Agent Skills bundle — lives in
[moorcheh-ai/memanto-agent-skills](https://github.com/moorcheh-ai/memanto-agent-skills).
The skills, commands, agent, hooks, and status line are shared between the two; change them
here and port the change there, or vice versa.

## Python in hooks and the status line

Both are stdlib-only Python 3.10+ — `memanto` already requires Python, so this adds no
dependency. Two constraints they must keep:

- **Never crash.** A hook or status line that raises disrupts the session it decorates. Guard
  every step and degrade to shorter output or none.
- **Write UTF-8 explicitly.** Windows consoles default to cp1252, which cannot encode the
  `👾` brand mark — without an explicit reconfigure the whole line silently vanishes.

`hooks.json` registers both `python` and `python3` for every Python hook because neither
spelling exists everywhere. Whichever is missing fails harmlessly, so anything they invoke must
be idempotent — `session_start.py` and `notify.py` both claim a lock file so the work runs once
even when both interpreters resolve.

A plugin cannot ship a `statusLine`: plugin `settings.json` only honors `agent` and
`subagentStatusLine`. The entry is written into the user's `~/.claude/settings.json`, once,
and never over an existing one.

## Subprocess output

Anything that shells out to `memanto` must pass `encoding="utf-8", errors="replace"` to
`subprocess.run`. The CLI emits Rich box-drawing and spinner glyphs; without an explicit codec
Python decodes them with the locale code page and the reader thread dies, silently discarding
the output.

## SKILL.md Format

YAML frontmatter is **required** — without it the skill violates the
[Agent Skills spec](https://agentskills.io/specification) and other tools will not load it.

```markdown
---
name: my-skill
description: What it does and when to use it. Lead with the trigger, since this is what the model reads to decide whether to load the skill.
allowed-tools: Bash(memanto:*)
license: MIT
---

# Skill Name

Body content.
```

Keep `description` specific about *when* to use the skill, not just what it is. It is the only
part always in context.

## Command Format

Commands are **prompts**, not documentation. Write instructions addressed to the agent, and
consume the user's input via `$ARGUMENTS`.

```markdown
---
description: One line, shown in the / menu
argument-hint: <what the user should type>
allowed-tools: Bash(memanto:*)
disable-model-invocation: true
---

Do the thing with:

$ARGUMENTS

1. Step one...
```

All commands here set `disable-model-invocation: true`. The `memanto` skill is the single path
Claude takes on its own; commands are user-invoked shortcuts. This keeps twelve command
descriptions out of every conversation's context.

## Guidelines

- **Verify every command against the installed CLI** before documenting it. Run
  `memanto <cmd> --help`. Most bugs in this repo have been documentation drifting ahead of or
  behind the CLI.
- Prefer the `memanto` CLI over raw HTTP. The CLI talks to Moorcheh directly and needs no
  server; raw HTTP against `localhost:8000` requires `memanto serve` to be running. If a
  cookbook does show the REST API, state that prerequisite explicitly.
- Reference bundled files with `${CLAUDE_PLUGIN_ROOT}` or `${CLAUDE_SKILL_DIR}`, never
  repo-relative paths — those break once the plugin is installed elsewhere.
- Keep the `version` in `plugin.json` and `marketplace.json` in sync, and add a
  [CHANGELOG](CHANGELOG.md) entry.
- Match the existing style in `skills/memanto/`.

## Validating

Before opening a PR:

```bash
# Marketplace manifest (this is what `validate .` picks up at the repo root)
claude plugin validate . --strict

# Plugin manifest
claude plugin validate ./.claude-plugin/plugin.json --strict

# Components
claude plugin validate ./skills --strict
claude plugin validate ./commands --strict
claude plugin validate ./agents --strict
```

`--strict` turns warnings into failures, which is what CI should use — missing frontmatter and
unknown manifest fields are warnings the runtime tolerates but that break other tools.

Then install it locally and exercise it end to end:

```bash
claude --plugin-dir .
```

## Reporting Issues

Open an issue at
[github.com/moorcheh-ai/claude-memanto](https://github.com/moorcheh-ai/claude-memanto/issues).
