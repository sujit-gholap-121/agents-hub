---
name: herdr
description: Drive the Herdr desktop app and herdr-core (hc) on this Mac - create/rename spaces and tabs, split panes, start Claude Code / Codex / Cursor agents in panes, name panes, read or message other agents' terminals, wait on them. Use whenever the user mentions Herdr, spaces, tabs, panes, splits, or running several agents side by side, or when running inside a Herdr terminal ($HC_TERMINAL_ID is set).
---

# Herdr

Two programs, both on PATH (`~/.local/bin`):

- `herdr-app` - the window: spaces > tabs > panes, layouts, mods. Config lives in `~/.config/herdr-app/` (its `AGENTS.md` is the full manual).
- `hc` (herdr-core) - runs every terminal and agent, keeps them alive when the window closes.

The full, version-accurate manuals are printed by the tools themselves. Read the one you need before acting:

```sh
herdr-app --skill   # driving the window (spaces, tabs, panes, layouts)
hc --skill          # running, reading, messaging and waiting on agents/programs
herdr-app commands  # every app command and its arguments (JSON)
```

## Inspect first

```sh
herdr-app ls        # spaces > tabs > terminals, with pane IDs
herdr-app status    # focused space/tab/pane, layout, errors
hc ls               # every live terminal (address like t7, agent, cwd, where)
```

## Hard-won rules (read these - they cost many attempts to learn)

1. **Always target panes explicitly.** When a script runs from inside a Herdr terminal, implicit
   targets (`terminal.open --place right`, `pane.split` with no `--pane`) act on the *caller's own
   tab*, not the tab the user is looking at. `space.new` from a script does not focus the new space.
   So: create, read the returned IDs, then pass `--pane <id>`.
2. **`pane.split` with `--pane` is the reliable way to add panes anywhere:**
   ```sh
   herdr-app do pane.split --direction right --pane <pane-id> --argv '["claude"]'
   herdr-app do pane.split --direction down  --pane <pane-id> --argv '["codex"]'
   ```
   Each prints `{ "pane", "terminal" }`; chain on the new pane ID.
3. **Closing the only pane of a tab closes the space** (asks for `--confirmed true`). Add the new
   panes first, then close the placeholder shell.
4. **Names.** Spaces and tabs: `space.new --name X --tab Y`, `space.rename`, `tab.rename`.
   Panes can't be renamed after start. This Mac's layout `mine` uses the `pane-named` header mod,
   which shows a `title` label: `hc label <terminal> title=Codex`.
   `hc run --name` only accepts `a-z 0-9 - _` and is the core name, not the header.
5. **`hc run -- <cmd> --json`** passes `--json` to the program. Put hc flags before `--`.
6. Pane sizes have no command; the user drags dividers.
7. Agent CLIs: Claude Code `claude`, Codex `codex`, Cursor `cursor-agent`.

## Recipe: a named space with Claude | Cursor / Codex

```sh
J() { python3 -c "import json,sys;print(json.load(sys.stdin)['$1'])"; }
out=$(herdr-app do space.new --name "AI Agents" --tab "Agents" --folder "$PWD")
SP=$(echo "$out" | J space)
SHELL_PANE=$(herdr-app ls | python3 -c "import json,sys
d=json.load(sys.stdin)
print([s for s in d['spaces'] if s['id']=='$SP'][0]['tabs'][0]['terminals'][0]['pane'])")
C=$(herdr-app do pane.split --direction right --pane $SHELL_PANE --argv '["claude"]')
U=$(herdr-app do pane.split --direction right --pane $(echo "$C" | J pane) --argv '["cursor-agent"]')
X=$(herdr-app do pane.split --direction down  --pane $(echo "$U" | J pane) --argv '["codex"]')
herdr-app do pane.close --pane $SHELL_PANE
herdr-app do space.select --id $SP
hc label $(echo "$C" | J terminal) title=Claude
hc label $(echo "$U" | J terminal) title=Cursor
hc label $(echo "$X" | J terminal) title=Codex
```

## Working with other agents

```sh
hc read t7 --lines 50          # what's on its screen
hc send t7 "review src/x.rs"   # type a message + Enter
hc wait t7                     # block until it stops working (never sleep-poll)
hc run --bg --name helper --cwd "$PWD" codex "task..."   # background sub-agent
```

Check your work with `herdr-app status` (errors) and `herdr-app shot file.png` (a screenshot of the window).
