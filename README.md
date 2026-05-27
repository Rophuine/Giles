# Giles

A Claude Code plugin that keeps agent-notes files (`CLAUDE.md`, `AGENTS.md`, project memory docs) tidy as a codebase grows.

Named for [Rupert Giles](https://en.wikipedia.org/wiki/Rupert_Giles) — Buffyverse Watcher, librarian, keeper of the record.

## What it does

When enabled, Giles teaches Claude to treat your repo's `CLAUDE.md` as an **index** that points at `docs/` sub-trees, not as a content store. As you work and learn things, Claude routes new knowledge to the right home (patterns, design docs, history) instead of accreting paragraphs in the always-loaded file. It also distinguishes "active work" from "post-completion archaeology" so in-progress status stays in whatever working record the repo uses (in-repo plan doc, external tracker, PR descriptions — Giles adapts) and doesn't pollute the agent-notes file.

The plugin ships:

- **Principle + maintenance rhythm** (loaded into every session via the plugin's `CLAUDE.md`) — shapes how Claude handles new knowledge during normal work.
- **`giles-cleanup` skill** — a one-off restructure procedure for when a repo's `CLAUDE.md` has already grown unwieldy. Triggers on phrases like "clean up CLAUDE.md", "restructure the docs", "knowledge base is getting too big".
- **`giles-distill` skill** — an end-of-session pass that filters what was learned during the session and routes the genuinely valuable items into the knowledge base. Triggers on phrases like "what did we learn?", "capture what's worth saving", "distill the session".

## Install

This repo is a Claude Code marketplace (`giles-plugins`) that currently contains one plugin (`giles`). Two-step install:

```
/plugin marketplace add <path-or-git-url>
/plugin install giles@giles-plugins
```

Where `<path-or-git-url>` is either the local filesystem path to this repo (e.g. `c:\dev\lee\claude-plugins\giles`) or its GitHub URL after pushing (e.g. `lionellpack/giles`).

After install, the skills appear as `giles:giles-cleanup` and `giles:giles-distill` in the available-skills list.

## When the cleanup skill applies

- Your `CLAUDE.md` is past ~250 lines.
- A section is monotonically accreting (every new milestone / pattern adds a paragraph).
- Duplication between `CLAUDE.md` and `docs/*` — the prose is in both places and rotting.
- The user asks to "clean up", "restructure", or "organise" their agent notes.

## When the distill skill applies

- You're wrapping up a session that involved substantive work — debugging, design, exploration.
- You want to capture what was learned before the session ends and the context is gone.
- The user asks "what did we learn?", "capture what's worth saving", "distill the session".

The skill filters candidates for what's genuinely worth keeping (expensive to rediscover, non-obvious, important to correctness, useful to future agents) and routes the survivors into the existing knowledge base per Giles' index-not-dump discipline.

## Layout

```
giles/                                       # marketplace repo root
├── .claude-plugin/
│   └── marketplace.json                     # marketplace catalog (lists plugins)
├── plugins/
│   └── giles/                               # the Giles plugin
│       ├── .claude-plugin/
│       │   └── plugin.json                  # plugin manifest
│       ├── CLAUDE.md                        # loaded as session context when plugin enabled
│       └── skills/
│           ├── giles-cleanup/
│           │   └── SKILL.md                 # one-off restructure procedure
│           └── giles-distill/
│               └── SKILL.md                 # end-of-session capture procedure
├── LICENSE
└── README.md
```
