---
name: giles-setup
description: Configure Giles' routing directives — interview the user about any external knowledge stores (a database, a notes app, an MCP-backed store, a file) and write the matching storage/routing instruction into the global or repo CLAUDE.md, so giles-distill and giles-cleanup route knowledge there without anyone editing the skills. Use when the user wants to "set up Giles", "configure routing", "point Giles at my knowledge base / database / notes", or invokes "/giles-setup".
---

# Giles setup — configure routing directives

`giles-distill` and `giles-cleanup` both consult the global (`~/.claude/CLAUDE.md`) and repo `CLAUDE.md` for explicit storage/routing instructions and honour any they find. This skill is the front door for *writing* those instructions: interview the user about where they want knowledge to go, then compose a clear directive in the right `CLAUDE.md`. The skills stay tool-agnostic — they only know to look for a directive; everything specific to a store lives in the directive this skill writes.

The output is plain prose under a `## Knowledge routing` heading, not a config format. distill and cleanup read it the way they read the rest of `CLAUDE.md`.

## When this skill applies

- The user wants Giles to send knowledge to an external store — a database, a notes app, an MCP-backed memory, a shared doc, a file — instead of or alongside the repo knowledge base.
- The user asks to "set up Giles", "configure routing", "point Giles at my <store>", "/giles-setup".
- A directive already exists and the user wants to change it (add a category, flip additive↔instead, retarget the store).

It does NOT apply to the actual capture work — that's `giles-distill`. This skill only writes the routing rule the capture skills obey.

## Procedure

### 1. Read what's already there

Read the global `~/.claude/CLAUDE.md` and the repo `CLAUDE.md` before asking anything. If a `## Knowledge routing` directive (or equivalent storage/routing instruction) already exists, summarise it back to the user and frame the session as *editing* it — don't write a second, parallel directive. Two routing blocks that disagree are worse than none.

### 2. Interview the user

These choices shape long-lived behaviour, so they're genuine forks worth asking about directly rather than guessing. Gather:

- **The store** — what it is and, crucially, **how an agent writes to it**. The directive is useless if the capture skills can't act on it. Pin down the mechanism: an MCP tool (name it), a CLI command, a file or directory path to append to, an API the agent can call, or "the agent drafts it and I paste it in manually." If the write path needs a tool the session doesn't have, say so — the directive can still be written, but flag that the capture skills will only be able to *draft* for that store until the tool is available.
- **Scope** — global (`~/.claude/CLAUDE.md`, applies in every repo) or this repo only (repo `CLAUDE.md`). Cross-repo / user-global knowledge usually wants the global file; a project-specific store wants the repo file.
- **Which knowledge routes there** — everything durable, or a specific branch: cross-repo / user-global knowledge (the kind that would otherwise reach auto-memory or a global-instructions file), repo `docs/` knowledge, patterns only, decisions only. Be concrete; the capture skills match against these categories.
- **Additive or instead** — does matching knowledge *also* go to the named store (additive, the default — keeps the repo self-contained for agents without the store), or go there *instead* of its normal home? Make the trade-off explicit: `instead` for `docs/` content means a teammate without the store sees an index of pointers they can't follow; `instead` for the auto-memory branch is usually safe, since that content was never repo-portable.
- **Index convention** — when content is diverted wholesale, how should the `CLAUDE.md` index entry reference it? Typically the store's record id or URL rather than a local path. (Skip if the directive is purely additive and nothing is diverted.)

Don't over-ask. If the user's opening request already answers some of these ("point Giles at my Obsidian vault at ~/notes, everything, instead of docs/"), confirm the gaps and move on.

### 3. Compose the directive and confirm it

Assemble a `## Knowledge routing` block from the answers and show it to the user before writing. Keep it readable prose the capture skills can follow. Shape:

```markdown
## Knowledge routing

I keep durable knowledge in <store>. Write to it via <mechanism>.

When distilling or cleaning up:
- Route <categories> to <store> <additively | instead of their normal home>.
- <index-entry convention, if anything is diverted wholesale>.
- Keep CLAUDE.md files as the index and the directive holders — they stay local.
```

Fill every `<...>` with the user's actual answers; leave no placeholders. If the write mechanism is a named tool, name it in the directive so the capture skills can act without rediscovering it.

### 4. Write it into the chosen CLAUDE.md

- Write the block into the file the scope answer chose — global or repo. Create the `## Knowledge routing` section if absent; replace it in place if you're editing an existing directive (step 1), never append a duplicate.
- Touch only the routing block. This skill doesn't restructure the rest of the file — that's `giles-cleanup`.
- If the chosen file doesn't exist yet (a fresh repo with no `CLAUDE.md`, or no global one), create it with just the heading and the routing block; don't scaffold an index the repo hasn't earned.

### 5. Verify and hand off

- Read the block back and confirm it names a real store and a concrete write mechanism — no placeholders left.
- Tell the user it's live: the next `giles-distill` or `giles-cleanup` pass will consult it, route matching items per the directive, and report applied directives in its summary.
- If the write mechanism needs a tool the session lacks, restate that the capture skills will draft-only for that store until it's wired up.

## What NOT to do

- **Don't bake a store into the skill.** Everything store-specific lives in the directive you write into `CLAUDE.md`, never in this file. A fresh reader of this skill sees only a generic "interview, then write a routing directive" mechanism.
- **Don't write a directive the capture skills can't act on.** A store with no stated write mechanism produces a rule that can't run. Pin the mechanism down, or flag draft-only explicitly.
- **Don't duplicate an existing directive.** Edit the one that's there. Parallel routing blocks that disagree are a footgun.
- **Don't default to `instead`.** Additive keeps the repo self-contained; reach for `instead` only when the user asks for it, and surface what it costs (pointers a storeless agent can't follow).
- **Don't restructure the rest of `CLAUDE.md`.** This skill writes one routing block. Sprawl elsewhere in the file is a `giles-cleanup` job — suggest it if you notice it, don't fold it in here.
