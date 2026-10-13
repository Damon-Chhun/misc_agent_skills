---
name: personal-writing-assistant
description: >
  Write or edit technical documents in the owner's personal voice, trained from sample documents they provide. Supports three modes: write (draft or edit a doc), train (add new writing samples to refine the style guide), and samples (list what's been indexed). Use when someone says "write in my style", "draft a design doc for me", "edit this in my voice", "draft a spike doc", "draft a change doc", "write up the options", "train on this doc", "add this sample", or invokes /personal-writing-assistant. Invoke even when the user gives you a brain dump or bullet points and says "make this a proper doc" — the write mode handles that.
---

# Personal Writing Assistant

Writes and edits technical documents in the owner's personal voice. The style is learned from sample documents stored in `references/` and distilled into `STYLE.md`.

Anyone can install this skill and train it on their own writing — the style guide is not hardcoded to any individual. `STYLE.md` and `references/` are generated locally the first time you run `train` mode on your own documents; they aren't shipped with this skill definition, since the whole point is that the voice is yours, not a sample corpus borrowed from somewhere else.

## Modes

### `write` (delegated to agent)
Draft or edit a document in the trained style. **This mode is implemented as a standalone agent** — when invoked from the main conversation, the orchestrator should dispatch the `personal-writing-assistant` agent with the user's request and any raw material they've provided. The agent loads `STYLE.md`, applies the full style guide, and iterates with the user until the doc feels right.

If running inline (no agent dispatch available), follow the write workflow below.

### `train`
Add new writing samples to improve the style guide. Runs interactively in the main conversation — it writes files and re-derives `STYLE.md`.

### `samples`
List all indexed training documents. Runs inline.

---

## Mode: write (inline fallback)

Use this when the agent cannot be dispatched separately.

### Phase 1 — Intake

If the user hasn't provided context already, ask:
1. **Document type** — change doc, options analysis, investigation/spike, cross-team contract, PR description?
2. **Topic** — what system/feature/bug is this about?
3. **Audience** — teammates only, cross-team, or leadership?
4. **Raw material** — paste a brain dump, existing draft, or bullet points.

Skip questions already answered. If nothing is provided at all, ask for a topic and doc type, then draft.

### Phase 2 — Draft

Load `STYLE.md`. Apply every rule in it — structure, tone, vocabulary, quirks, and the pre-output checklist. Do not editorialize or "improve" on the style; reproduce it faithfully, quirks included.

### Phase 3 — Review

Present the draft. Ask:
- Anything to expand or cut?
- Any facts wrong?
- Does the tone feel right?

Iterate until satisfied.

---

## Mode: train

Use this mode to add new sample documents and re-derive the style guide.

### Phase 1 — Collect the sample

Ask the user to paste the document (or provide a file path / URL). Accept any format: Google Doc text, markdown, Confluence export, raw notes.

If the user provides multiple docs at once, accept them all.

### Phase 2 — Save the sample

1. Choose a short slug for the document (e.g. `api-deprecation-plan`, `incident-retro-notes`). Derive from the title if obvious; otherwise ask.
2. Write the raw text to `references/<slug>.md` with a frontmatter header:

```
---
title: <title>
added: <YYYY-MM-DD>
source: <url or "pasted">
---
```

3. Update `references/INDEX.md` — add one line: `| <slug> | <title> | <YYYY-MM-DD> | [Google Doc](url) / \`local:<path>\` / pasted |`

### Phase 3 — Re-derive the style guide

Read all files in `references/`. Analyze across all samples for:
- Structure patterns (opening, closing, body organization)
- Tone and voice (we/I usage, hedging, confidence markers)
- Technical explanation patterns (before/after, actor-based sections, step sequencing)
- Sentence patterns (length, parentheticals, em/en-dash usage)
- Vocabulary (recurring phrases, domain terms, abbreviations)
- List conventions (numbered vs bulleted, nesting, backreferences)
- Header conventions (format, depth, naming)
- Code block usage (when, how labeled, comment style)
- Options/tradeoffs structure (pros/cons format, recommendation placement)
- Distinctive quirks (consistent errors, signature phrases)

Rewrite `STYLE.md` in full. Preserve any patterns that remain consistent; update sections where new samples reveal new evidence. Include a pre-output checklist at the end.

### Phase 4 — Confirm

Show the user:
- Which samples are now indexed (count + slugs)
- A summary of what changed in `STYLE.md` (new patterns found, anything removed)

Ask: does the updated style feel accurate?

---

## Mode: samples

Read each `references/<slug>.md` file and extract frontmatter fields (`title`, `added`, `source`). Display as a table:

| Slug | Title | Added | Source |
|---|---|---|---|
| ... | ... | ... | [Google Doc](url) / `local:<path>` / pasted |

Format the Source column based on what the frontmatter contains:
- Google Docs URL → `[Google Doc](url)`
- Local file path → `` `local:<path>` ``
- `"pasted"` → `pasted`

Fall back to `references/INDEX.md` if individual files are unavailable.

If no references directory exists yet, say "No training samples indexed yet. Run `/personal-writing-assistant train` to add your first document."

---

## File layout

```
personal-writing-assistant/
├── SKILL.md           # This file — workflow instructions
├── STYLE.md           # Derived style guide (auto-updated by train mode, generated per-user)
└── references/
    ├── INDEX.md       # One-line registry of all samples
    └── <slug>.md      # Raw sample documents
```
