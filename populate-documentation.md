---
name: populate-documentation
description: >
  Populates stub reference docs in a repo's docs/ directory by reading source
  code and writing full content using the documentation-writer and
  personal-writing-assistant skills. Works through all stubs in docs/ by default,
  or a scoped subset if specified. Processes stubs sequentially to keep token
  usage low. Use after scaffold-documentation has created the stub files.
  Trigger phrases: "populate the stubs", "fill in the reference docs",
  "write the docs", "populate references", "fill out the docs".
---

# Populate References

Reads each stub in `docs/`, loads the referenced source files, and writes
full content using the `documentation-writer` skill (Diátaxis-guided) followed
by `personal-writing-assistant` polish. Stubs are processed one at a time to
keep token usage manageable.

The human checkpoint comes after all stubs are done — review the populated
docs and request revisions if needed.

## Steps

### Step 1 — Discover stubs

Find all files under `docs/` in the target repo with `status: stub` in their
frontmatter. If the user scoped the run to a specific category or file list,
filter to that subset. Otherwise work through all stubs.

Print the list of stubs found and confirm with the user before writing begins —
this is the only checkpoint.

### Step 1b — Infer corpus conventions

Before writing any content, read one already-populated doc in `docs/` to learn
the conventions the corpus actually uses. Prefer a doc with
`status: verified-by-human`, then `pending-human-approval`; fall back to a
sibling stub. From it, confirm the exact frontmatter schema in play —
including any fields beyond the scaffold default (`feature`, `category`,
`description`, `file_patterns`, `branch`, `last-updated`), e.g. a multi-line
`description`. Match that schema when writing in Step 2; do not write a minimal
subset. If no populated doc exists yet, use the scaffold default schema
documented in `scaffold-documentation`.

The correct field name for referenced file paths is `file_patterns:`, not
`source:`. If no existing populated doc is found to infer the schema from,
default to `file_patterns:`. Do not introduce a `source:` field — it is a
known drift trap that requires a bulk rename to fix.

Keep `file_patterns` tight. If you refine the field while populating, prefer
literal paths for the files the doc actually cites; reach for a glob only when
the doc covers a whole directory. A broad glob (`src/utils/*.js`) makes this
step read far more files than the doc needs and makes the refresh drift-scan
flag unrelated changes as stale.

This step exists because writing into an existing corpus from the stub alone
produces frontmatter drift that needs a second correction pass. If this skill
is ever bypassed in favor of a direct agent prompt, carry this read-one-first
step into that prompt.

### Step 2 — Populate each stub sequentially

Process stubs **one at a time** — read source, write the doc, move to the next.
Do NOT dispatch parallel agents to populate multiple stubs at once unless the
user explicitly asks for parallel execution. Parallel population is a token
multiplier that buys little throughput here (each stub reads source files
sequentially anyway) while multiplying context-window usage. Default to
sequential; let the user opt into parallel when speed matters more than cost.
The same applies to any research phase that precedes population.

Work through the stubs one at a time. For each stub:

**a) Read the stub.** Extract from its frontmatter:
- `feature`, `category`, `description`, `file_patterns`

The `> Stub` callout line contains the specific signals (file paths, function
names, comment text) that justified this doc — use these as your starting point.

**b) Read the source files.** Read each file listed in the `file_patterns` field.
Focus on the functions, types, patterns, or comments called out in the `> Stub`
signal line. You do not need to read entire files — target the relevant sections.

**b.1) Verify identifiers from the prompt/stub.** Before quoting any header,
field, constant, or function name that came from the prompt or the `> Stub`
signal line, grep for it in the repo. Zero hits means the prompt name is
approximate — find the real name in source and use that. The prompt is a
description, not a specification.

**b.2) Sweep siblings of the specified source files.** After reading the
files listed in `file_patterns`, list siblings in each source directory (`ls`
the dir, look at barrel `index.*` files, scan any `utils/` subfolder). Decide
whether utility files, barrel exports, factory functions, or sibling
implementations add load-bearing mechanism that the anchor files reference
but do not contain. For architecture and dependencies docs especially, the
`file_patterns` field anchors the reading — it is not the complete reading list.

**b.3) Label provenance for external libraries.** When documenting an
external dependency whose type definitions are not in-repo, derive the API
contract from production callers (grep imports across the repo, read the
3-5 heaviest callers, observe destructured fields and call shapes). In the
populated doc, label the derived section with a note like
`Field list observed in <repo-name>; not from the library's type
declarations.` This prevents overstating coverage.

**c) Determine the Diátaxis document type** from the stub's category:

| Category | Diátaxis type | Rationale |
|---|---|---|
| gotchas | Explanation | Understanding-oriented — why this surprises people |
| architecture | Explanation | Understanding-oriented — why this layer exists and how it fits |
| patterns | How-to Guide | Problem-oriented — how to use this building block correctly |
| models | Reference | Information-oriented — what this type/schema contains |
| examples | Tutorial | Learning-oriented — walkthrough to a concrete outcome |
| dependencies | Reference | Information-oriented — what this package provides and how to use it |

**d) Invoke `documentation-writer`.** Pre-answer its clarifying questions so it
does not pause for input:
- Document type: from table above
- Target audience: Senior frontend engineers onboarding to this repo
- User's goal: Understand what the stub description covers without needing to
  read source code or ask someone
- Scope: Cover only what the `file_patterns` files and signals justify — don't expand

When `documentation-writer` invokes `personal-writing-assistant`, let it run.

**e) Write the populated doc** back to the same file path, replacing the stub.
Update frontmatter:
- `status: pending-human-approval` (was: `stub`) — the doc is agent-authored
  and ready for human review, not further agent editing. Naming the gating
  actor lets a follow-up review skill find docs awaiting approval.
- `author: <model name>` — the model that populated the doc (e.g.
  `Claude Sonnet 4.6`). Leave the human half unset; the `review-documentation` skill
  appends the verifying human's name on approval (` + <human name>`).
- `last-updated: <today's date, YYYY-MM-DD>`

Do not set `last-verified` — that field is owned by `review-documentation` and stamped
only when a human approves the doc. Its absence is the signal that a doc has
never been human-verified.

Keep the 5 standard sections but replace all `<!-- fill in -->` placeholders
with real content. The Body section may have as many subsections as the topic
warrants. Do not leave any `<!-- fill in -->` placeholders.

**f) Print a one-line status** (`✓ gotchas/window-appshell-globals.md`) and
move to the next stub.

### Step 3 — Verify and report

After all stubs are done, verify each populated file:
- `status` is `pending-human-approval` (not `stub`)
- `author` is set to the model name (human half not yet appended)
- No `<!-- fill in -->` placeholders remain
- All 5 standard sections are present with real content

Report any stubs that failed or still have placeholders.

### Step 4 — Summary and hand off

Print a summary table:

```
Populated N docs in docs/:

  ✓ gotchas/window-appshell-globals.md
  ✓ dependencies/session-start-sdk.md
  ✗ patterns/appshell-subscribe-unsubscribe.md — placeholder remaining in Body
  ...

Review the docs and request revisions for any that need rework.
```

Do not auto-retry failed stubs — surface them for the user to decide.
