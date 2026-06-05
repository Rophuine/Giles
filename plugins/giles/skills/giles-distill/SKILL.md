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

**First, check for routing directives.** Before applying the built-in destinations below, read the global (`~/.claude/CLAUDE.md`) and repo-level `CLAUDE.md` for any explicit storage or routing instructions — a user or repo may declare where certain keepers should go (e.g. "when distilling, also send cross-cutting keepers to <somewhere>"). Honour any you find as **additional** targets: route the matching items there *as well as* through the normal repo-KB routing below, unless the instruction explicitly says to send them somewhere *instead*. This keeps the skill decoupled from any specific store — the skill only knows to look; the destinations live in `CLAUDE.md`. The directive governs *where* a keeper goes, not *whether* it's kept: items still pass the step-2 filter first.

Then, for each surviving item, propose where it goes per the Giles principle:

- **Fits an existing pattern / doc** → extend that file. Index update in `CLAUDE.md` only if the entry doesn't yet exist there.
- **Earns its own page** → new `docs/<topic>.md` or `docs/patterns/<slug>.md` + one-line index entry in `CLAUDE.md`.
- **Truly a new category** → pick the best home and create the sub-tree per "Steward, not clerk" in the plugin's `CLAUDE.md`. Name the addition in your reply so the user can redirect if they disagree. Escalate only on a genuine fork (multiple reasonable homes with trade-offs, or a decision that locks in the repo's long-term shape).
- **Not repo knowledge at all** — cross-repo personal preferences, machine-local references, anything user-specific that wouldn't help an agent working in this repo for a different user. Flag for the user, don't write to the repo KB. Auto-memory and global instructions have their own routing for this kind of thing — *unless* a routing directive (above) names a store for this cross-repo / user-global knowledge, in which case send it there instead of auto-memory. A directive overrides the default flag/auto-memory handling for the categories it names; this is the one case where the directive reaches beyond the repo-KB destinations.
- **Belongs in the active working record** (in-repo plan doc, PR description, external tracker) — point at it there if it's task-state rather than reusable knowledge.

Inspect the repo's existing structure before proposing. Don't invent new sub-trees that parallel ones that already exist.

### 4. Apply — decide as the steward, report what you did

The Giles principle puts you in the steward role: make routine structural calls and tell the user in your reply. The pass-the-stamp pattern (asking the user to bless each obvious decision) is friction without value — the user sees the diff and can push back. This skill's job is to help you slow down and reflect, not to introduce a per-decision approval gate.

**Just do them and report:**

- New entry in an existing sub-tree (e.g. another file under `docs/patterns/`).
- Extension to an existing `docs/<topic>.md` file.
- Index entry in `CLAUDE.md` that points at a file you just created (or updates the one-liner for an existing entry).
- New sub-tree (`docs/<category>/`) or top-level doc when one obviously fits the content — the kind of call where alternatives would be clearly wrong.
- New top-level section in `CLAUDE.md` when the content genuinely needs its own heading.

**Escalate to a real question only on a genuine fork:**

- Multiple reasonable homes for the same content, with real trade-offs between them.
- A decision that locks in the repo's long-term shape (e.g. "introduce ADRs as a convention, or keep using prose design docs").
- Splits, merges, or rearranging of existing structure — those undo prior decisions, so the user should weigh in.
- Cases where you're genuinely unsure and pretending to decide would just defer the question to a worse moment.

If you catch yourself drafting a question whose options are "the obvious one" plus two clearly-wrong alternatives, that's not a fork — skip the question, decide, and name the addition in your end-of-pass summary.

Write everything in one batch — files in parallel (they're independent), index updates in `CLAUDE.md` last so the index always points at files that exist. End with a short "wrote these N items, including a new `docs/<category>/` sub-tree because X" summary so the user can skim the diff and see any structural shifts at a glance. If a `CLAUDE.md` routing directive applied, name it in the summary too ("routed N cross-cutting items to <target> per repo `CLAUDE.md`") so the user can see where keepers went beyond the repo KB.

### 5. Verify

- Each new or extended file is small and self-contained — no session narration leaked in.
- Each index entry in `CLAUDE.md` resolves to its target — a real local file, or the external store a routing directive diverted the content to — with a specific one-liner ("Pinned error wording is a test contract", not "Testing notes").
- Diff is reviewable — the user can skim it and see exactly what landed where.

## What NOT to do

- **Don't ask for stamps on obvious calls.** The steward decides routine structural questions (new sub-tree, new top-level section, new top-level doc) and reports them in the reply. Sign-off is for genuine forks: splits, merges, decisions that lock in long-term shape, multiple reasonable homes with real trade-offs. Manufactured questions ("which of these one-real-and-two-wrong options?") are friction without value.
- **Don't capture session narration.** "We tried X, then Y, then Z" is a story, not knowledge. Distill to the durable fact: "X is preferred over Y because Z" or "Y fails in case Z".
- **Don't pad the KB to feel productive.** An empty distill pass is a legitimate outcome. The cost of a low-value entry is real — it sits in the index forever and dilutes the signal.
- **Don't introduce a new category silently.** Steward-led doesn't mean unannounced — when you create a new sub-tree or top-level doc, name it explicitly in your reply ("added `docs/decisions/` for ADR-style entries because…") so the user can see and redirect if they disagree. Silent additions accumulate into structure no one signed up for.
- **Don't relocate auto-memory content into the repo KB**, or vice versa, as a side effect. They're different systems for different things. Route each new item to its right home; don't reorganise the other. (Exception: an explicit `CLAUDE.md` routing directive may name an external store for user-global content that would otherwise reach auto-memory — honour it for the categories it names. That's directed routing, not a side effect.)
- **Don't duplicate existing entries.** If the knowledge is already in the repo, extend or correct the existing file rather than creating a parallel one.
- **Don't treat heuristics as a checklist.** The "valuable" signals are guidance, not a rubric. An item that hits no listed signal but is clearly valuable still belongs; an item that hits two but feels marginal probably doesn't.
