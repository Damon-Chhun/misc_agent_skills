---
name: documentation-workflow
description: >
  End-to-end reference-documentation workflow for any repo. Orchestrates the
  discover-documentation-candidates → scaffold-documentation → populate-documentation-categories →
  populate-documentation → review-documentation chain across three modes: init (no docs/ yet),
  add (docs/ exists but a topic is missing), and refresh (drift-check existing docs
  against the codebase). Auto-detects the mode from repo state and the request, then
  confirms before running. Use to set up docs for a repo, document a new feature
  area, or audit existing docs for staleness.
  Trigger phrases: "document this repo", "set up docs", "add docs for <topic>",
  "the docs are missing <topic>", "refresh the docs", "check the docs for drift",
  "are the docs still accurate", "build reference docs".
---

# Doc Workflow

A single entry point that chains the reference-documentation skills end to end.
It does not re-implement them — it sequences them, passes scope between them, and
owns the human checkpoints. The skills it orchestrates:

```
discover-documentation-candidates → scaffold-documentation → populate-documentation-categories → populate-documentation → review-documentation
```

Each underlying skill keeps its own contract; this skill is the harness. When you
need to understand what a step does in detail, read that skill — don't duplicate
its logic here.

## Modes

There are three modes. They share most of the chain; they differ in how they
start and what scope they carry.

| Mode | When it applies | Chain |
|------|-----------------|-------|
| **init** | The repo has no `docs/` directory (or an empty one) | interview → `discover-documentation-candidates` → ✋ confirm candidates → `scaffold-documentation` → `populate-documentation-categories` → ✋ optional tag review → `populate-documentation` → `review-documentation` |
| **add** | `docs/` exists, but the engineer wants docs for a specific topic that isn't covered | scoped `discover-documentation-candidates` → ✋ confirm → `scaffold-documentation` → `populate-documentation-categories` → `populate-documentation` → `review-documentation` |
| **refresh** | `docs/` exists; the engineer wants to know which docs have gone stale | drift scan → ✋ present flagged docs → engineer picks which to redo → `populate-documentation` on those → `review-documentation` |

✋ = human checkpoint. The discover→scaffold checkpoint (confirming which
candidates to stub) is load-bearing — never skip it.

## Step 0 — Detect and confirm the mode

Auto-detect, then confirm — don't make the engineer pick from a menu every time.

1. Check repo state:
   ```bash
   ls docs/ 2>/dev/null | head
   ```
   - No `docs/` or empty → lean **init**
   - `docs/` has content → **add** or **refresh**, disambiguate by the request

2. Parse the request for intent:
   - "set up docs", "document this repo", nothing exists → **init**
   - "add docs for X", "we're missing docs on X", names a specific topic → **add**
   - "refresh", "drift", "still accurate", "audit" → **refresh**

3. State the detected mode in one line and let the engineer redirect:
   > "No `docs/` directory here — running in **init** mode (full first-time docs
   > setup). Say the word if you'd rather scope to a single area instead."

Proceed once confirmed (or after a brief pause if the signal is unambiguous and
auto mode is active).

## Step 1 — Resolve the target repo and branch

Confirm the repo root (where `docs/` lives or will live) and the current branch:

```bash
git rev-parse --abbrev-ref HEAD
```

For **refresh** mode, the engineer may specify a different branch to drift-check
against (e.g. `main` / `master`). Default to the current branch if none is given.
All underlying skills operate relative to this repo root.

---

## init mode

### init.1 — Brief interview

The repo has no docs yet. Before scanning, ask the engineer 2–3 quick questions to
frame the scan (one message, not one-at-a-time — this is framing, not deep design):
- What does this repo do, in a sentence? (anchors feature naming)
- Any areas they already know are worth documenting? (seeds discover-documentation-candidates scope)
- Anything explicitly out of scope? (e.g. generated code, vendored dirs)

Keep it short. The point is to give `discover-documentation-candidates` a head start, not to design
the whole doc set up front.

### init.2 — Run the chain

Run the full chain in order, honoring each skill's own steps and checkpoints:

1. **`discover-documentation-candidates`** — scan the repo, produce the feature-anchored candidate
   list. Pass the interview answers as framing.
2. **✋ Confirm candidates** — present the candidate list; the engineer confirms
   which to stub (all / a named subset / a hand-picked list). This is the
   load-bearing checkpoint.
3. **`scaffold-documentation`** — write stubs for confirmed candidates into `docs/`,
   including the `docs/README.md` index. (scaffold owns README registration.)
4. **`populate-documentation-categories`** — derive the tag vocabulary and write `tags` into each
   stub. Tags before content, by design.
5. **✋ Optional tag review** — if the proposed vocabulary is large or the engineer
   wants a look, pause for tag approval. Otherwise proceed.
6. **`populate-documentation`** — populate every stub. Docs land at
   `pending-human-approval` with `author: <model>`.
7. **`review-documentation`** — walk the engineer through verification (see "Final phase").

---

## add mode

`docs/` already exists; the engineer noticed a specific topic isn't covered (e.g.
"there's no doc for background widgets"). This is init's chain, scoped tight.

### add.1 — Capture the scope

Get the topic from the engineer as a feature area and/or a set of source files
(e.g. "background widgets — `BackgroundWidgets.js`, the renderConditions filter").
Scope is what keeps this fast and cheap — a full-repo `discover-documentation-candidates` here would
be wasteful.

### add.2 — Run the scoped chain

1. **`discover-documentation-candidates`** scoped to that area only — produce candidates just for
   the topic.
2. **✋ Confirm candidates** — usually 1–3 docs; confirm which to stub.
3. **`scaffold-documentation`** — stub them, update `docs/README.md`.
4. **`populate-documentation-categories`** — validate/extend tags for the new stubs against the
   existing vocabulary (don't re-derive the whole repo's tags; reuse what's there).
5. **`populate-documentation`** — populate the new stubs.
6. **`review-documentation`** — verify (see "Final phase").

> **Do NOT skip to `review-documentation` after `populate-documentation`
> completes.** Tags come before the human verification pass: run
> `populate-documentation-categories` (step 4) first so the docs are tagged
> before they're reviewed. The chain order above is fixed — each step's
> successor is named explicitly so the agent doesn't jump ahead to review.

---

## refresh mode

`docs/` exists; the engineer wants to know what's gone stale relative to the code.
This mode **flags only** — it never silently rewrites a doc. The engineer decides
what to redo.

### refresh.1 — Drift scan

For each doc under `docs/`, read its frontmatter and bucket it:

- **Verified docs** (`status: verified-by-human` with a `last-verified` date) →
  candidates for a drift check.
- **Never-verified docs** (`status: pending-human-approval`, no `last-verified`) →
  surface separately as "never verified" — not stale, just never confirmed.
- **Stubs** (`status: stub`) → surface as "never populated."

For each *verified* doc, check whether its cited source has changed since it was
last verified. The entries in `file_patterns:` may be literal paths
(`src/utils/config.js`) or globs (`src/utils/*.js`, `src/components/nav/**`),
so pass each one as a `:(glob)` pathspec. Without `:(glob)`, git's default
pathspec treats a single `*` as matching across `/`, so `src/utils/*.js` would
also match `src/utils/sub/foo.js` and over-flag docs as stale; `:(glob)` makes
`*` stop at directory boundaries:

```bash
git log --since=<last-verified> --oneline <branch> -- \
  <for each entry in the doc's `file_patterns:` field: ':(glob)<entry>'>
```

- Commits exist touching the cited files → **flag as potentially stale**, and keep
  the commit list as evidence.
- No commits → the doc is current; leave it.

If the engineer specified a branch in Step 1, use it; otherwise the current branch.

### refresh.2 — ✋ Present findings

Show three groups, clearly separated:

```
Potentially stale (cited source changed since last-verified):
  patterns/legacy/example-contracts.md   last-verified 2026-05-29 → 4 commits since
     a1b2c3d  fix: rename event constant names
     ...
Never verified (populated, awaiting review):
  patterns/legacy/background-widgets.md
Never populated (stubs):
  models/foo.md
```

The engineer picks which docs to act on. Stale docs they confirm → re-populate.
Never-verified docs → route straight to `review-documentation`. Stubs → route to
`populate-documentation`.

### refresh.3 — Act on the engineer's choices

- For docs chosen to re-populate: run **`populate-documentation`** on that scoped set
  (it reads current source and rewrites; `status` returns to
  `pending-human-approval`, `author` reset to model-only).
- Then run **`review-documentation`** on everything now pending (see "Final phase").

Never auto-update without the engineer confirming the doc. Flag-only is the contract.

---

## Final phase — review-documentation (all modes)

Every mode ends by invoking **`review-documentation`**, which walks the engineer through each
`pending-human-approval` doc and promotes approved ones to `verified-by-human`
(stamping the verifying human into `author` and setting `last-verified`).

This is also why `review-documentation` is a standalone skill: an engineer can verify docs
any time without re-running the whole workflow. Within `documentation-workflow` it's the
closing checkpoint; on its own it's the verification gate.

## Pre-flight check

Before reporting the workflow done, confirm:
- The mode you ran matches what the engineer actually wanted (re-read their request).
- Every checkpoint that the chosen mode requires actually paused for human input —
  especially the discover→scaffold candidate confirmation.
- `docs/README.md` was updated (scaffold owns this; verify it happened).
- The chain ended at `review-documentation`, not at `populate-documentation`.
- `populate-documentation-categories` ran before `review-documentation` (tags
  applied before the verification pass) — it must not be skipped.

If any of these were skipped, surface it rather than quietly reporting success.
