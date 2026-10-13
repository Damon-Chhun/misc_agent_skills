---
name: technical-plan
version: 0.1.0
description: >
  Author an implementation plan that embeds concrete code sketches — not just a
  file list and prose. For each file to change or create, show the intended code
  (type definitions, key component bodies, registry/config diffs, representative
  fixtures) grounded in real APIs you have read, with a brief rationale per file
  and a concrete verification section at the end. Use when writing a plan the
  user will review and approve before implementation, or when the user says
  "write a plan", "add code examples to the plan", "show me what the code will
  look like", "make this plan reviewable". Code sketches are illustrative, not
  final, and must cite the source APIs/line refs they rely on.
---

# technical-plan

A plan's value to a reviewer is how concretely it lets them predict the
resulting code. This skill produces plans that show the code you intend to
write — turning plan review into near-code review, so design issues surface
before implementation, not after.

This plan-authoring style is validated across real planning sessions. Driven
by observation #34 in `~/.claude/skill-observations/log.md`.

This skill writes a plan document only. It does not implement — hand off to
`/implement`, `/tdd`, or direct coding after the plan is approved.

---

## The core rule

**Every file you plan to change or create gets a code sketch, not just a
description.** A bullet that says "add a `CardItem` component that renders the
DataDisplay" is weaker than the actual JSX. Show the code.

Anti-pattern (prose-only):
> - `CardItem.tsx` — new component wrapping `@design-system/data-display`, takes a card
>   prop, renders supertitle/title/amount.

Pattern (code sketch):
````markdown
**`src/components/CardItem.tsx`** (new) — card body, wraps `@design-system/data-display`.

```tsx
import { DataDisplay } from '@design-system/data-display';
import type { Card } from '../types';

export const CardItem = ({ card }: { card: Card }) => (
  <DataDisplay
    supertitle={card.supertitle}
    title={card.title}
    amount={card.amount}
  />
);
```
Rationale: DataDisplay gives token-aligned typography for free; matches Figma
node 101:94992. (API confirmed at `node_modules/@design-system/data-display/.../index.d.ts`.)
````

---

## Phase 1 — Ground the plan in real APIs

Before sketching any code, read the actual APIs the plan depends on — source
files, type definitions, existing components, config/registry shapes. Do not
sketch against assumed signatures.

- Read the real component/function signatures you will call.
- Read at least one existing example of the pattern you are replicating
  (a sibling component, an existing registry entry, a fixture).
- Note the file path (and line refs where useful) for each API you rely on —
  you will cite these in the sketches.

A code sketch built on a guessed API is worse than prose, because it looks
authoritative while being wrong.

> When a handoff doc or prior session summary describes the design or APIs,
> treat it as a hypothesis to verify against the source, not as ground truth.
> (See observations #32–#33.)

---

## Phase 2 — Write the plan with code sketches

Structure:

1. **Goal** — one paragraph: what this builds and the success condition.
2. **Approach** — a short prose overview of the design decision and why,
   including any alternatives rejected.
3. **Files** — one entry per file to change or create. Each entry has:
   - file path + `(new)` / `(edit)` marker
   - a one-line purpose
   - **a code sketch** — types, key component bodies, registry/config diffs,
     representative fixtures (mock JSON, props). Use real, illustrative values.
   - a brief rationale (why this shape) and the source API/line ref it relies on
4. **Verification** — see Phase 3.

Keep prose lean — the code carries the weight. Show diffs for edits to existing
files (`// before` / `// after` or a unified-diff-style block) rather than
re-pasting the whole file.

### Scope in-code comments to the file that owns them

You write sketches with full cross-file context; the resulting comments ship
inside a single file where that context is absent. Before finalizing each
sketch, check every comment destined for file X: it may only reference concepts
X's own domain owns — its own types, its own mechanism, the general contract it
exposes to callers. It must not name a *caller's* specific business logic,
trigger condition, or domain vocabulary that only makes sense one or two files
away.

Concrete tell: a comment that name-drops a value (a specific outcome string, an
enum member, another module's internal state) which is never declared or checked
anywhere in the file being edited is leaking. Fix by stating the general contract
instead:

> Anti-pattern (in `BottomSheetContext.tsx`):
> `// reload the route when the player finishes with a 'doneWithTopic' restart`
>
> Pattern:
> `// reload the underlying route when the caller requests it`

This is separate from the API-grounding discipline above: grounding verifies a
comment is *true*; this check verifies it is *scoped to the right file*. Plan
prose and rationale may freely reference cross-file concepts — the constraint
applies only to comments inside the code that will ship. (Observation #41.)

---

## Phase 3 — Concrete verification section

End every plan with a verification section that states, concretely, how to
confirm the work is correct:

- the exact commands to run (build, lint, test) and the expected result
- what to visually check, and against what (e.g. "compare Storybook output
  side-by-side against Figma node X — confirm spatial layout, not just that it
  renders")
- any fixtures/mocks the verification depends on

"Make it work" is not a verification criterion. "Story renders without errors
AND card layout matches Figma node 101:143510 (240px cards, side-by-side, peek
chevron)" is.

---

## Pre-flight checklist

Before presenting the plan, re-read it and confirm:

1. Every file entry has a code sketch, not just a prose description.
2. Every sketch cites the real API/source it relies on (path, line ref).
3. The plan states up front that code sketches are **illustrative, not final**.
4. The verification section names concrete commands and a fidelity check, not
   "make it work".
5. No file is listed without showing what changes inside it.
6. Every in-code comment references only concepts its own file owns — no
   comment name-drops a caller's business logic, trigger value, or domain
   vocabulary the file itself never declares or checks.

If any check fails, fix the plan before presenting it.
