# Giles — agent-notes discipline

When this plugin is enabled, treat every repo's agent-notes file (`CLAUDE.md`, `AGENTS.md`, or equivalent) as an **index**, not a content store. The rules below govern how new knowledge enters the repo during normal work; the `knowledge-base-cleanup` skill (shipped with this plugin) handles the one-off restructure when an existing repo has already grown unwieldy.

## Principle: index, not dump

A repo's `CLAUDE.md` is an **index**, not a content store. Deep content — patterns, design discussions, troubleshooting recipes, milestone history — lives in `docs/` sub-trees that the index points at and that load on demand. The agent's job is to keep that structure tidy as the repo grows.

Typical sub-trees that emerge in repos using this discipline. The patterns + topic-docs shapes are near-universal; the milestone-based docs only apply when a repo organises in-progress work as in-repo numbered milestones (adapt to the repo's actual planning convention if different — see "Active work vs archaeology" below).

- **Near-universal**:
  - `docs/patterns/<slug>.md` — distilled recurring patterns (one file per pattern, indexed by one-liners in `CLAUDE.md`).
  - `docs/<topic>.md` — design discussions, troubleshooting recipes, anything that earns its own page.
- **When the repo uses in-repo milestone plans**:
  - `docs/m<n>-<topic>.md` — active milestone working doc (spec + per-task implementation notes during active work).
  - `docs/history-m<n>.md` — completed-milestone archaeology (created by renaming the working doc on milestone completion).

When you learn something new in a repo:

- **Fits an existing category** — write it to that sub-tree, add a one-line index entry in `CLAUDE.md`. Don't expand the index entry; the file owns the prose.
- **Doesn't fit any existing category** — surface the gap to the user before creating a new sub-tree or top-level doc. Introducing a new category is a structural decision, not a silent action.

## Active work vs archaeology

Active work has a separate rhythm from the static knowledge base. Wherever current status, running notes, and per-task implementation details live, they do NOT belong in `CLAUDE.md` — status reports bloat the always-loaded context.

**The shape of "the working doc" is the repo's call** and depends on how the repo organises in-progress work. Apply the principle cautiously; don't impose a structure the repo doesn't use:

- **In-repo milestone plans** (e.g. `docs/m<n>-<topic>.md`): the plan doc IS the working doc. Don't pre-create a `docs/history-m<n>.md` for an active milestone — history docs are completion artifacts. On milestone completion, condense the working doc and rename it.
- **External trackers** (Linear, Jira, GitHub Issues, etc.): the tracker IS the working record. `CLAUDE.md` should not mirror ticket status; if a pointer to "what's currently in flight" is useful, name the tracker, don't copy state.
- **No formal in-repo planning** (just commits + PRs): git / PR history IS the working record. `CLAUDE.md` still doesn't accrete status; reusable lessons still extract to `docs/`.
- **Other conventions** (sprints, ADRs, RFCs, feature-branch READMEs, design-doc folders, etc.): same pattern — find where the repo already records in-progress work and let it stay there.

When you learn something *reusable* during active work (a pattern, a cross-cutting decision worth a dedicated doc), extract it into the knowledge-base structure with a pointer back instead of letting it accrete in the working doc — or worse, in `CLAUDE.md`. At natural completion boundaries (milestone end, sprint end, feature ship, whatever the repo uses), do an extraction-and-condense pass: what reusable lessons surfaced? Where do they belong in the structure? What can be archived?

If you don't know what the repo's working-doc convention is, ask the user before introducing one. Inventing a milestone structure (or any new structure) for a repo that doesn't use one is the same category-introduction mistake the principle warns about.

## Hand-off prompts are pointers, not knowledge dumps

Same principle applied to transient session-to-session text: hand-off prompts (and any "context for the next session" notes) should be ≤5 sentences telling the new session WHERE to read in the repo's structure — not what to think. If you catch yourself writing "and the new session should also know X" in a hand-off, that's the cue to write X into the appropriate repo file first, then point at it. The structure is the memory; the hand-off is just a pointer into it.

## When to suggest a cleanup pass

Surface to the user (don't act silently) when you notice:

- `CLAUDE.md` (or equivalent) > 250 lines.
- A section is monotonically accreting (every new milestone / pattern / decision adds a paragraph inline).
- Any `docs/*.md` file > ~600 lines (consider sub-splitting).
- A Patterns-style index > 40 entries (consider regrouping by topic).
- Duplication between `CLAUDE.md` and `docs/*` — prose in both places, rotting.

The procedure for the restructure is the `knowledge-base-cleanup` skill — invoke via the Skill tool. Also triggered by phrases like "clean up CLAUDE.md", "restructure the docs", "/kb-cleanup".

## Greenfield repos

When working in a fresh repo with no `CLAUDE.md` (or just a one-paragraph stub), apply the principle from the start: shape any new agent-notes file as an index that anticipates `docs/` sub-trees rather than as a place to accrete prose. Don't pre-create empty sub-tree dirs (premature), but DO add a brief Knowledge-base maintenance section the first time the file grows past a handful of entries — encode the rhythm so future agents follow it without needing this plugin loaded. The `knowledge-base-cleanup` skill's procedure (step 4) describes what the maintenance section should contain.
