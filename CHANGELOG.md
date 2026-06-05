# Changelog

All notable changes to the Giles plugin are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the plugin adheres
to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] — 2026-06-05

### Added

- **Routing directives.** Both skills now consult the global (`~/.claude/CLAUDE.md`)
  and repo `CLAUDE.md` for explicit storage/routing instructions and honour any
  they find — without anyone editing the skills. A user or repo can declare where
  certain kinds of knowledge should go; the destinations live in `CLAUDE.md`, so
  the skills stay decoupled from any specific store.
  - `giles-distill` checks for directives at the start of its Route step and
    reports applied directives in its end-of-pass summary.
  - `giles-cleanup` checks during inventory, folds directive-routing into the
    step-2 proposal (so it's covered by the user's sign-off), and honours it at
    extraction.
- Directives are **additive by default** (route matching items to the named store
  *as well as* the normal repo-KB destinations) and divert only when they say
  *instead*. They never change *what* is kept, only *where* it goes.
- An `instead` directive can divert the cross-repo / user-global branch that would
  otherwise reach auto-memory or a global-instructions file — the one case where a
  directive routes outside the repo-KB destinations. This makes "use my external
  store instead of auto-memory" fully expressible.

### Changed

- Both skills' verify steps now accept an index entry that resolves to an external
  store (a record id/URL) named by a routing directive, not only a local file.

## [0.3.0] — 2026-05-28

### Changed

- Reframed Giles as **steward, not clerk**: routine structural calls (new sub-tree,
  new top-level doc, new section) are made directly and named in the reply;
  escalation to the user is reserved for genuine forks.

## [0.2.0] — 2026-05-27

### Added

- **`giles-distill` skill** — an end-of-session pass that filters what was learned
  during a session and routes the genuinely valuable items into the knowledge base.

### Changed

- Renamed the cleanup skill to **`giles-cleanup`**.

## [0.1.0] — 2026-05-27

### Added

- Initial plugin packaging: restructured the repo into a Claude Code marketplace
  (`giles-plugins`) shipping the **Giles** plugin — the index-not-dump principle
  (loaded via the plugin's `CLAUDE.md`) plus the cleanup skill.
