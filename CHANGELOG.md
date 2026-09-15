# Changelog

All notable changes to the MEMANTO Claude Code plugin are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0]

First release from this repository. The plugin previously shipped from
[moorcheh-ai/memanto-agent-skills](https://github.com/moorcheh-ai/memanto-agent-skills)
alongside the Cursor and Codex integrations; this is the same plugin, packaged on its own for
Claude Code.

### Added

- `memanto` skill — remember, recall, answer, edit/forget/expire, file upload, conflict
  resolution, expiry policy, and `MEMORY.md` sync.
- `memanto-cookbooks` skill — blueprints for persistent agent memory, session continuity,
  memory export and audit, daily summaries, memory-grounded RAG, and migration from
  Mem0/Letta/Supermemory.
- `memory-scout` subagent — fans out several recalls and returns a sourced brief.
- Twelve commands: `quickstart`, `remember`, `recall`, `answer`, `capture`, `forget`,
  `upload`, `conflicts`, `sync`, `session`, `status`, `statusline`.
- `SessionStart` and `PreCompact` hooks that keep `MEMORY.md` current.
- `PostToolUse` hook that renders `memanto` Bash calls as readable summaries.
- Status line showing agent, memory count, memories added this session, and sync freshness.
