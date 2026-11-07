# Shared agent skills

One home for skills used by Claude Code, Codex and Cursor.

```
~/.agents/
├── skills/             THE REAL SKILLS (flat: skills/<name>/SKILL.md). Edit here.
├── skills-by-group/    Grouped view - links only, for browsing.
├── .skill-lock.json    Used by `npx skills` (vercel-labs/skills) to track and update skills.
└── backup-skills-2026-11-07.tgz   Backup taken before the move.
```

## Who reads what

| Agent       | Reads                                                         |
|-------------|---------------------------------------------------------------|
| Codex       | `~/.agents/skills` directly                                   |
| Claude Code | `~/.claude/skills/<name>` → link to `~/.agents/skills/<name>` |
| Cursor      | `~/.claude/skills` and `~/.agents/skills`                     |

Built-in skills stay where their app manages them, so they're not moved:
`~/.claude/skills/synced` (Claude app), `~/.codex/skills/.system` (Codex),
`~/.cursor/skills-cursor` (Cursor).

## Groups

| Group                  | Skills |
|------------------------|--------|
| agents-and-terminals   | herdr, call-cursor-agent, find-skills, full-output-enforcement |
| design-ui              | apple-design, design-taste-frontend(-v1), gpt-taste, high-end-visual-design, minimalist-ui, industrial-brutalist-ui, stitch-design-taste, redesign-existing-projects, web-design-guidelines, image-to-code |
| mockups-and-branding   | imagegen-frontend-web, imagegen-frontend-mobile, brandkit |
| higgsfield-media       | all higgsfield-* |
| backend-database       | supabase, supabase-postgres-best-practices |
| testing-browser        | playwright-cli |
| reverse-engineering    | reverse-engineer-anything |

## Adding a skill

```sh
# 1. put it in the shared folder
mv my-skill ~/.agents/skills/            # (or: npx skills add <repo> - it installs here)
# 2. let Claude Code see it
ln -s ../../.agents/skills/my-skill ~/.claude/skills/my-skill
# 3. file it in a group
ln -s ../../skills/my-skill ~/.agents/skills-by-group/<group>/my-skill
```

Keep `skills/` flat. Agents only discover `skills/<name>/SKILL.md`, so put groups in
`skills-by-group/`, never as subfolders of `skills/`.
