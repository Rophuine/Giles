---
name: giles-distill
description: At session end (or any reflective pause), distill valuable knowledge learned during the session and route it into the repo's knowledge base per Giles' index-not-dump discipline. Use when the user asks to "capture what we learned", "save what's worth saving", "distill the session", "what should we remember from this", or invokes "/giles-distill". Walks through reflect → filter for value → route → propose to user → apply.
---

# Giles distill — capturing session learnings

The deliberate end-of-session (or reflective pause) ritual: re-read the session with the question "what should outlast this conversation?" front of mind, filter for what's genuinely valuable, and route it into the repo's knowledge base per Giles' index-not-dump discipline.

This skill complements passive accumulation. During normal work the principle in this plugin's `CLAUDE.md` keeps knowledge moving into the right files as it's learned. Distill is for the slower beat — the deliberate review at session end where things that slipped through get caught. For the one-off restructure of a sprawling `CLAUDE.md`, see the sibling `giles-cleanup` skill.

## When this skill applies

- Wrapping up a session and there's been substantive work — debugging, design, exploration.
- The user asks: "what did we learn?", "capture what's worth saving", "distill the session", "/giles-distill".
- At a natural completion boundary (feature shipped, milestone closed, problem solved) where reusable lessons may have surfaced.

It does NOT apply when the session was purely mechanical execution of an already-known plan with no discovery.

## Procedure

### 1. Reflect on the session

Skim the session's history. List candidate learnings broadly — be generous here, the filter step is next. The list below sketches the breadth; it's not exhaustive and the categories aren't crisp. Many learnings sit between them.

- **Problems hit and solved** — bugs, environmental quirks, debugging excursions, failures the next agent would otherwise repeat.
- **Tool / mechanical gotchas** — recurring tooling behaviours that bite anyone using the same tool the same way. Cheap to write down, expensive to rediscover every time.
- **Decisions made** — architectural choices, trade-offs, things explicitly rejected (and why).
- **Patterns noticed** — recurring shapes in the code, conventions discovered while reading.
- **Architecture and context** — system structure, domain facts, historical reasons, "the reason X is the way it is".
- **External systems** — what they are, how this repo touches them, dashboards/queries/ticket conventions, where to look for what.
- **Ways of working with the user** — collaboration preferences they've expressed or validated in this repo.
- **Anything else** that the next agent would have to relearn.

### 2. Filter for value

For each candidate, ask: **is this worth keeping?** "Valuable" is judgment-laden; the heuristics below are signals, not a rubric, and not exhaustive. The strongest items hit several. Most candidates won't.

Signals that **argue for** keeping:

- **Expensive to rediscover** — required multiple turns of grepping, a debugging excursion, reading external docs, or a back-and-forth with the user. If the next agent would have to repeat that work, write it down.
- **Non-obvious from the code as it stands** — a future agent reading the file naively would not infer this. The code shows *what*; the knowledge captures *why*.
- **Important to correctness** — getting this wrong breaks things, ships bugs, or sends the next agent down a wrong path. Silent footguns are exactly what the KB exists for.
- **Cross-cutting or recurring** — applies in multiple places, or will come up again. One-place quirks usually live as a comment in the code; cross-cutting ones earn a doc.
- **Captures a rationale** — the reason behind a non-obvious choice. Especially if the choice would look wrong on first reading.

Signals that **argue against** keeping:

- **Trivially derivable from current code** — file paths, function names, "the X module does Y." `grep` finds these; the KB shouldn't duplicate them.
- **One-off task state** — what was done this session belongs in commit messages / PR descriptions, not the KB.
- **Status reports** — "we got the tests passing", "the bug is fixed". Past-tense narration of the session is not knowledge.
- **Speculation** — things the user *might* want later. Wait until it's load-bearing.
- **Already documented** — check the repo first; extending or correcting an existing entry beats creating a parallel one.

If after filtering the list is empty, **say so plainly and stop**. Not every session produces durable knowledge, and padding the KB is worse than leaving it alone. An empty distill pass is a legitimate outcome.

### 3. Route each kept item

For each surviving item, propose where it goes per the Giles principle:

- **Fits an existing pattern / doc** → extend that file. Index update in `CLAUDE.md` only if the entry doesn't yet exist there.
- **Earns its own page** → new `docs/<topic>.md` or `docs/patterns/<slug>.md` + one-line index entry in `CLAUDE.md`.
- **Truly a new category** → surface the structural call to the user before creating a new sub-tree. Introducing a category is a structural decision, not a silent action.
- **Not repo knowledge at all** — cross-repo personal preferences, machine-local references, anything user-specific that wouldn't help an agent working in this repo for a different user. Flag for the user, don't write to the repo KB. Auto-memory and global instructions have their own routing for this kind of thing.
- **Belongs in the active working record** (in-repo plan doc, PR description, external tracker) — point at it there if it's task-state rather than reusable knowledge.

Inspect the repo's existing structure before proposing. Don't invent new sub-trees that parallel ones that already exist.

### 4. Apply — write directly, check on structural changes

The Giles principle gives you the routing; this skill's job is to help you slow down and reflect, not to introduce a per-decision approval gate. For most kept items, just write.

**Routine writes (just do them):**

- New entry in an existing sub-tree (e.g. another file under `docs/patterns/`).
- Extension to an existing `docs/<topic>.md` file.
- Index entry in `CLAUDE.md` that points at a file you just created (or that updates the one-liner for an existing entry).

**Structural changes (pause and check first):**

- Introducing a new sub-tree / new category (e.g. creating `docs/decisions/` when only `docs/patterns/` existed).
- Splitting an existing file into multiple, merging two, or otherwise rearranging the shape.
- Adding a brand-new top-level section in `CLAUDE.md`.
- Anything that changes the shape of the KB rather than adding to it.

For structural items, surface the proposed shape to the user before writing. Pair it with the routine items so the user can see the whole pass in one go (and can still push back on a routine item if they disagree). Then write everything in one batch — files in parallel (they're independent), index updates in `CLAUDE.md` last so the index always points at files that exist.

When the pass is purely routine, just write and report. A short "wrote these N items" summary at the end so the user can skim the diff is enough.

### 5. Verify

- Each new or extended file is small and self-contained — no session narration leaked in.
- Each index entry in `CLAUDE.md` resolves to a real file with a specific one-liner ("Pinned error wording is a test contract", not "Testing notes").
- Diff is reviewable — the user can skim it and see exactly what landed where.

## What NOT to do

- **Don't make structural changes without explicit sign-off.** New sub-trees, splits, merges, new top-level sections in `CLAUDE.md` — those are structural decisions and belong to the user. Routine additions to existing structure don't need a check; the principle and the routing in step 3 are enough.
- **Don't capture session narration.** "We tried X, then Y, then Z" is a story, not knowledge. Distill to the durable fact: "X is preferred over Y because Z" or "Y fails in case Z".
- **Don't pad the KB to feel productive.** An empty distill pass is a legitimate outcome. The cost of a low-value entry is real — it sits in the index forever and dilutes the signal.
- **Don't introduce a new category silently.** If nothing existing fits, surface the gap before creating a new sub-tree.
- **Don't relocate auto-memory content into the repo KB**, or vice versa, as a side effect. They're different systems for different things. Route each new item to its right home; don't reorganise the other.
- **Don't duplicate existing entries.** If the knowledge is already in the repo, extend or correct the existing file rather than creating a parallel one.
- **Don't treat heuristics as a checklist.** The "valuable" signals are guidance, not a rubric. An item that hits no listed signal but is clearly valuable still belongs; an item that hits two but feels marginal probably doesn't.
