---
name: discover-documentation-candidates
description: >
  Scans a repo using codebase-memory-mcp and suggests reference documentation
  candidates organized by feature, with related examples, models/schemas,
  architecture patterns, and gotchas grouped under each feature entry point.
  Use when starting to build out .claude/references/ for a new repo, onboarding
  to an unfamiliar codebase, or when you want to know what's worth documenting.
  Trigger phrases: "discover features", "what should I document", "suggest
  reference docs", "scan the codebase", "what's worth documenting here".
---

# Discover Features

Scans the repo and produces a feature-anchored map of reference doc candidates.
Each feature is the anchor; its related examples, models, patterns, and gotchas
hang off it. This preserves the connections between files — which schema backs
which feature, which fixture exercises which flow — rather than producing
disconnected per-category lists.

A "Shared / Cross-cutting" section at the end catches anything that didn't
trace to a specific feature entry point.

## Steps

**Environment fallback (Steps 1–4):** Steps 1–4 below call
`mcp__codebase-memory-mcp__*` tools. If those tools are unavailable in the
current environment (e.g. the skill is invoked from a reduced-tool context
such as an eval subagent, or `ToolSearch` returns no matches for them),
substitute filesystem exploration: `find`/`ls` for structure, `grep -rn`
for code search, `cat` (or `Read`) for content. Output quality is
equivalent for repos under ~600 files.

### Step 1 — Index check

Call `mcp__codebase-memory-mcp__index_status`. If not indexed, run
`mcp__codebase-memory-mcp__index_repository` with the current working directory first.

If `index_status` returns an MCP transport error (e.g. `-32000: Connection
closed`), do not stall — call `index_repository` directly. The indexer is
idempotent and safe to call even if already indexed.

### Step 2 — Doc inventory + architecture scan

**First, inventory existing documentation.** Before any code search, run:

```bash
ls docs/ .claude/references/ 2>/dev/null; ls -1 README.md ARCHITECTURE.md CLAUDE.md 2>/dev/null
```

Note what human-written docs already exist. For any architecture or relationship question that comes up during this skill, **check these docs first** — they express intent and design decisions that code alone does not. Code search fills gaps; docs provide the intentional narrative.

**Then** call `mcp__codebase-memory-mcp__get_architecture` to get the top-level structure. Note:
- Primary entry points
- Source tree layout and layer conventions
- Key frameworks and libraries in use

This gives you the lay of the land before drilling into features.

### Step 3 — Identify feature entry points

**Scope:** Focus on the application's own source code and any existing documentation
(e.g. `CLAUDE.md`, `README.md`, `docs/`, `documents/`). Skip directories that are
AI/agent tooling or IDE configuration — these are not features of the application:
- `.claude/`, `.cursor/`, `.agents/`, `.ai-context/`, `.devassist-plugins*`
- Any Claude Code plugin layer (commands, skills, MCP configs, Stork workflows)
- CI/CD config (`Jenkinsfile*`, `.github/`)
- Test infrastructure config (`jest.config.js`, `playwright.config.js`, etc.)

Use `mcp__codebase-memory-mcp__search_graph` to find feature/capability anchors
in the application source:
- Route handlers and page/screen-level components
- Named user-facing flows
- Capability registrations or feature flag–gated code blocks
- Named exports that represent discrete product features

These become the top-level items in your output. **Cap at 8 features** — aim for
the most meaningful ones, not every file. A feature is something a product person
would name. If there are many candidates, prefer features with the most connections
in the graph (higher fan-in/fan-out = more likely to need documentation).

### Step 4 — Classify each feature's related artifacts

For each feature entry point, use `mcp__codebase-memory-mcp__search_graph` and
`mcp__codebase-memory-mcp__search_code` to find related artifacts by name/path
proximity and import patterns. Reserve `mcp__codebase-memory-mcp__trace_path` for
only the 2–3 most complex or highly-connected features — it is expensive and should
not be called for every feature.

Classify what you find into 5 buckets:

| Bucket | What to look for |
|---|---|
| **Examples** | Test fixtures, demo files, Storybook stories, `__fixtures__`/`__mocks__` dirs, files named `example`, `demo`, `sample` |
| **Models & Schemas** | TypeScript interfaces/types, Zod/Yup schemas, JSON schema files, API response shapes, files in `types/`, `models/`, `schemas/` |
| **Architecture** | System-level design: how this feature is wired into the app at a high level, data flow across layers, cross-service contracts, entry point sequencing, why this layer exists. Think "a new engineer needs a map, not a how-to" |
| **Patterns** | Implementation-level conventions: base classes, HOCs, custom hooks specific to this feature, provider/context trees, event/pubsub wiring — things that tell you *how* to use the feature's building blocks |
| **Gotchas** | See below — these are the highest-value candidates |

#### Recognizing gotchas

Gotchas are things you can't understand from reading the code alone — they
require tribal knowledge of external systems, the host runtime, or historical
context. Look for two classes of signal:

1. **Comment breadcrumbs** — `FIXME`, `TODO`, `HACK`, `XXX`, or inline ticket
   references (e.g. `// PROJ-1234`) clustered around non-obvious code.
   Use `mcp__codebase-memory-mcp__search_code` with these patterns.

2. **Implicit environmental contracts** — references to globals or injected values
   with no local definition. Look for `getGlobal(`, `window.`, `globalThis.`, or
   similar patterns where the value is consumed but never initialized in this repo.
   These represent silent assumptions about what the runtime or host provides.

Run search_graph queries in parallel where possible to keep the scan fast.

Write the output card for each feature as you finish classifying it — don't batch all feature cards and emit them at the end. On large repos, the scan can take several minutes; incremental output means partial results are visible if the session drops.

### Step 5 — Scan package.json for notable dependencies

Read `package.json` from the repo root. For each entry in `dependencies` (not
`devDependencies`), decide whether it warrants a reference doc candidate.

**Skip** well-known open-source packages that don't need explanation:
`react`, `react-dom`, `redux`, `react-redux`, `reselect`, `classnames`,
`lodash`, `axios`, `jquery`, `styled-components`, `typescript`, etc.

**Flag** packages that represent non-obvious contracts an engineer needs to
understand to use correctly — these are prime reference doc candidates:
- Internal/platform namespaces (e.g. `@platform/*`, `@ui-kit/*`, `@core-app/*`, `@internal/*`, `@shell/*`)
- Service client SDKs (e.g. `@service-clients/*`)
- Design system packages where usage conventions aren't obvious (e.g. `@design-system/*`)
- Any package whose name doesn't make its purpose clear from the name alone

For each flagged package, note:
- What it is and why it's non-obvious
- Which files in the repo import it (a quick `search_code` or `search_graph` for
  the package name to find usage callsites)

These become candidates in a **"External Dependencies"** subsection of the
Shared / Cross-cutting section.

### Step 6 — Collect cross-cutting candidates

After tracing all features, note anything meaningful that didn't connect to a
specific feature: shared utility families, global config systems, cross-service
clients, pubsub contracts used repo-wide. These go in a "Shared / Cross-cutting"
section rather than under any one feature.

### Step 7 — Present candidates

Present one card per feature, then the shared section. For each candidate include
the specific signals found (file paths, function names, comment text) so the user
can judge whether it's worth documenting.

Only surface docs where real substance was found. Skip a bucket entirely for a
given feature if nothing meaningful turned up — don't pad with generic entries.

**Output format:**

Before the feature cards, include a metadata header. Get the current branch by
running `git rev-parse --abbrev-ref HEAD` in the repo directory.

```
**Repo:** <repo-name>
**Branch:** <git-branch>
**Scanned:** <date>

---

## [Feature Name]
Entry point: src/features/auth/AuthFlow.tsx

| Category | Proposed file | What it would cover | Signal |
|---|---|---|---|
| Examples | `examples/auth-flow.md` | Auth fixture data and story walkthrough | `src/__fixtures__/auth/`, `AuthFlow.stories.tsx` |
| Models | `models/auth-user.md` | AuthUser type and authSchema shape | `src/types/AuthUser.ts`, `src/schemas/authSchema.ts` |
| Architecture | `architecture/auth-flow.md` | How AuthFlow is wired into the app shell, the props contract with the host, and the redirect lifecycle | `src/features/auth/AuthFlow.tsx` entry wiring, `src/index.ts` mount sequence |
| Patterns | `patterns/auth-hooks.md` | `useAuth` and `useAuthGuard` hook conventions — when to use each | `src/hooks/useAuth.ts`, `src/hooks/useAuthGuard.ts` |
| Gotchas | `gotchas/auth-provider.md` | Shell injects `window.__authProvider` — no local init | `window.__authProvider` consumed in `useAuth.ts`, never defined here |

---

## [Next Feature Name]
...

---

## Shared / Cross-cutting

| Category | Proposed file | What it would cover | Signal |
|---|---|---|---|
| Patterns | `patterns/pubsub-contracts.md` | All repo-wide pubsub events | EventBus usage across 12 files |

### External Dependencies

| Package | Proposed file | What it would cover | Used in |
|---|---|---|---|
| `@platform/session-start-sdk` | `dependencies/session-start-sdk.md` | How to initialize a session, what config it expects | `src/utils/srs.js`, `src/index-unified.js` |
| `@ui-kit/core-utils` | `dependencies/core-utils.md` | HTTP client singleton, utility hooks — what this namespace provides | `src/app-shell/http-client.js` |
```

### Step 8 — Ask the user

After presenting all candidates, ask:
> Which of these should I stub out? Say "all", list specific files by name, or "none" to skip. (The `scaffold-documentation` skill will create the stubs — I'll hand off the confirmed list.)

Do not create stubs in this skill.
