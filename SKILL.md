---
name: html-system
description: >
  Generate an interactive, data-driven system diagram for any project or Salesforce org.
  Derives architecture from code or a plan, presents abstraction level options with time
  estimates, writes a JSON spec to knowledge/, then renders a self-contained HTML diagram
  to outputs/ using Dagre automatic layout. Every node carries ten description fields —
  plain-English explanations plus design pattern, principle, seam, tradeoff, and Enterprise
  Integration Pattern (with automatic Salesforce mapping when Salesforce indicators are
  detected). A sidebar opens on node click showing all fields and dependency explanations.
  Use this skill whenever a user wants to visualise a codebase, understand a system's
  architecture, learn design or integration patterns from a real example, or diagram a
  Salesforce integration.
allowed-tools: Write Read Bash WebFetch WebSearch
---

# html-system

Two outputs per run:
- `knowledge/[system]-spec.json` — living architecture document, updated as the system grows
- `outputs/[system]-l[N].html` — self-contained interactive diagram, level-suffixed so diagrams at different levels coexist

---

## HARD CONSTRAINTS

1. **Never output HTML in the response body.** Use the Write tool only. Typing `<!DOCTYPE` or `<html` in the response body is an error — stop and use Write instead.
2. **TodoWrite is the first tool call** — create the checklist below before any other work.
3. **Phase 1 is never skipped.** Always present the abstraction level table and wait for the user to pick — even when re-running with an existing spec, even when the user says "same inputs as before." The level choice is always the user's to make.
4. **Write the spec before reading the template.**
5. **After reading the template, the HTML Write must be the immediate next call** — no planning, no intermediate reads between template read and HTML write. Exception: when `flows` is present in the spec and `%%FLOWS%%` is not yet in the template, injecting `const FLOWS = [...]` into the template string before writing is the one permitted step between read and write.
6. **Never overwrite a diagram at a different abstraction level.** If `outputs/[system]-l1.html` already exists and Level 4 was chosen, write to `outputs/[system]-l4.html` — never overwrite the other level's file. Check the outputs/ directory before writing.

**TodoWrite checklist for every invocation:**
```
1. Derive system architecture
2. Present abstraction levels + time estimates — wait for user to pick
3. Write spec → knowledge/[system]-spec.json
4. Read template.html
5. Write HTML → outputs/[system]-l[N].html  (N = chosen level number)
6. Start local server if not running, confirm file readable
```

---

## When invoked — detect pattern

**Pattern A — Current project:** Read the codebase. Entry points, main modules, config, package manifests. Derive architecture from what you find. Do not ask the user to describe it.

**Pattern B — Another repo or path:** Read README, main entry, package manifest. Derive architecture from that.

**Pattern C — From a plan or conversation:** The system has already been described or a plan exists. Use those components directly. No reading required.

---

## Phase 0 — Derive architecture (silent)

Identify all of the following:
- **Entry points** — what starts the system (CLI, webhook, scheduler, UI)
- **Processing stages** — what transforms data or state along the way
- **External dependencies** — APIs, databases, queues, external services
- **Outputs** — what the system produces (files, messages, side effects)
- **Human touchpoints** — where a person takes action → purple YOU nodes
- **Automated steps** — what runs without human involvement → gold PIPELINE nodes
- **Persistent storage** — what gets written to disk and survives restarts → green STORED nodes

**Integration pattern sources — fetch before writing the first `integration` field on any node (skip entirely if no nodes will receive an integration value):**

1. **EIP catalog** — fetch the authoritative pattern list:
   `WebFetch https://www.enterpriseintegrationpatterns.com/patterns/messaging/`
   Extract all pattern names grouped by category. Use only names from this list when writing `integration` fields. Do not use pattern names from training memory.

2. **Salesforce integration patterns** — do not hardcode a URL; discover it fresh each run:
   `WebSearch "Salesforce integration patterns and practices official documentation filetype:pdf OR site:developer.salesforce.com OR site:architect.salesforce.com"`
   Take the top result that is the official Salesforce guide (PDF or developer docs page). `WebFetch` it to extract the current pattern names and their associated Salesforce APIs. Use only names from this fetched document when writing `salesforce_integration` fields.

**Platform detection:** While reading the codebase, check for Salesforce indicators: `sfdx-project.json`, `force-app/` directory, `.cls` Apex files, `package.xml`. If any are present, fetch the Salesforce guide (step 2 above) and populate `salesforce_integration` on every node that also has an `integration` value, mapping its EIP pattern to the equivalent Salesforce pattern and API. If no Salesforce indicators are found, omit `salesforce_integration` from all nodes.

---

## Phase 1 — Abstraction level selection

After deriving the architecture, present this table. Fill node counts and time estimates from the actual project — do not use fixed values. Time reflects spec-writing time at that level for this specific project. The Real-world equivalent column is fixed — do not change it per project.

| # | Level | What nodes represent | Real-world equivalent | Est. nodes | Est. time |
|---|-------|---------------------|-----------------------|------------|-----------|
| 1 | Source | Every function and class | C4 Code diagram — used for debugging and developer onboarding | [N] | [T] min |
| 2 | Module | One node per file or logical component | C4 Component diagram — used in tech lead and architecture reviews | [N] | [T] min |
| 3 | Data flow | What data exists at each transformation point | Data Flow Diagram (DFD) — used by data engineers tracing data through a pipeline | [N] | [T] min |
| 4 | Layer | Grouped architectural layers | C4 Container diagram — used by solution architects and DevOps | [N] | [T] min |
| 5 | Domain | Business process steps — no technology names | C4 System Context + DDD Bounded Context — used with non-technical stakeholders and product managers | [N] | [T] min |
| 6 | Concept | The core idea only | Executive summary / architecture overview — used in pitch decks and one-pagers | [N] | [T] min |

Wait for the user to pick before proceeding.

---

## Phase 2 — Write spec file

Write to `knowledge/[system-slug]-spec.json` in the current working directory.

### Full spec schema

```json
{
  "system": "Human-readable system name",
  "level": 2,
  "levelName": "Module",
  "nodes": [],
  "edges": [],
  "sections": []
}
```

### Node object

Every field is required unless marked optional.

```json
{
  "id": "unique-kebab-case-id",
  "label": "Display Name",
  "color": "gold",
  "section": "section-id",

  "is": "What this technology or component IS in plain English. Expand all abbreviations on first use across the spec.",
  "works": "How it operates mechanically, step by step.",
  "without": "What breaks or becomes impossible downstream if this node is removed.",
  "constraint": "One key limit, version requirement, or common failure mode.",
  "analogy": "One-sentence metaphor for a non-technical reader. Never omit.",

  "pattern": "Pattern Name — one-sentence definition of the pattern and why it applies to this specific node.",
  "principle": "Principle Name — one-sentence explanation of why this node exists as its own isolated component.",
  "seam": "What you would swap this for, and the specific condition that would trigger the swap. Omit only if no realistic alternative exists.",
  "tradeoff": "The specific technology you did NOT choose here, and the exact reason it was rejected for this use case. Omit only if no obvious alternative was considered.",
  "integration": "Integration Pattern Name — one-sentence definition of the Enterprise Integration Pattern and how this node implements or participates in it. Use only for nodes that embody a recognised pattern (Message Channel, Aggregator, Content Enricher, Request-Reply, Command Message, Message Store, Event-Driven Consumer, etc.). Omit for nodes where no integration pattern applies.",
  "salesforce_integration": "Salesforce Pattern Name — the official Salesforce integration pattern this node maps to and a one-sentence explanation of which Salesforce API or mechanism implements it. Use only pattern names from the document fetched in Phase 0. Only populated when Salesforce indicators are detected in Phase 0. Omit for non-Salesforce systems."
}
```

**color values:**
- `"purple"` — YOU: requires human action
- `"gold"` — PIPELINE: automated, runs without you
- `"green"` — STORED: written to disk, persists across sessions

**Abbreviation rule:** Every abbreviation must be expanded on first use anywhere in the spec:
`API (Application Programming Interface) — a set of rules for how one program asks another to do work over a network`
Subsequent nodes may use the short form freely.

**seam vs tradeoff — strictly different:**
- `seam` = what you COULD swap to in the future (names the replacement + the trigger condition)
- `tradeoff` = what you CHOSE NOT to use already (names the rejected alternative + the specific reason)

**pattern and principle — always named:**
Start with the pattern or principle name, then ` — `, then the definition. If you cannot name the pattern or principle, research it before writing.

### Edge object

```json
{
  "from": "node-id",
  "to": "node-id",
  "label": "Data or action label shown on the diagram",
  "dependency": "Why this dependency runs from [from] to [to] and not the reverse — names the specific design benefit of this direction."
}
```

`dependency` explains direction. Example: "Extractors depend on core utilities, not the reverse — this means a new extractor can be added without changing any existing code."

**Termination rule — do not model consumption as an edge.** A human node (purple) that receives the final output of a pipeline is a consumer, not a loop-back. Do not add an edge from the last output node back to the human node unless the human's response to that output triggers a new action that re-enters the same pipeline. "User views the result" is the end state of the graph, not an edge. If you cannot explain in the `dependency` field why reversing the arrow would break something, the edge does not belong.

### Section object

```json
{
  "id": "section-id",
  "label": "Section display label",
  "nodes": ["node-id-1", "node-id-2", "node-id-3"]
}
```

Sections are visual groupings only — they draw a subtle background highlight and label on the diagram. They do not affect layout. Dagre computes layout automatically from graph topology. Parallel nodes are placed in parallel; sequential chains are placed in sequence.

### Optional: `flows` array — enables collapsible explanation box

When present, generates a collapsible explanation box below the diagram. Each flow produces one paragraph of inline colored labels drawn from node `works` fields, showing how nodes connect narratively.

```json
"flows": [
  {
    "title": "Flow display name (shown as section header)",
    "sectionIds": ["section-id-1", "section-id-2"],
    "entryId": "id-of-first-node-in-flow",
    "approvalId": "id-of-last-node-in-flow"
  }
]
```

The explanation box renders: entry node `works` → all nodes in `sectionIds` (in section order) → approval node `works`. Each node name is shown as a colored inline dot + label using its semantic color (purple/gold/green).

**To activate in HTML output:** inject `const FLOWS = %%FLOWS%%;` into the HTML alongside `NODES`/`EDGES`/`SECTIONS`. The template.html placeholder `%%FLOWS%%` has not yet been added to the template — until it is, inject the constant directly when writing the output file. When `FLOWS` is an empty array `[]`, the explanation box is hidden automatically.

---

## Phase 3 — Render HTML

1. Read `~/.claude/skills/html-system/template.html`
2. Substitute placeholders:
   - `%%TITLE%%` → the `system` string from the spec (plain text, no quotes)
   - `%%NODES%%` → the `nodes` array as compact JSON (no pretty-printing)
   - `%%EDGES%%` → the `edges` array as compact JSON
   - `%%SECTIONS%%` → the `sections` array as compact JSON
3. Write to `outputs/[system-slug]-l[N].html` where N is the chosen level number (e.g. `html-system-l4.html` for Level 4)

After reading the template, the Write tool is the immediate next call.

If the level-suffixed output file already exists from a previous run, Read it (at minimum the first few lines) before calling Write — the Write tool will error if the target file was not read earlier in the same session.

---

## Semantic color system — three values only

| `color` value | Hex | What it means |
|---------------|-----|---------------|
| `"purple"` | `#8B5CF6` | YOU — a human must take action |
| `"gold"` | `#D4A017` | PIPELINE — system runs without you |
| `"green"` | `#4CAF50` | STORED — written to disk, survives restarts |

Never introduce a fourth color. If a node does not fit these three, reconsider its semantic role.

---

## Node description quality rules

**One role per field — strictly enforced:**
- `is` — what the technology IS, not its role in this system
- `works` — mechanical action, step by step
- `without` — consequence of removal downstream (not a description of what it does)
- `constraint` — one specific limit or failure mode (not a general note)
- `analogy` — metaphor for a non-technical reader (never a technical restatement)
- `pattern` — named design pattern + definition + why it applies here (not a category label)
- `principle` — named design principle + why it justifies this node being isolated
- `seam` — specific replacement + trigger condition
- `tradeoff` — specific rejected alternative + specific reason rejected
- `integration` — named EIP pattern from the fetched catalog + how this specific node implements it (not a category label; must match a real pattern name from the fetched list)
- `salesforce_integration` — named Salesforce pattern from the fetched guide + which specific Salesforce API implements it; omit if no Salesforce indicators detected

**Analogy rule:** The analogy must work for someone who has never seen the system. If it requires knowing what the node does to understand the analogy, rewrite it.

**`works` field and the explanation box:** when `flows` is present in the spec, the explanation box pastes `works` text directly after the node label inline — the reader sees `[NodeName] [works text]`. Write `works` as prose that continues naturally from the node name: "receives every user message and checks the gate timestamp before deciding whether to fire" rather than "Fires on submit. Checks timestamp. Returns boolean." Mechanical bullet style reads correctly in the sidebar but breaks in the explanation box.

**Depth rule:** `pattern`, `principle`, `seam`, and `tradeoff` must be specific to this node in this project. Generic statements like "follows good software design" are not acceptable.

---

## What the template renders automatically

These are not Claude's responsibility — the template handles them:

- Dagre automatic graph layout (parallel nodes placed in parallel; no coordinates needed)
- Node rendering as uniform styled rectangles with semantic color borders
- Edge rendering with arrowheads and labels
- Section background highlights with labels
- **Sidebar** — opens on node click; shows all ten description fields (is, works, without, constraint, analogy, pattern, principle, seam, tradeoff, integration pattern) plus Salesforce integration pattern when present, plus incoming and outgoing edge dependency explanations
- Stats grid — node counts by role, diagram layers (sections), seam count
- KPI chips — you / pipeline / stored counts
- Traveling dot animation — follows topological traversal order at constant speed
- Entrance animations — nodes and edges fade in on load
- Legend
- **Edge validator** — logs ghost IDs, missing-incoming nodes, and disconnected islands to the browser console at boot
- **Consistent zoom level** — the SVG viewBox is always at least 1930×1280 regardless of how many nodes the diagram has. Small diagrams are centered with whitespace rather than zoomed in. Do not attempt manual viewBox overrides.

---

## Extensible layout registry

The template contains a layout registry for cases where Dagre's automatic layout produces a clearly wrong result for a specific section. To add a new layout: write a new `registerLayout('name', fn)` call in template.html and apply it to the relevant section. The new layout becomes available for all future diagrams. Name it after the shape it produces.

For the vast majority of diagrams, Dagre handles layout automatically and the registry does not need to be extended.

---

## Optional: last-updated date in legend

To add a last-updated date to the legend row (aligned right), inject this snippet into the HTML output's `<script>` block:

```javascript
const ld = document.getElementById('legend-date');
if (ld) {
  const d = new Date(document.lastModified);
  ld.textContent = 'last updated · ' + d.getFullYear() + '-' +
    String(d.getMonth() + 1).padStart(2, '0') + '-' +
    String(d.getDate()).padStart(2, '0');
}
```

And add `<span id="legend-date" style="margin-left:auto;opacity:.55;font-size:11px"></span>` to the legend row in the HTML. The date reads `document.lastModified` — it updates automatically every time the file is saved. This feature is not yet in the template; inject directly when building the output file.

---

## Completion report

After writing the HTML, output only:
- Spec path
- HTML path
- One line: `[N] nodes · [E] edges · Level [#] ([name]) · http://localhost:9000/[system-slug]-l[#].html`

---

## Final Step — Write observation

Append a timestamped entry to `~/.claude/skills/html-system/observations.md`:

```markdown
## [ISO 8601 timestamp]
[One sentence: abstraction level chosen, node/edge count, any spec or layout issues, output quality]
```

If `observations.md` does not exist, create it with this frontmatter first:

```markdown
---
last-reviewed: 1970-01-01T00:00:00Z
---
```
