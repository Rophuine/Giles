# Giles — Claude Code plugin marketplace

This repo is a Claude Code plugin marketplace (`giles-plugins`, per [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json)) that ships a single plugin: **Giles** ([plugins/giles/](plugins/giles/)). Giles defines a discipline for keeping agent-notes files (`CLAUDE.md`, `AGENTS.md`) as indexes that point at `docs/` sub-trees rather than as content dumps. It ships two skills: a one-off cleanup for restructuring sprawling agent-notes files, and an end-of-session distill for capturing valuable learnings into the knowledge base.

The plugin is the product; the marketplace is its delivery mechanism. The marketplace name `giles-plugins` is provisional (Leo's plugin naming follows a role-based convention — Giles the librarian — and may consolidate into a multi-plugin marketplace if more ship).

See [README.md](README.md) for human-facing docs (what it does, install instructions).

## Layout

- [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) — marketplace catalog. Lists the plugin and points at its source dir.
- [plugins/giles/CLAUDE.md](plugins/giles/CLAUDE.md) — **the authoritative content for what Giles teaches Claude.** Loads as session context when the plugin is installed in another repo. Any changes to the principle, rhythm, or examples go here.
- [plugins/giles/skills/giles-cleanup/SKILL.md](plugins/giles/skills/giles-cleanup/SKILL.md) — the cleanup-pass skill. Procedure for restructuring sprawling agent-notes files.
- [plugins/giles/skills/giles-distill/SKILL.md](plugins/giles/skills/giles-distill/SKILL.md) — the end-of-session distill skill. Filters session learnings for what's valuable and routes them into the knowledge base.
- [plugins/giles/.claude-plugin/plugin.json](plugins/giles/.claude-plugin/plugin.json) — plugin manifest.

## This repo applies its own discipline

Giles ships the index-not-dump principle, so this CLAUDE.md is deliberately short. The authoritative content for the discipline lives in [plugins/giles/CLAUDE.md](plugins/giles/CLAUDE.md); don't duplicate the rules here — two sources of truth would rot.

Routing for new knowledge:

- **What Giles teaches Claude** (principle, rhythm, examples) → edit [plugins/giles/CLAUDE.md](plugins/giles/CLAUDE.md).
- **The cleanup procedure** → edit [plugins/giles/skills/giles-cleanup/SKILL.md](plugins/giles/skills/giles-cleanup/SKILL.md).
- **The distill procedure** → edit [plugins/giles/skills/giles-distill/SKILL.md](plugins/giles/skills/giles-distill/SKILL.md).
- **Patterns or decisions about plugin development itself** (skill authoring, plugin packaging gotchas, marketplace structure decisions) — if it earns a paragraph, create `docs/<topic>.md` or `docs/patterns/<slug>.md` and add a one-line index entry here. Don't pre-create those dirs; introduce them with the first real entry.

## Testing changes locally

The plugin's CLAUDE.md and skills don't auto-load while you're editing them in this repo — they load only when the plugin is *installed*. To dogfood:

```
/plugin marketplace add c:\dev\lee\claude-plugins\giles
/plugin install giles@giles-plugins
```

Skills appear as `giles:giles-cleanup` and `giles:giles-distill` after install; plugin's CLAUDE.md loads into session context. Open a target repo (e.g. Lounger) to verify the principle and skills are visible and behave as intended.

## Commits

Match the repo's existing git history style.
