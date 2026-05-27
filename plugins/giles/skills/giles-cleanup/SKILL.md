---
name: giles-cleanup
description: Restructure a repo's CLAUDE.md (or equivalent agent-notes file) when it has accumulated long inline content that should live in docs/ sub-trees. Use when the file grows past ~250 lines, when a section is monotonically accreting (Current state, Patterns, decisions blocks), or when the user asks to "clean up", "restructure", or "organise" their repo's agent notes. Trigger on phrases like "clean up CLAUDE.md", "restructure the docs", "knowledge base is getting too big", "organise the agent notes", "/giles-cleanup". Walks through inventory → propose structure → extract files → rewrite index → verify.
---

# Giles cleanup — knowledge-base restructure

Restructure `CLAUDE.md` (or any agent-notes file: `AGENTS.md`, project-specific equivalents) from monolithic content into an index that points at sub-tree files. The principle: `CLAUDE.md` is auto-loaded into every session and pays its own context cost. Deep content — patterns, milestone history, design discussions, troubleshooting recipes — should live in files that load only when needed.

This skill is for the **one-off restructure** when a repo's knowledge base has grown unwieldy. For day-to-day maintenance after the restructure, the rules live in the maintenance section the restructure adds to `CLAUDE.md` itself. For end-of-session capture of newly-learned knowledge, see the sibling `giles-distill` skill.

## When this skill applies

- `CLAUDE.md` (or equivalent) > ~250 lines.
- A section is monotonically accreting — each new milestone / pattern / decision adds another paragraph.
- The user asks to "clean up", "restructure", or "organise" their agent notes.
- Duplication between `CLAUDE.md` and `docs/*` — the prose is in both places and rotting.

## Procedure

### 1. Read and inventory

Read the full `CLAUDE.md`. For each section, classify:

- **Stable and short** — keep inline (Conventions, Tests overview, Commits format, Memory pointer).
- **Accreting** — needs a sub-tree (Patterns, Current state, anything with growing entries).
- **One-off references** — link out to an existing or new `docs/*.md` file.

Also look at the auto-memory dir (`~/.claude/projects/<repo>/memory/MEMORY.md` and its per-entry files) — that's already the index-plus-subtree pattern, and any restructure should match its shape.

### 2. Propose the structure with the user

Before writing any files, propose:

- Which categories will get sub-trees (e.g. `docs/patterns/`, plus whatever else fits the repo — `docs/history-m<n>.md` if the repo uses numbered milestones, `docs/decisions/` for an ADR-style repo, `docs/<topic>.md` for one-off design discussions).
- Where the boundary sits between active work and archaeology. **This depends on the repo's planning convention** — in-repo milestone docs, sprint cycles tracked externally, feature-branch READMEs, ADRs, or no formal planning at all. Don't impose a milestone shape on a repo that doesn't use milestones; find where in-progress work is already recorded and respect that.
- How many "current state" entries the index keeps, if the repo has a Current state section at all.
- Whether the Patterns (or any other) index is flat or grouped by topic.

Surface trade-offs the user might push back on:

- Full-body inline vs. index-only (cost: discoverability; benefit: smaller per-session context).
- Grouping vs. flat list (cost: maintenance burden when patterns straddle topics; benefit: scannability past ~20 entries).
- How aggressively older entries roll out of Current state (cost: lost just-prior context; benefit: bounded growth).

Get an explicit sign-off before extracting files. **Don't restructure unilaterally** — these decisions about pace and structure are the user's.

### 3. Extract sub-tree files

For each accreting section, create one file per entry under the sub-tree dir:

- Filename = stable kebab-case slug. Title and body go inside the file.
- One pattern/entry per file. Plain markdown — no frontmatter required unless the repo already has a convention.
- Use `[[other-slug]]` (or markdown links) to cross-reference siblings in the same sub-tree.
- Aim for ~10–30 lines per file. If an entry is longer than that, consider whether it's actually two patterns.

Write these in parallel — they're independent.

### 4. Rewrite the index

Replace the accreting section in `CLAUDE.md` with one-line index entries:

```
- [Pattern title](docs/patterns/slug.md) — one-line summary that helps the agent decide whether to open the file.
```

The one-liner is doing real work: it's the only thing an agent sees by default. Make it specific enough to decide relevance. "Pinned error wording is a test contract" is good; "Testing pattern" is not.

For Current state: last 1–3 active-milestone entries (short summaries + link to the history doc) + one-line per completed milestone (pointing at `docs/history-m<n>.md`).

Add a **maintenance section** near the top of `CLAUDE.md` with:

- The **principle**: "CLAUDE.md is an index, not a dump. Each section points at a sub-tree that owns the prose; the agent's job is to maintain that structure, not overflow it. When knowledge genuinely doesn't fit any existing category, surface the gap to the user before creating a new sub-tree — introducing a new category is a structural decision, not a silent action."
- The **active-vs-archaeology distinction**: in-progress status / running notes / per-task implementation details live in whatever working record the repo uses (in-repo plan doc, external tracker, PR descriptions) — not in CLAUDE.md, and not in a pre-created history doc. The working record isn't a dumping ground either: reusable lessons (patterns, cross-cutting decisions) extract into the knowledge-base structure with a pointer back.
- Per-category routing rules — for the rhythm to follow, **tailored to the repo's planning convention**. For a milestone-shaped repo, this typically reads:
  - new pattern → `docs/patterns/<slug>.md` + one-liner in the Patterns index
  - new cross-cutting decision / design discussion → relevant `docs/<topic>.md` (or new one)
  - active milestone task completed → extend the active spec doc inline; update Current state to a one-liner; older lines roll out
  - **phase / sprint / equivalent completed → extract-and-condense pass**: lift any reusable lessons out into the knowledge-base structure, condense the just-finished phase's notes to status blocks (match the repo's existing post-completion shape if there is one)
  - milestone completed → rename the spec doc to `docs/history-m<n>.md` (archaeology, not active spec); collapse Current state to a single `M<n> complete — see ...` line
  
  For other repo shapes, adapt: an ADR-flow repo's rhythm centres on "decision drafted → decision accepted → ADR archived"; a continuous-delivery repo might just be "pattern noticed → extracted to docs/patterns/" with no phase boundaries at all. Match what the repo already does.
- Cleanup-pass triggers (quantitative — e.g. ">250 lines", ">40 index entries", ">600 lines per docs file"). Triggers are signals to surface with the user, not actions to take silently.

### 5. Verify

- Line counts: `CLAUDE.md` well under the trigger; sub-tree files small.
- Link integrity: every index link in `CLAUDE.md` resolves to a real file. Quick check: extract all link targets via grep, compare to file listing.
- Spot-check: open 2–3 sub-tree files; content matches what was inline before, nothing got lost in extraction.
- Skim the diff in the user's IDE so they can see what moved where; flag anything substantive that didn't make the cut.

### 6. Flag follow-ups

- If the repo has a `README.md` that lists docs, mention it may need updating to reference the new sub-tree(s).
- If other docs reference the old inline content ("see CLAUDE.md > Patterns > X"), flag but don't auto-fix — the user owns those references.
- If the user has a global agent-notes file (`~/.claude/CLAUDE.md`) that could benefit from the same principle, ask whether to add a pointer.

## What NOT to do

- **Don't restructure without explicit user sign-off** in step 2. The maintenance section codifies decisions about pace and structure that the user owns; choosing those defaults silently is overstepping.
- **Don't introduce file formats the repo doesn't already use** — no YAML frontmatter on sub-tree files unless the repo already uses it elsewhere. Match existing conventions.
- **Don't reorganise stable sections** (Conventions, Tests overview, Commits format) unless they're also accreting. They earn their inline spot.
- **Don't move content from `CLAUDE.md` to auto-memory** — auto-memory is user-specific machine-local state (personal prefs, external-system pointers), not repo knowledge. Repo knowledge stays checked in.
- **Don't pre-create history docs for active work.** History docs (`docs/history-m<n>.md` or any equivalent) are completion artifacts. While work is in progress, the working record — in-repo plan doc, external tracker, PR descriptions, whatever the repo uses — IS the place for running notes. Creating a history doc for active work just relocates the dumping ground from CLAUDE.md to docs/, without fixing the underlying issue.
- **Don't impose a planning structure the repo doesn't use.** If a repo doesn't organise work in numbered milestones (or sprints, or any in-repo phases), don't invent that structure as part of the cleanup. The active-vs-archaeology rule still applies — but "the working doc" may live entirely outside the repo (Linear, Jira, PR descriptions). Adapt the maintenance rhythm to what the repo already does; ask the user if unclear.
- **Don't relocate status reports without condensing.** If the source CLAUDE.md has long status-report-shaped entries (test counts, verbatim shell output, "out of scope" lists that duplicate the spec doc), moving them verbatim to a new file preserves the noise. Either condense to the curated shape the repo's existing post-completion docs use, fold them back into the active working record inline, or surface to the user that the content is mostly recoverable from `git log` + the spec doc and should just be dropped.
- **Don't delete content during extraction without flagging.** Status-report content that's truly recoverable from `git log` + spec docs can be dropped, but surface the call to the user — don't decide unilaterally. Deletion is its own pass.
