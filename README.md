# agents-hub

The single source of truth for my AI agent setup. Claude Code, Codex and Cursor all get their
**skills, subagents, MCP servers and global instructions** from here, via one command:

```sh
agents-sync
```

Backed up to the public repo `sujit-gholap-121/agents-hub` (every sync commits and pushes).

## Layout

```
~/.agents/
├── skills/<name>/SKILL.md       skills (flat, the open Agent Skills format)
├── agents/<name>.md             subagents (Claude-style markdown + frontmatter)
├── mcp/servers.json             MCP servers, with ${VAR} placeholders for secrets
├── instructions/AGENTS.md       global rules for every agent
├── experimental/{skills,agents} only switched on with --with-experimental
├── groups.json                  which group each skill belongs to
├── groups/                      generated browsing view (links), don't edit
├── bin/agents-sync              the sync tool
├── .env                         secrets for servers.json (NOT in git, chmod 600)
└── state/                       what the tool manages, logs, config backups (NOT in git)
```

## What goes where

| Hub | Claude Code | Codex | Cursor |
|---|---|---|---|
| skills | links in `~/.claude/skills` | reads `~/.agents/skills` directly | reads `~/.agents/skills` directly |
| agents | links in `~/.claude/agents` | generated `~/.codex/agents/*.toml` | links in `~/.cursor/agents` (app) + per-project (CLI, see below) |
| MCP | `claude mcp add-json -s user` | generated block in `~/.codex/config.toml` | `~/.cursor/mcp.json` |
| instructions | `~/.claude/CLAUDE.md` (link) | `~/.codex/AGENTS.md` (link) | `~/.cursor/rules/agents-hub.mdc` (always-apply rule) |

**Cursor CLI and agents:** `cursor-agent` only loads subagents from the project it is opened in.
Register a project once and every sync keeps it current (its git ignores the links locally):

```sh
agents-sync --project ~/Workspace/Github/escalara-portal
agents-sync --unproject ~/Workspace/Github/escalara-portal
```

## How a sync works

1. **Import.** Anything new that an installer put straight into one app (`npx skills add`,
   the Agency Agents app, `claude mcp add`, a hand-made Codex agent) is moved into the hub.
   Secret-looking MCP values go to `.env` and become `${VAR}` placeholders.
2. **Normalize.** Agent descriptions over 160 characters are shortened so all of them fit Claude
   Code's 15k-token limit. The original text is kept in `full_description:`.
3. **Export.** The hub is written into each app. Only entries the tool manages are touched.
   App-owned items stay where they are: `~/.claude/skills/synced`, `~/.codex/skills/.system`,
   `~/.cursor/skills-cursor`, Codex plugin MCPs, and MCPs listed in `servers.json` → `unmanaged`.
4. **Group, commit, push.**

## Everyday use

```sh
# new skill
npx skills add <repo>                          # installs into ~/.agents/skills
# or: mkdir ~/.agents/skills/my-skill && $EDITOR ~/.agents/skills/my-skill/SKILL.md

# new agent
$EDITOR ~/.agents/agents/engineering-my-agent.md   # prefix = group

# new MCP server: add to mcp/servers.json, e.g.
#   "github": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-github"],
#               "env": { "GITHUB_TOKEN": "${GITHUB_TOKEN}" }, "targets": ["claude", "codex", "cursor"] }
# and put GITHUB_TOKEN=... in ~/.agents/.env

# experiments
mv my-skill ~/.agents/experimental/skills/ && agents-sync --with-experimental

agents-sync                # then restart the apps (they read these at startup)
agents-sync --dry-run      # show what would change, change nothing
```

`targets` (default: all three) limits an MCP server to some apps. `"enabled": false` turns it off
everywhere. `"extra": {"codex": {...}}` adds app-specific fields (for example Codex timeouts).

## New machine

```sh
git clone https://github.com/sujit-gholap-121/agents-hub ~/.agents
ln -s ~/.agents/bin/agents-sync ~/.local/bin/agents-sync
# recreate ~/.agents/.env (secrets are never in git), then:
agents-sync
```

## Undo

Everything is in git: `git -C ~/.agents log`, then `git revert <commit>` and `agents-sync`.
The pre-hub backups are `backup-skills-2026-11-07.tgz` and `backup-agents-2026-11-07.tgz`
(not in git), and the last Codex config before each change is in `state/codex-config.toml.bak`.
