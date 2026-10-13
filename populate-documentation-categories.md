---
name: populate-documentation-categories
description: >
  Scans a repo's stub reference docs and codebase to propose a vocabulary of
  semantic tags, then updates each stub's frontmatter with a `tags` list after
  human approval. Complements discover-documentation-candidates / scaffold-documentation /
  populate-documentation. Use after scaffold-documentation has created stubs, before
  or after populate-documentation runs.
  Trigger phrases: "populate categories", "tag the stubs", "add tags to the docs",
  "categorize the reference docs", "add categories to the stubs", "scaffold categories".
---

# Populate Categories

Derives a repo-specific tag vocabulary from the codebase and existing stubs,
proposes it to the human, then writes an approved `tags` list into every stub's
(or draft's) frontmatter.

The `tags` field is a list of semantic strings — domain concepts, feature areas,
team ownership signals — that complement the structural `category` field already
in each stub. Think of `category` as the document *type* (gotcha, pattern,
architecture…) and `tags` as the document *topics* (auth, routing, pubsub,
shell-contract…).

## Steps

### Step 1 — Discover stubs

Find all files under `docs/` in the target repo that have `status: stub` or
`status: draft` in their frontmatter. Collect:
- The file path
- Its `feature` frontmatter field
- Its `category` frontmatter field
- Its `description` frontmatter field
- Any existing `tags` field (may be absent or empty)

Print the count found. If zero, stop and tell the user — there's nothing to tag.

### Step 2 — Scan for tag signals

Mine the codebase for vocabulary the tags should reflect. Run these searches in
parallel:

**a) Feature names from stubs.** Extract all unique `feature` values from the
discovered stubs. These are primary tag candidates — most stubs should carry
the tag for their own feature.

**b) Directory and module names.** Scan the source tree top-level directories
and key subdirectory names (e.g. `src/features/`, `src/services/`,
`src/hooks/`). Names that recur across files are strong signals.

**c) CLAUDE.md / README.** Read the repo's `CLAUDE.md` and `README.md`. Extract
named systems, layers, concepts, and subsystem labels — these are the terms the
team already uses to describe the codebase.

**d) Comments and annotations.** Search for `@domain`, `@area`, `@team`, or
similar annotation patterns in source files. Also scan for recurring capitalized
concept names in comments (e.g. "AppShell", "NavContext", "CaaS") that appear
in 3+ files.

**e) Package namespaces.** From `package.json`, extract namespaces from
internal package names (e.g. `@platform` → `platform`,
`@design-system` → `design-system`). These often map directly to useful tags.

### Step 3 — Propose a tag vocabulary

From the signals above, derive a proposed tag list. Aim for 8–20 tags — enough
to be useful, few enough to stay consistent across docs.

**Tag naming rules:**
- Lowercase kebab-case (e.g. `auth`, `nav-context`, `shell-contract`, `pubsub`)
- Prefer the term the codebase actually uses over generic alternatives
- One concept per tag — don't merge two ideas into one tag
- Avoid tags so broad they'd apply to every doc (e.g. `frontend`, `react`)
- Include feature-level tags for all features found in Step 2a

Present the proposed vocabulary as a table:

```
## Proposed tag vocabulary

| Tag | Rationale |
|---|---|
| `auth` | Feature name from stubs; `src/features/auth/` directory |
| `shell-contract` | Appears in CLAUDE.md; window.globals consumed in 4 files |
| `pubsub` | EventBus imported in 12 files |
| `design-system` | `@design-system/*` packages in package.json |
| ... | ... |
```

Then ask:
> Approve this vocabulary, or steer me? You can:
> - Say **"approve"** to proceed with all tags as listed
> - **Add tags**: list new ones (e.g. "also add: payments, a11y")
> - **Remove tags**: name the ones to drop
> - **Rename tags**: e.g. "rename shell-contract to appshell"
> - Say **"start over"** to re-propose from scratch with different guidance

Wait for the human's response before continuing.

### Step 4 — Apply approved vocabulary to stubs

Once the vocabulary is approved (with any edits incorporated), update each
stub's frontmatter.

**Assignment rules:**

For each stub, assign tags by matching:
1. Its `feature` value maps directly to a tag of the same name (if that tag
   exists in the vocabulary)
2. Its `description` and `file_patterns` fields contain terms that match or closely
   relate to a tag
3. Its `category` value may suggest tags (e.g. `gotchas` docs often warrant
   a `gotcha` or context-specific tag; `dependencies` docs warrant the
   package's namespace tag)

Assign 1–4 tags per stub. Err toward fewer rather than more — a doc with 5+
tags is probably too broadly tagged.

**Frontmatter update:** Add or replace the `tags` field in each stub's
frontmatter. Place it after the `category` field:

```yaml
---
status: stub
feature: auth
category: gotchas
tags:
  - auth
  - shell-contract
description: ...
```

If a stub already has a `tags` field, merge the approved tags with any
pre-existing values (dedup, keep the union).

Update stubs in batches — don't ask for per-file confirmation.

### Step 5 — Report

After all files are updated, print a summary:

```
Tagged N docs in docs/:

  ✓ gotchas/auth-provider.md          → auth, shell-contract
  ✓ patterns/pubsub-contracts.md      → pubsub
  ✓ dependencies/session-start-sdk.md → platform
  ...

Tag distribution:
  auth          — 4 docs
  shell-contract — 3 docs
  pubsub        — 2 docs
  ...
```

If any stub could not be tagged (no strong signal for any approved tag), list it
as untagged rather than forcing a guess. The user can assign tags manually.

### Step 6 — Generate the docs index

A tag vocabulary that lives only in frontmatter is invisible to an agent that
reads one file at a time — to answer "which docs cover navigation?" it would
have to grep every file. The index closes that gap: it makes the vocabulary
queryable from a single file. Generate (or **regenerate**) `docs/README.md`
after tagging completes.

**Regenerate, do not append.** The tag vocabulary and the doc set both change
between runs. Overwrite `docs/README.md` each time so it always reflects the
current state — never append to a stale version.

The README has three sections:

**1. Tag decision tree.** A table mapping task context → relevant tags → a
suggested starting doc, so an agent can jump from a signal to the right doc
without scanning. One row per common task context:

```markdown
## Find the right doc

| If you're working on… | Tags | Start here |
|---|---|---|
| Authentication / session | `auth`, `shell-contract` | `gotchas/auth-provider.md` |
| Cross-widget messaging | `pubsub` | `patterns/pubsub-contracts.md` |
| Navigation / routing | `nav-context` | `architecture/nav-context.md` |
```

**2. By category.** One table per `category` (architecture, patterns, gotchas,
models, examples, dependencies — only those present). Each row: doc name (as a
relative link), one-line `description`, and its tags.

```markdown
## By category

### Architecture
| Doc | Description | Tags |
|---|---|---|
| [nav-context.md](architecture/nav-context.md) | How NavContext propagates… | `nav-context` |
```

**3. By tag.** One subsection per tag, listing every doc carrying it as a
relative link.

```markdown
## By tag

### `auth`
- [gotchas/auth-provider.md](gotchas/auth-provider.md)
- [patterns/login-flow.md](patterns/login-flow.md)
```

Pull every value from the current frontmatter of the tagged docs — the
`category`, `description`, and `tags` fields. Print a one-line confirmation
(`✓ docs/README.md regenerated — 3 categories, 12 tags, 39 docs`) when done.
