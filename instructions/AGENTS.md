# Global instructions (all agents)

<!-- Single source: ~/.agents/instructions/AGENTS.md
     Shown to Claude Code (~/.claude/CLAUDE.md), Codex (~/.codex/AGENTS.md) and
     Cursor (~/.cursor/rules/agents-hub.mdc). Edit here, then run `agents-sync`. -->

## Skills, agents and MCP servers

- Everything is kept in one hub: `~/.agents` (see `~/.agents/README.md`).
- Install new skills into `~/.agents/skills/<name>/SKILL.md` and new subagents into
  `~/.agents/agents/<name>.md`; add MCP servers to `~/.agents/mcp/servers.json`.
  Then run `agents-sync` so every app gets them. Don't install into one app's folder only.
- Experiments go in `~/.agents/experimental/` and are only switched on with
  `agents-sync --with-experimental`.

## My preferences

<!-- Add your own rules here: coding style, languages, how to report, etc. -->
