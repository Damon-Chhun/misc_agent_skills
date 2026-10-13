---
name: logical-commit
version: 0.1.0
description: >
  After a multi-file implementation session, group the changed files into
  logical commits that match the repo's commit conventions, then commit them in
  order — instead of one giant commit or arbitrary manual splitting. Detects the
  repo's message style from git log, groups related files (lib + its spec,
  component + spec + storybook, config-only), drafts a conventional message per
  group, and presents the proposed groups for approval before committing. Use
  when the working tree has many changes ready to commit, or when the user says
  "commit these logically", "split these into commits", "structure the commits",
  "group the commits". The approval gate before committing is mandatory.
---

# logical-commit

Commits are documentation. This skill structures a working tree full of changes
into semantically-grouped commits — a clean history that makes PRs easier to
review and `git bisect` easier to run.

Driven by observation #28 in `~/.claude/skill-observations/log.md`.

The agent proposes the grouping; the human approves before anything is
committed. This gate is non-negotiable.

---

## Phase 1 — Learn the repo's commit conventions

Read recent history to detect the message style — do not assume:

```bash
git log --oneline -20
```

Note: prefix style (`feat:`, `fix:`, `docs:` conventional commits? scope in
parens?), ticket reference format (`PROJ-123`, `(#42)`), and any
co-author trailer convention. The drafted messages must match what you observe.

---

## Phase 2 — Inventory the changes

```bash
git status --short
git diff --stat HEAD
```

Get the full list of changed files (staged and unstaged). If something is
already staged, fold it into the grouping rather than treating staging as the
final answer.

---

## Phase 3 — Group into logical units

Group files so each commit is one coherent, reviewable change. Common groupings:

- a lib/util change **+ its spec/test**
- a component **+ its spec + its storybook story**
- a config-only change (lockfiles, tsconfig, CI) on its own
- a docs-only change on its own
- a type/interface change + the first consumer that needs it

Rules of thumb:
- A commit should pass review on its own — no "WIP" half-changes split across
  commits.
- Tests travel with the code they test, in the same commit.
- Keep unrelated concerns in separate commits even if touched in the same
  session.
- Order the commits so the history reads as a logical build sequence
  (types/interfaces before consumers, lib before UI, etc.).

---

## Phase 4 — Draft a message per group

For each group, draft a commit message in the detected convention:
- conventional prefix matching the change type
- scope if the repo uses scopes
- ticket reference if the repo uses them
- a co-author trailer if the repo convention requires one (this repo:
  `Co-Authored-By: Claude <noreply@anthropic.com>` with the active model name
  appended — check the repo's CLAUDE.md)

---

## Phase 5 — Present for approval (mandatory gate)

Show the user the full plan before committing anything:

```
Commit 1: feat: add CardItem component
  - src/components/CardItem.tsx
  - src/components/CardItem.spec.tsx
  - src/components/CardItem.stories.tsx

Commit 2: chore: bump dependency version
  - package.json
  - pnpm-lock.yaml
```

Wait for explicit approval. The user may reassign files, merge or split groups,
or rewrite messages. Apply their changes and re-present if the grouping changed
materially.

---

## Phase 6 — Execute the commits

On approval, stage and commit each group in order:

```bash
git reset            # clear the index, start from a clean slate
git add <group-1-files> && git commit -m "<message-1>"
git add <group-2-files> && git commit -m "<message-2>"
```

Do not push unless the user asks. Report the resulting `git log --oneline`.

---

## Pre-flight checklist

Before presenting groups:

1. Messages match the repo's detected convention (prefix, scope, ticket,
   co-author trailer) — confirmed against `git log`, not assumed.
2. Every changed file is assigned to exactly one group; none are dropped.
3. Tests/specs are grouped with the code they cover.
4. Commit order reads as a logical build sequence.
5. You have NOT committed anything yet — the approval gate comes first.
