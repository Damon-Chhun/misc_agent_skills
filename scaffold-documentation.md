---
name: scaffold-documentation
description: >
  Creates stub reference docs in a repo's docs/ directory from a confirmed
  candidate list produced by discover-documentation-candidates. Writes the directory structure
  and boilerplate stubs — does not populate content. A follow-up skill will
  examine stubs and source code to populate them with the human.
  Trigger phrases: "scaffold the stubs", "create the reference docs", "stub out
  the candidates", "set up docs/", "create the docs structure".
---

# Scaffold References

Takes the confirmed candidate list from a `discover-documentation-candidates` run and writes
stub files into `docs/` in the target repo. Each stub gets frontmatter,
a generated description, and a standard section skeleton — enough for a
follow-up skill to orient itself and populate the doc with a human.

This skill does not generate content. It scaffolds structure.

## Directory layout

Stubs are written under `docs/` in the target repo using this structure:

```
docs/
  examples/
  architecture/
  patterns/
  models/
  features/
  gotchas/
  dependencies/
```

Create subdirectories only as needed — don't create empty directories for
categories that have no confirmed candidates.

## Steps

### Step 1 — Confirm the candidate list

The user should have a list of confirmed candidates from `discover-documentation-candidates`
(either "all", a specific named subset, or a list of proposed file paths).
If it's not clear which candidates are confirmed, ask before proceeding.

Each candidate has:
- A proposed file path (e.g. `gotchas/window-appshell-globals.md`)
- A category (Examples, Architecture, Patterns, Models, Gotchas, Dependencies)
- A feature name it belongs to (or "Shared / Cross-cutting")
- A "What it would cover" description
- A Signal (specific file paths, function names, comment text from the scan)

### Step 2 — Resolve the target repo root

The target repo is wherever `discover-documentation-candidates` was run. Confirm the path
before writing anything. All stubs are written relative to `<repo-root>/docs/`.

Get the current branch by running `git rev-parse --abbrev-ref HEAD` in the
repo root — you'll need this for the stub frontmatter.

### Step 3 — Write stubs

For each confirmed candidate, create the stub file. Create parent directories
as needed.

**Stub template:**

```markdown
---
status: stub
feature: <feature name, or "Shared / Cross-cutting">
category: <examples | architecture | patterns | models | gotchas | dependencies>
description: <one short paragraph — infer from the "What it would cover" field
  and the Signal evidence. Write what this doc will explain once populated,
  not what the stub currently contains.>
file_patterns: <comma-separated list of the most relevant file paths from
  Signal — keep this tight. Prefer literal paths (`src/utils/config.js`) for
  the handful of files the doc actually cites. Use a glob (`src/utils/*.js`,
  `src/components/nav/**`) only when the doc genuinely covers a whole directory.
  Broad globs over-match: they make the populate step read far more files than
  the doc needs and make the refresh drift-scan flag unrelated changes as stale.>
branch: <current git branch>
last-updated: <today's date, YYYY-MM-DD>
last-verified: N/A
---

# <Title derived from the proposed filename>

> Stub — not yet populated. Source signals: <echo the Signal field from the
> candidate — specific file paths, function names, comment text that justified
> this candidate.>

## Overview

<!-- fill in -->

## Body

<!-- fill in — add subsections as needed for this topic -->

## See Also

<!-- links to related docs in this repo -->

## References

<!-- external links, Confluence pages, Jira tickets, etc. -->

## Key Files / Folders

<!-- file paths most relevant to this topic -->
```

**Frontmatter lifecycle.** A stub starts with `status: stub`, `last-verified: N/A`, and no `author` field. `author` is added downstream — do not write it at the stub stage:
- `populate-documentation` flips `status` to `pending-human-approval` and adds
  `author: <model name>`
- `review-documentation` flips `status` to `verified-by-human`, appends the verifying
  human to `author`, and stamps `last-verified: <date>`

`last-verified` is what `documentation-workflow` refresh mode compares against `git log`
to detect drift — it starts as `N/A` and is only stamped with a real date by `review-documentation` after a human approves.

**Writing the description:** Use the candidate's "What it would cover" field
as the seed. Expand it into one short paragraph that explains what an engineer
will learn from this doc once it's populated. Be specific — reference the
actual systems, hooks, or contracts involved. Do not write "this doc covers X"
— write what X is.

**Writing the title:** Derive from the proposed filename. Convert kebab-case
to title case (e.g. `window-appshell-globals.md` → `window.AppShell Globals`,
`sandbox-bridge-middleware.md` → `Sandbox Bridge Middleware`).

Write stubs in batches — don't ask for confirmation between each file.

### Step 4 — Report what was created

After all stubs are written, print a summary:

```
Created N stubs in <repo-root>/docs/:

  gotchas/window-appshell-globals.md
  gotchas/sandbox-messenger-vs-sandbox-bridge.md
  patterns/sandbox-bridge-middleware.md
  ...

Next: run the populate-documentation skill to work through each stub with source
code and fill in the content.
```

Do not open or display the stub contents — just list what was created.
