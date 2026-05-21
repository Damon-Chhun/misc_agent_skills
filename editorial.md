---
name: editorial
description: >
  Generates a polished, self-contained HTML document page in the "warm parchment editorial"
  aesthetic — Lora/Inter/JetBrains Mono fonts, terra-cotta accent (#8b3a1a), warm off-white
  background (#f5f0eb). Use for workflows, plans, guides, implementation docs, onboarding pages,
  and any technical content that needs to look good as a standalone HTML file.
  Trigger phrases: "editorial", "create an editorial page", "make an HTML doc", "write this up as a page",
  "generate a workflow page", "make a visual guide", "create a plan page", "doc this as HTML",
  "write a visual plan", "make an onboarding page", "create a reference page".
  Anti-trigger: use the "diagram" skill instead when the primary output is a flow/sequence/flowchart diagram.
version: 1.0.0
---

# editorial

Generates a polished, self-contained HTML documentation page. Warm editorial aesthetic,
structured sections, code blocks, file trees, step timelines, callouts — all from a prose
description of the content.

---

## Design system

### Fonts
Load from Google Fonts (the only permitted CDN call):
```
https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;0,700;1,400&family=JetBrains+Mono:wght@400;500;600&family=Inter:wght@400;500;600;700&display=swap
```

- **Lora** — body text, `h1`. Serif, editorial weight.
- **Inter** — section headers `h2`, labels, captions, metadata, callouts, table headers, step bodies.
- **JetBrains Mono** — all `code`, code blocks, file trees, monospace labels.

### Color tokens
```css
:root {
  --bg: #f5f0eb;          /* warm parchment */
  --surface: #edeae4;     /* slightly darker parchment — cards, file trees */
  --border: #d4cfc8;      /* warm grey rule */
  --text: #1a1714;        /* near-black body */
  --muted: #6b6560;       /* secondary text */
  --accent: #8b3a1a;      /* terra cotta — headings, code, highlights */
  --accent-light: #c4622a;/* lighter rust — hover tones, alt accent */
  --code-bg: #111;        /* dark code block background */
  --code-text: #e2e8f0;   /* light code text */
  --code-key: #e2966a;    /* warm orange — keys, property names */
  --code-val: #a3c4a3;    /* muted green — values, strings */
}
```

### Layout
```css
body {
  background: var(--bg);
  color: var(--text);
  font-family: 'Lora', Georgia, serif;
  font-size: 17px;
  line-height: 1.75;
}
.page {
  max-width: 1100px;
  margin: 0 auto;
  padding: 0 48px 120px;
}
```

---

## Components

### Hero
```html
<div class="hero">
  <div class="hero-eyebrow">TEAM · TOPIC · Date</div>
  <h1>Page title here.</h1>
  <p class="hero-sub">One or two sentences of context.</p>
</div>
<hr class="section-rule" />
```
CSS:
```css
.hero { padding: 72px 0 52px; border-bottom: 1px solid var(--border); }
.hero-eyebrow { font-family:'Inter',sans-serif; font-size:11px; font-weight:700; letter-spacing:0.12em; text-transform:uppercase; color:var(--accent); margin-bottom:20px; }
.hero h1 { font-family:'Lora',Georgia,serif; font-size:clamp(34px,5vw,52px); font-weight:700; line-height:1.1; letter-spacing:-0.01em; margin-bottom:24px; }
.hero-sub { font-family:'Inter',sans-serif; font-size:15px; color:var(--muted); max-width:680px; line-height:1.6; }
hr.section-rule { border:none; border-top:1px solid var(--border); margin:0; }
```

### Section
```html
<div class="section">
  <div class="section-header">
    <span class="section-num">01</span>
    <h2>Section Title</h2>
  </div>
  <p>Body text...</p>
</div>
<hr class="section-rule" />
```
CSS:
```css
.section { padding: 56px 0 0; }
.section-header { display:flex; align-items:baseline; gap:20px; margin-bottom:28px; padding-bottom:12px; border-bottom:1px solid var(--border); }
.section-num { font-family:'Inter',sans-serif; font-size:11px; font-weight:700; letter-spacing:0.1em; color:var(--accent); white-space:nowrap; }
.section h2 { font-family:'Inter',sans-serif; font-size:13px; font-weight:700; letter-spacing:0.08em; text-transform:uppercase; }
.section p { margin-bottom:20px; }
.section p:last-child { margin-bottom:0; }
```

### Inline code
```css
code { font-family:'JetBrains Mono',monospace; font-size:0.82em; background:var(--surface); border:1px solid var(--border); border-radius:3px; padding:1px 5px; color:var(--accent); }
```

### Dark code block
```html
<div class="code-block">
  <pre><span class="code-comment"># comment</span>
<span class="code-key">key</span>: <span class="code-val">value</span></pre>
</div>
```
CSS:
```css
.code-block { background:var(--code-bg); border-radius:6px; padding:24px 28px; margin:24px 0; overflow-x:auto; }
.code-block pre { font-family:'JetBrains Mono',monospace; font-size:13px; line-height:1.8; color:var(--code-text); white-space:pre; }
.code-key { color:var(--code-key); }
.code-val { color:var(--code-val); }
.code-comment { color:#555; }
```

### File tree (light surface)
```html
<div class="file-tree">
<pre><span class="ft-dir">root/</span>
├── <span class="ft-dir">subdir/</span>       <span class="ft-comment"># comment</span>
│   └── <span class="ft-file">file.md</span>
└── <span class="ft-file">other.md</span></pre>
</div>
```
CSS:
```css
.file-tree { background:var(--surface); border:1px solid var(--border); border-radius:6px; padding:20px 24px; margin:20px 0; }
.file-tree pre { font-family:'JetBrains Mono',monospace; font-size:13px; line-height:1.9; }
.ft-dir { color:var(--accent); }
.ft-comment { color:var(--muted); }
.ft-file { color:var(--text); }
```
Additional span classes as needed: `.ft-personal` (`color:var(--accent-light)`), `.ft-gitignored` (`color:#999; font-style:italic`).

### Step timeline
```html
<div class="steps">
  <div class="step">
    <div class="step-num">1.</div>
    <div class="step-body">
      <h4>Step title</h4>
      <p>Description of the step.</p>
    </div>
  </div>
</div>
```
CSS:
```css
.steps { margin:24px 0; }
.step { display:flex; gap:24px; padding:20px 0; border-bottom:1px solid var(--border); }
.step:last-child { border-bottom:none; }
.step-num { font-family:'JetBrains Mono',monospace; font-size:13px; font-weight:600; color:var(--accent); white-space:nowrap; padding-top:2px; min-width:32px; }
.step-body h4 { font-family:'Inter',sans-serif; font-size:15px; font-weight:700; margin-bottom:6px; }
.step-body p { font-family:'Inter',sans-serif; font-size:14px; color:var(--muted); line-height:1.6; margin-bottom:10px; }
.step-body p:last-child { margin-bottom:0; }
```

### Callout (warm highlight)
```html
<div class="callout">
  <strong>Key point:</strong> explanation here.
</div>
```
```html
<!-- Info variant (muted) -->
<div class="callout info">
  <strong>Note:</strong> secondary info here.
</div>
```
CSS:
```css
.callout { background:#ede4d8; border-radius:6px; padding:22px 26px; margin:24px 0; font-size:16px; line-height:1.7; }
.callout strong { color:var(--accent); }
.callout.info { background:var(--surface); border:1px solid var(--border); font-size:14px; font-family:'Inter',sans-serif; color:var(--muted); }
.callout.info strong { color:var(--text); }
```

### Two-column card grid
```html
<div class="two-col">
  <div class="card">
    <div class="card-name">name/path</div>
    <div class="card-desc">Description of this item.</div>
  </div>
  <div class="card dashed">  <!-- dashed = optional/WIP -->
    <div class="card-name">optional-item</div>
    <div class="card-desc">Description.</div>
  </div>
</div>
```
CSS:
```css
.two-col { display:grid; grid-template-columns:1fr 1fr; gap:12px; margin:20px 0; }
.card { background:var(--surface); border:1px solid var(--border); border-radius:6px; padding:18px 20px; }
.card .card-name { font-family:'JetBrains Mono',monospace; font-size:13px; color:var(--accent); font-weight:600; margin-bottom:6px; }
.card .card-desc { font-family:'Inter',sans-serif; font-size:13px; color:var(--muted); line-height:1.5; }
.card.dashed { border-style:dashed; }
.card.dashed .card-name { color:var(--muted); }
```

### Flow pipeline (horizontal nodes + arrows)
```html
<div class="flow">
  <div class="flow-node accent">
    Node Label
    <sub>subtitle</sub>
  </div>
  <div class="flow-arrow">→</div>
  <div class="flow-node">
    Next Node
    <sub>subtitle</sub>
  </div>
</div>
```
CSS:
```css
.flow { display:flex; align-items:center; gap:0; margin:28px 0; flex-wrap:wrap; }
.flow-node { background:var(--surface); border:1px solid var(--border); border-radius:6px; padding:12px 16px; font-family:'Inter',sans-serif; font-size:13px; font-weight:600; color:var(--text); text-align:center; min-width:130px; }
.flow-node.accent { background:#ede4d8; border-color:var(--accent); color:var(--accent); }
.flow-node sub { display:block; font-size:11px; font-weight:400; color:var(--muted); margin-top:2px; font-family:'JetBrains Mono',monospace; }
.flow-arrow { font-size:18px; color:var(--muted); padding:0 10px; }
```

### Table
```html
<table class="data-table">
  <thead>
    <tr><th>Column A</th><th>Column B</th><th>Notes</th></tr>
  </thead>
  <tbody>
    <tr>
      <td class="mono">path/or/code</td>
      <td class="yes">Value</td>
      <td>Note text</td>
    </tr>
  </tbody>
</table>
```
CSS:
```css
.data-table { width:100%; border-collapse:collapse; margin:20px 0; font-size:14px; }
.data-table th { font-family:'Inter',sans-serif; font-size:11px; font-weight:700; letter-spacing:0.08em; text-transform:uppercase; color:var(--muted); text-align:left; padding:8px 14px; border-bottom:1px solid var(--border); }
.data-table td { padding:12px 14px; border-bottom:1px solid var(--border); vertical-align:top; font-family:'Inter',sans-serif; color:var(--muted); }
.data-table tr:last-child td { border-bottom:none; }
.data-table td.mono { font-family:'JetBrains Mono',monospace; font-size:12.5px; color:var(--accent); white-space:nowrap; }
.data-table td.yes { color:var(--text); font-weight:600; }
.data-table td.no { color:#bbb; }
```

### Footer
```html
<footer>
  TEAM · repo · filename · updated Month YYYY
</footer>
```
CSS:
```css
footer { margin-top:80px; padding-top:20px; border-top:1px solid var(--border); font-family:'Inter',sans-serif; font-size:12px; color:var(--muted); text-align:center; }
```

---

## Boilerplate shell

Every generated page starts with this shell. Inline all CSS — no external stylesheet.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{{title}}</title>
  <link href="https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;0,700;1,400&family=JetBrains+Mono:wght@400;500;600&family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    /* paste all CSS tokens + component styles here */
  </style>
</head>
<body>
<div class="page">

  <!-- hero -->
  <!-- hr.section-rule -->

  <!-- sections -->

  <footer>...</footer>

</div>
</body>
</html>
```

---

## Intake

1. **Understand the content** — extract the topic, audience, key sections, and any code/commands/file paths from the user's description. Don't ask them to restructure their input.
2. **Map content to components** — steps → step timeline, commands → dark code block, tree structures → file tree, key points → callout, comparisons → table.
3. **Infer section count and numbering** — start at `01`, increment per logical section. 4–8 sections is typical.
4. **Ask for output path only.** Default: current working directory, kebab-case title derived from content (e.g. `personal-workflow.html`).
5. Write the complete HTML to the specified path and report the absolute path.

## Rules

- All CSS inline in `<style>`. No external stylesheets beyond Google Fonts.
- No JavaScript unless the user explicitly asks for interactivity.
- Every section ends with `<hr class="section-rule" />` except the last before the footer.
- Keep `<p>` and step body text in `font-family: Inter` — body `font-family: Lora` is for the outer shell only (large headings and flow text).
- Only include components the content actually needs — don't add a table "just in case".
