---
name: review-documentation
description: >
  Walks an engineer through reference docs awaiting verification and transitions
  approved ones from status `pending-human-approval` to `verified-by-human`,
  stamping the verifying human and a `last-verified` date. Use after
  populate-documentation has written content, or any time docs are pending approval.
  The final phase of the documentation-workflow pipeline; also runnable standalone.
  Trigger phrases: "review the docs", "verify the docs", "approve the reference
  docs", "review pending docs", "mark docs verified".
---

# Review Docs

Populated reference docs land at `status: pending-human-approval` — agent-authored,
awaiting a human's eyes. This skill is the human checkpoint that promotes them to
`verified-by-human`. Verification is what makes a doc trustworthy to readers and
to `documentation-workflow` refresh mode (which uses `last-verified` to detect drift).

Run it standalone after `populate-documentation`, or let `documentation-workflow` invoke it as
its final phase.

## What verification changes

On a human's approval of a doc, three frontmatter fields are updated:

- `status: verified-by-human` (was `pending-human-approval`)
- `author: <model> + <human name>` — the model half is already present from
  `populate-documentation`; append the verifying human (e.g.
  `Claude Sonnet 4.6` → `Claude Sonnet 4.6 + Jordan Lee`)
- `last-verified: <today's date, YYYY-MM-DD>` — added (this is the anchor refresh
  mode compares against `git log`)

Nothing else in the doc is touched on a plain approval.

## Steps

### Step 1 — Find docs pending approval

Find all files under `docs/` in the target repo with
`status: pending-human-approval` in their frontmatter:

```bash
grep -rl "status: pending-human-approval" docs/
```

If the user scoped the run to a specific category, file list, or feature area,
filter to that subset. Otherwise gather all pending docs.

If none are found, report that and stop — there is nothing to verify.

### Step 2 — Get the verifying human's name

Ask who is verifying (or default to the repo's known author if one is configured
in CLAUDE.md / memory). This name is appended to `author` on each approval. Ask
once, up front — not per doc.

### Step 3 — Walk each pending doc

Process one doc at a time. For each:

**a) Show the doc.** Print its path, then its full content (or, for long docs, the
Overview + Body sections and a note that the rest is available). The human is
verifying that the content is accurate against the codebase — give them enough to
judge.

**b) Ask for a decision:**
- **approve** — content is accurate, promote to `verified-by-human`
- **revise** — content needs rework; capture a one-line note on what's wrong
- **skip** — leave as `pending-human-approval`, move on

**c) Act on the decision:**

- **approve:** Update the three frontmatter fields (Step "What verification
  changes"). Print `✓ verified <path>`.
- **revise:** Leave `status` as `pending-human-approval`. Record the human's note
  — either as a `> Review note: <note>` line under the title, or surfaced in the
  final summary for the engineer to act on with `populate-documentation`. Do not
  rewrite the content here; this skill verifies, it does not author. Print
  `↻ revision requested <path>`.
- **skip:** No change. Print `– skipped <path>`.

Do not batch — one decision per doc. The point of this skill is the human actually
reading each doc.

### Step 4 — Summary

Print a summary table:

```
Reviewed N docs:

  ✓ verified            patterns/legacy/example-contracts.md
  ✓ verified            architecture/legacy/example-bridge.md
  ↻ revision requested  patterns/legacy/background-widgets.md — "renderConditions list is stale"
  – skipped             patterns/legacy/dynamic-widget-renderer.md

Verified docs are now status: verified-by-human, author appended, last-verified stamped.
Docs needing revision: re-run populate-documentation on them, then review again.
```

For any revision-requested docs, point the engineer at `populate-documentation` to
rewrite, then back here to verify.

## Pre-flight check

Before finishing, re-read the docs you marked approved and confirm each one has all
three fields updated correctly — `status: verified-by-human`, the human appended to
`author`, and a `last-verified` date. A doc that got `status` flipped but kept a
model-only `author` or no `last-verified` will read as verified but break refresh
mode's drift comparison. Fix any that are inconsistent before reporting done.
