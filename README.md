# /html-system

A Claude Code skill that builds a detailed interactive SVG system diagram — multi-row pipeline with an animated traveling dot, full plain-English node descriptions, semantic color legend, and a stats grid. Outputs a single self-contained `.html` file.

Designed for systems you want to **study and understand deeply**, not just present. Every node carries a plain-English explanation, a "without this" consequence, and an analogy for non-technical readers.

## Install

```bash
mkdir -p ~/.claude/skills/html-system
curl -sf https://raw.githubusercontent.com/mitchelfletcher11/html-system/main/SKILL.md \
     -o ~/.claude/skills/html-system/SKILL.md
curl -sf https://raw.githubusercontent.com/mitchelfletcher11/html-system/main/template.html \
     -o ~/.claude/skills/html-system/template.html
```

`template.html` is the build scaffold the skill starts from — keep it next to
`SKILL.md`. No credentials or other setup required (see [SETUP.md](SETUP.md)).
Restart Claude Code, then run `/html-system` to verify.

## Usage

```
/html-system
```

Three usage patterns — the skill detects which applies automatically:

- **Current project** — "make a system diagram of this project" → reads the codebase and derives architecture
- **Another repo or path** — name a directory or repo → reads its key files and derives architecture
- **From a plan** — you have just described or produced a build plan → uses those components directly as nodes

No need to describe the system yourself in any case.

## What it produces

- Single `.html` file saved to `/outputs/`
- Scrollable SVG pipeline diagram, top-to-bottom
- Animated traveling dot moving at constant speed through every node
- Branch dots showing parallel paths splitting simultaneously
- Persistent breathing/pulse animations on AI and server nodes
- Full node descriptions: what it IS, how it works, without-this consequence, key constraint, plain-language analogy
- Semantic color system: purple = YOU (human action required), gold = AUTOMATED pipeline, green = STORED to disk
- Header with KPI chips (YOU / pipeline / stored counts)
- Horizontal legend strip and footer
- Optional stats grid with three panels below the SVG
- SVG text overlap detector fires red highlights if any badge text collides

## Semantic color system

| Color | Hex | Meaning |
|-------|-----|---------|
| Purple | `#8B5CF6` | **YOU** — requires human action |
| Gold | `#D4A017` | **AUTOMATED** — system runs without you |
| Green | `#4CAF50` | **STORED** — written to disk, persists across sessions |

Red `#E8351A` is reserved exclusively for the overlap diagnostic — it never appears in nodes or connections.

## Node description format

Every node renders six lines in strict order:

1. **Title** (12px Syne, white)
2. **What it IS** (7px, accent color) — technology defined in plain English with all abbreviations expanded
3. **How it works** (7px, #777) — mechanical step-by-step description
4. **Without this** (6.5px, #666) — what breaks or is missing if this node is removed
5. **Constraint / detail** (6px, #666) — version, limit, or key failure mode
6. **Analogy** (6px, #777) — one-sentence metaphor for non-technical readers

Abbreviations are always expanded on first use: `API (Application Programming Interface) = a published set of rules for how one program asks another to do work`.

## Layout rules

- Row gaps: **300px minimum** for verbose nodes (abbreviation-expanded descriptions run 190–230px of rendered height)
- `foreignObject{overflow:hidden}` clips any overflow — safe_h is calculated per column pair before writing
- Connection paths are routed to avoid all foreignObject description areas
- Traveling dot uses constant speed: `520px / 3000ms`

## Technical stack

- **Anime.js 3.2.1** — all animations (entrance, traveling dot, branch dots, persistent node pulses)
- **Syne** (Google Fonts) — headers and KPI chips
- **DM Mono** (Google Fonts) — all body text and node descriptions
- Dark theme: `#0A0A0A` background

## Release notes

See **[CHANGELOG.md](CHANGELOG.md)** for the full version history.
