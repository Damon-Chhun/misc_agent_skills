---
name: logical-pull-request
version: 0.2.0
description: >
  Before opening a PR or substantially rewriting its description, check the
  target repo's structural conventions instead of drafting freeform: does it
  have a `.github/PULL_REQUEST_TEMPLATE.md`, what do the branch's commits
  actually say, and does every path referenced in the draft body resolve in
  the repo (not gitignored, not a local-only sibling directory). Use when the
  user says "create a PR", "open a pull request", "update the PR description",
  or "write the PR body". Mirrors `logical-commit`'s detect-conventions-first
  shape, applied to PR descriptions instead of commit messages.
---

# logical-pull-request

PR descriptions have two independent constraints that are easy to conflate:
matching the **repo's structural convention** (a template file, required
checkboxes/headers) and matching the **author's voice** for prose. A draft
that nails voice but ignores the repo's template, or that references a path
true only in the author's local working directory, fails the first reader
who isn't in this conversation.

Driven by observation #39 in `~/.claude/skill-observations/log.md`.

---

## Phase 1 — Detect the repo's PR template

Run the singular-file check and the plural-directory check **separately** —
do not combine them into one `ls` call. A combined glob prints a "no such
file" line for whichever pattern doesn't match, and that noise sitting next
to a real hit is easy to misread as "nothing found" (this happened on this
skill's own first real use — see observation #40).

```bash
test -f .github/PULL_REQUEST_TEMPLATE.md && echo "FOUND: .github/PULL_REQUEST_TEMPLATE.md"
ls .github/PULL_REQUEST_TEMPLATE/*.md 2>/dev/null
```

State the conclusion as an explicit assertion before moving on — do not infer
it from scanning combined output:

```
template found: yes/no — path: <path or "none">
```

If a template exists, the PR body **must** be structured against its
sections/checklist — not freeform prose appended below it. Treat template
headers and checkboxes as fixed; only the prose under each section is yours
to draft.

If no template exists, fall back to this repo's convention if one is
documented (e.g. this repo's CLAUDE.md: 3–6 sentences, what/why/link, no
`## Context / ## Work / ## Test plan` headers unless requested).

---

## Phase 2 — Gather source material from commits, not just conversation

```bash
git log <base>..<branch> --oneline
git log <base>..<branch>
```

Use the actual commit messages as primary source material for the
Description section. Conversation context can supplement, but the commit
history is what a reviewer can independently verify — don't reconstruct a
narrative that the commits don't support.

---

## Phase 3 — Validate every referenced path

Before finalizing, scan the drafted body for file/path references (e.g.
`docs/foo.md`, `wip-plans/...`) and confirm each one resolves in the target
repo:

```bash
git show <branch>:<path> >/dev/null 2>&1 && echo ok || echo "MISSING: <path>"
```

Reject or strip any reference that's gitignored, in a sibling/local-only
directory, or otherwise doesn't exist in the repo the PR lives in. A path
that's true "in the moment" of this conversation but not true in the repo is
meaningless to any other reader.

---

## Phase 4 — Apply voice, not structure

Do **not** treat the skill's own first-pass prose as final. After gathering
the facts/commits/themes (Phase 2), hand the drafted content to the
`personal-writing-assistant` agent with the one-line description
`"PR description"` and let it phrase the result in the user's voice before the
PR is posted or updated. Leave template headers and checkboxes untouched —
voice governs sentences, not section names.

Default to a **scannable, bulleted-by-theme** structure (group related
commits under a short bold theme header, one bullet per change) rather than
dense paragraphs — this is the user's preferred format. A good reference
pattern: themes like `FeatureA sharing`, `ServiceClient singleton`, bug
fixes, hook extraction, test reorg — each with its own bullet group.

---

## Phase 5 — Enforce the 1500-character limit

PR description body text is capped at **1500 characters**, excluding
diagrams, screenshots, and attachments. This keeps descriptions scannable —
a reviewer shouldn't have to read a wall of text to find what changed.

- Count only prose/bullets/headers actually rendered as text. Embedded
  images, diagram markup rendered as an image, and links to
  screenshots/attachments don't count toward the limit.
- Count against the body *after* the voice pass (Phase 4) and *after*
  template headers/checkboxes are in place — those count too, since they're
  part of what the reader sees.
- If the draft exceeds 1500 characters, cut content — don't shrink font or
  rely on collapsible sections as a workaround. Prefer trimming to the
  theme-header/bullet structure from Phase 4; a diagram or screenshot is
  often a better substitute for a paragraph of explanation than more prose.
- State the count explicitly before finalizing: `character count: <n> / 1500`.

---

## Phase 6 — Choose the PR mechanism (prefer an org PR-automation MCP tool on GHE)

Before shelling out to `gh`, check which tool actually opens the PR. On
self-hosted GitHub Enterprise instances, `gh` is often not installed
and ad-hoc PAT auth against the GHE instance fails — a recurring dead end.

If an org-specific PR-automation MCP tool is available in the current session,
prefer it for create/update/get operations against a GHE host — e.g.
`create_pull_request`, `update_pull_request`, `get_pull_request`,
`get_pull_request_diff`, `get_pull_request_files`, `list_pull_requests`.

Only fall back to `gh` (or the GitHub REST API directly) if no such MCP tool
is available in the session.

---

## Phase 7 — Confirm title convention

Match the PR title to the repo's detected conventional-commit style (same
convention detection `logical-commit` Phase 1 already does — reuse it if a
`logical-commit` pass ran earlier in the session).

---

## Pre-flight checklist

Before opening or updating the PR:

1. Checked for `.github/PULL_REQUEST_TEMPLATE.md` (or `PULL_REQUEST_TEMPLATE/*.md`) via separate, isolated commands — stated the yes/no/path conclusion explicitly, not inferred from combined `ls` output. If found, the body follows its structure.
2. Description content is grounded in `git log <base>..<branch>`, not reconstructed purely from conversation.
3. Every file/path referenced in the body resolves in the target repo (checked, not assumed).
4. Voice pass applied only to prose, not to template headers/checkboxes.
5. Body text (excluding diagrams/screenshots/attachments) is ≤1500 characters — count stated explicitly.
6. On a GHE host, checked for and preferred an org-specific PR-automation MCP tool before falling back to `gh`.
7. Title matches the repo's detected commit convention.
