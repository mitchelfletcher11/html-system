# HTML System Overview Builder

Build a detailed interactive SVG reference diagram of a system — multi-row pipeline, animated traveling dot, full node descriptions, semantic color legend. Output is a self-contained `.html` file saved to `/outputs/`.

This format is for systems you want to study and understand deeply, not just present. Every node carries a plain-English explanation, a "without this" consequence, and an analogy.

---

## When invoked

Three usage patterns — detect which applies:

**Pattern A — Current project:** User says "make a system diagram of this project" or similar with no other context. Read the codebase: entry points, main modules, config files, dependencies, data flow. Derive the system architecture from what you find. Do not ask the user to describe it.

**Pattern B — Another repo or path:** User names a repo or directory. Read its key files (README, main entry, package.json / pyproject.toml / Cargo.toml, config). Derive architecture from that.

**Pattern C — From a plan:** User has just described or you have just produced a build plan, implementation plan, or architecture design. Use that plan's components directly as the nodes — no reading required.

In all three cases: do the derivation work first (read files, extract components), then write the HTML file in one shot. Do not ask the user to describe the system.

---

## Deriving the system architecture

When reading a codebase to build the diagram, extract:

1. **Entry points** — what starts the system? (CLI commands, API endpoints, UI, schedulers)
2. **Processing stages** — what transforms data or state along the way?
3. **External dependencies** — APIs, databases, file systems, message queues, external services
4. **Outputs** — what does the system produce? (files, responses, side effects, stored state)
5. **Human touchpoints** — where does a person take action? (these become purple YOU nodes)
6. **Automated steps** — what runs without human involvement? (gold PIPELINE nodes)
7. **Persistent storage** — what gets written to disk and survives restarts? (green STORED nodes)

Group the above into logical rows (typically 3–6 rows). Each row is a conceptual layer — e.g. row 1: entry / row 2: core processing / row 3: routing / row 4: knowledge / row 5: outputs. The diagram reads top-to-bottom and left-to-right.

---

## Step — Build

Derive the architecture, then save as `[system-slug].html` to `/outputs/`.

**CRITICAL — write-first rule:** The Write tool must be the **first tool call after** the architecture is derived. No planning passes between derivation and writing. Hold the full design in working memory and write the complete file in one shot. Do NOT output any HTML in the response body. Do NOT print the file contents to chat. Report only: file path, local server URL, one-line confirmation.

---

## Technical stack

```html
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@700;800&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js"></script>
```

---

## Semantic color system — three colors, no exceptions

| Color | Hex | Meaning |
|-------|-----|---------|
| Purple | `#8B5CF6` | **YOU** — requires human action |
| Gold | `#D4A017` | **AUTOMATED** — system runs without you |
| Green | `#4CAF50` | **STORED** — written to disk, persists across sessions |

Apply to: node icon strokes, first description line color, connection line colors, cdot fills, arrowhead markers.

**Red `#E8351A` is reserved exclusively for the overlap diagnostic.** Never use it in nodes, connections, icons, or descriptions. The reason red is available as a diagnostic signal is precisely because it appears nowhere else in the design — if you see red on screen, it means the detector fired, not that something is highlighted for emphasis.

**Introduce a new color only when a node has a genuinely distinct role** that cannot be expressed as a variation of the three existing categories. The bar is high: the new color must mean exactly one thing across the entire diagram, it must appear in the legend, and it must not resemble any existing color closely enough to cause confusion. If in doubt, discuss with the user before adding a fourth color. Keep the total number of semantic colors as low as possible — every additional color reduces the signal value of all the others.

Horizontal legend strip between header and SVG. **No legend title** — just dots and their meaning, nothing else:

```html
<div class="legend">
  <span><i style="background:#8B5CF6"></i>you — action required</span>
  <span><i style="background:#D4A017"></i>automated pipeline</span>
  <span><i style="background:#4CAF50"></i>stored to disk</span>
</div>
```
```css
.legend{display:flex;gap:clamp(10px,1.6vw,26px);align-items:center;padding:clamp(2px,0.3vh,5px) 0;flex-shrink:0}
.legend span{display:flex;align-items:center;gap:5px;font-size:clamp(6.5px,0.72vw,9px);color:#666}
.legend i{display:inline-block;width:7px;height:7px;border-radius:50%;flex-shrink:0}
```

Each label describes the **significance** of the color — what it means for the reader — not the name of the category. "action required" not "user", "stored to disk" not "files". If a fourth color is introduced, add it to the legend with an equally plain description. The footer must also be updated to match.

Footer:
```html
<footer>
  <span><i style="background:#8B5CF6"></i>YOU — manual steps requiring your action</span>
  <span><i style="background:#D4A017"></i>PIPELINE — automated, system runs without you</span>
  <span><i style="background:#4CAF50"></i>STORED — files written to disk, persist across sessions</span>
</footer>
```
```css
footer{display:flex;align-items:center;justify-content:center;gap:clamp(8px,1.3vw,20px);border-top:1px solid rgba(255,255,255,.05);padding-top:clamp(2px,0.3vh,5px);flex-shrink:0}
footer span{display:flex;align-items:center;gap:4px;font-size:clamp(6px,0.66vw,8.5px);color:#444}
footer i{display:inline-block;width:6px;height:6px;border-radius:50%;flex-shrink:0}
```

---

## Body layout

This is a **single vertically-scrolling page**. The SVG is taller than the viewport — the user scrolls to read the full diagram. Do not use `overflow:hidden` on `html` or `body`.

```css
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0}
html{overflow-y:auto;background:#0A0A0A}
body{background:#0A0A0A;color:#F5F2EE;font-family:'DM Mono',monospace;
  display:flex;flex-direction:column;
  padding:clamp(6px,0.9vh,12px) clamp(12px,2vw,32px) clamp(8px,1vh,16px);
  gap:clamp(4px,0.5vh,8px)}
body>*{min-height:0}
svg.fill{width:100%;height:auto;display:block}
foreignObject{overflow:hidden}
```

Row order: header · legend · SVG · stats grid (optional) · footer.

### Header with KPI chips

The header contains the system name on the left and small count chips on the right — one chip per key metric (e.g. number of YOU nodes, PIPELINE nodes, STORED nodes, distinct paths).

```html
<div class="hdr">
  <div class="logo-title">System Name <span style="color:#444;font-weight:400">— subtitle</span></div>
  <div class="kpis">
    <div class="kpi"><span class="kpi-v" style="color:#8B5CF6">2</span><span class="kpi-l">you</span></div>
    <div class="kpi"><span class="kpi-v" style="color:#D4A017">12</span><span class="kpi-l">pipeline</span></div>
    <div class="kpi"><span class="kpi-v" style="color:#4CAF50">4</span><span class="kpi-l">stored</span></div>
  </div>
</div>
```
```css
.hdr{display:flex;align-items:center;justify-content:space-between;border-bottom:1px solid rgba(255,255,255,.07);padding-bottom:clamp(3px,0.4vh,6px);flex-shrink:0}
.logo-title{font-family:'Syne',sans-serif;font-size:clamp(14px,1.9vw,26px);font-weight:800;letter-spacing:-.02em}
.kpis{display:flex;gap:clamp(4px,0.6vw,10px);align-items:center}
.kpi{background:#141414;border:1px solid rgba(255,255,255,.07);border-radius:4px;padding:clamp(2px,0.3vh,5px) clamp(6px,0.7vw,11px);display:flex;flex-direction:column;align-items:center}
.kpi-v{font-family:'Syne',sans-serif;font-size:clamp(11px,1.3vw,18px);font-weight:800}
.kpi-l{font-size:clamp(5.5px,0.62vw,7.5px);color:#555;letter-spacing:.12em;text-transform:uppercase}
```

KPI chip colors match the semantic system: purple for YOU count, gold for pipeline count, green for stored/output count.

---

## SVG viewBox

```html
<svg class="fill" viewBox="-40 0 [W] [H]" preserveAspectRatio="xMidYMid meet" style="overflow:visible">
```

- Width: rightmost node x + 120 (right fO margin) + 40 (left offset) + 40 (right buffer)
- Height: bottom row node y + largest bottom fO content height + 80 buffer
- Start x at `-40` so left-edge descriptions have room

### Markers and filters

```svg
<defs>
  <marker id="af"  markerWidth="6" markerHeight="4" refX="5" refY="2" orient="auto" markerUnits="userSpaceOnUse"><path d="M0,0 L0,4 L6,2 z" fill="rgba(139,92,246,.9)"/></marker>
  <marker id="agf" markerWidth="6" markerHeight="4" refX="5" refY="2" orient="auto" markerUnits="userSpaceOnUse"><path d="M0,0 L0,4 L6,2 z" fill="rgba(212,160,23,.9)"/></marker>
  <marker id="agr" markerWidth="6" markerHeight="4" refX="5" refY="2" orient="auto" markerUnits="userSpaceOnUse"><path d="M0,0 L0,4 L6,2 z" fill="rgba(76,175,80,.9)"/></marker>
  <filter id="gR" x="-60%" y="-60%" width="220%" height="220%">
    <feGaussianBlur stdDeviation="6" result="b"/>
    <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
  </filter>
  <filter id="gs" x="-50%" y="-50%" width="200%" height="200%"><feGaussianBlur stdDeviation="3"/></filter>
</defs>
```

---

## Node structure — always nested groups

```svg
<g transform="translate(cx, cy)"><g class="nd" opacity="0">
  <!-- icon shapes centered at 0,0 -->
  <foreignObject x="-120" y="[icon_bottom+5]" width="240" height="[safe_h]">
    <div xmlns="http://www.w3.org/1999/xhtml" style="font-family:'DM Mono',monospace;text-align:left;line-height:1.5;padding:0 5px;">
      <div style="font-family:'Syne',sans-serif;font-weight:800;font-size:12px;color:#F5F2EE;margin-bottom:3px;">Node Name</div>
      <div style="font-size:7px;color:[accent];margin-bottom:2px;">What this IS — define the technology in plain English, not its role</div>
      <div style="font-size:7px;color:#777;margin-bottom:2px;">How it works mechanically</div>
      <div style="font-size:6.5px;color:#666;margin-bottom:2px;">Without this: what breaks or is missing downstream</div>
      <div style="font-size:6px;color:#666;margin-bottom:2px;">Key constraint, limit, or common failure mode</div>
      <div style="font-size:6px;color:#4a4a4a;">Analogy: one-sentence metaphor</div>
    </div>
  </foreignObject>
</g></g>
```

Accent color = node's semantic color for the first description line only. All subsequent lines use greys. Analogy always `#4a4a4a`.

**Node fill and stroke colors:**
- Most node icons: `fill="#141414"` (panel dark), stroke in the node's semantic color
- YOU nodes: `fill="#181818"` (one shade lighter — distinguishes human-action nodes from pure automation)
- Node stroke opacity: `.55–.6` for main nodes, `.4` for sub-chain nodes

**foreignObject dimensions — three sizes:**

| Node type | x offset | width | usable text width |
|-----------|----------|-------|-------------------|
| Main node (large icon) | `-120` | `240` | 230px (5px padding each side) |
| Sub-chain node (small icon) | `-100` | `200` | 190px |
| Leaf/file node (tiny icon) | `-90` | `180` | 170px |

**height = safe_h** — calculated per column (see Safe foreignObject height section). Set it to the calculated value and rely on `foreignObject{overflow:hidden}` to clip any overflow. Do not guess a height and hope it fits.

The outer `<g transform>` positions the node. The inner `<g class="nd">` is the Anime.js animation target. Never put both on the same element.

### SVG icon design

Icons are centered at 0,0 within the outer `<g transform>`. Keep them compact — icon shapes stay within ±20px of the center point. All use `fill="#141414"` (panel dark) with a stroke in the node's semantic color.

**Icon shapes by conceptual role:**

| Role | Shape | Example SVG |
|------|-------|-------------|
| Human / user | Circle head + path body | `<circle cx="0" cy="-14" r="8" .../><path d="M-11,0 Q-11,-8 0,-8 Q11,-8 11,0 L11,8 L-11,8 Z" .../>` |
| Computer / terminal | Rect with text lines | `<rect x="-17" y="-22" width="34" height="22" rx="3" .../><text x="-11" y="-9" ...>$_</text>` |
| Cloud API / network | Cloud path | Multi-curve `<path d="M-18,2 Q-20,-10 -10,-13 ..."` |
| AI / LLM brain | Circle with inner ring + 3 dots | `<circle r="20" .../><circle r="12" .../><circle cx="-7" r="2.8" />` × 3 |
| File / document | Small rect with horizontal lines | `<rect x="-13" y="-20" .../><line y1="-13" .../><line y1="-6" .../>` |
| Decision / router | Upward triangle | `<polygon points="0,-22 20,6 -20,6" .../>` |
| Knowledge / database | Three stacked rects | `<rect y="-22" .../><rect y="-14" .../><rect y="-6" .../>` |
| Memory / persistence | Rounded rect with vertical lines | `<rect rx="7" .../><line x1="-6" .../>` × 3 |
| Output / generator | Rect with down-arrow icon | `<rect .../><line x1="0" y1="-10" x2="0" y2="-3" .../><path d="M-6,-6 L0,0 L6,-6" .../>` |

**Badge text inside icons** (e.g. ">70%", "skill", "REST", "?") — small centered SVG `<text>` in the semantic color, font-family DM Mono, typically 7–8px, `text-anchor="middle"`.

Icons with badge text must pass the overlap detector check — badge texts from adjacent nodes must not overlap. If they do, increase node spacing or adjust icon x-positions.

### Description writing rules — enforced in every node

**No abbreviations without full expansion.** Every technical term or acronym must be written out the first time it appears in a node description, using this format:

```
ABBR (Full Name) = plain-language definition of what it actually IS
```

Examples from correct usage:
- `API (Application Programming Interface) = a set of published rules for how one program asks another to do work over the internet`
- `CLI (Command Line Interface) = a text-only program you control by typing commands rather than clicking buttons`
- `LLM (Large Language Model) = a neural network trained on billions of documents to predict the most probable next token`
- `JSON (JavaScript Object Notation) = plain text structured as key:value pairs — e.g. {model:claude-sonnet, content:your question}`
- `MCP (Model Context Protocol) = an open standard that lets an AI model call tools running as separate processes on the same machine`

The definition must be comprehensible to someone with no technical background. Never restate the name. Never say "a tool that does X" — say what it physically IS.

**Line purposes — strictly one role per line:**
1. **Title** (12px Syne, white) — node name only, no description
2. **What it IS** (7px, accent color) — the technical definition with abbreviation expanded
3. **How it works** (7px, #777) — the mechanical action, what happens step by step
4. **Without this** (6.5px, #666) — what breaks or is impossible if this node is removed
5. **Constraint / detail** (6px, #666) — version, limit, failure mode, or key dependency
6. **Analogy** (6px, #4a4a4a) — "Analogy: [one-sentence metaphor for non-technical readers]"

Never collapse two of these into one line. Never skip the analogy.

---

## Safe foreignObject height — the critical layout rule

This is the rule that causes the most iteration if skipped. Do the full calculation before placing any node.

### Step 1 — Calculate safe_h for every node

```
safe_h = (next_element_icon_top_in_same_x_column) - (this_node_fO_start_y)

fO_start_y   = node_center_y + fO_y_offset
               fO_y_offset = distance from center to top of foreignObject
               (rect icon: +4 to +9, circle icon: +r+5, polygon: +9)

icon_top     = next_node_center_y - icon_top_offset
               icon_top_offset by shape: rect=18, circle=r+10, polygon=22

x_column     = fO x-range of this node (e.g. center ± 100 for width=200)
constraint   = ONLY apply if another node's fO x-range overlaps this one
```

If no node shares the x-column below, safe_h is unconstrained — use 160–250px.

### Step 2 — Estimate actual content height

Every description renders as wrapped monospace HTML. Estimate before writing:

```
line_height = font_size × 1.5
chars_per_line ≈ (fO_usable_width_px) / (font_size_px × 0.6)
  where usable_width = fO_width - 10  (5px padding each side)

line_count = ceil(char_count / chars_per_line)
line_height_px = line_count × (font_size × 1.5) + margin_bottom

**Critical:** the estimates below assume short, 2-wrapped-line descriptions. When descriptions follow the abbreviation-expansion rule (e.g. "API (Application Programming Interface) = ..."), each line wraps to 4-6 lines and real content height is 190–230px per node. Always count the actual character length of each line and apply the chars_per_line formula — do not use the totals below as a substitute.

Short/compact node (5 lines + analogy, 1–2 wrapped lines each, width=240):
  title   12px Syne  1 line  = 18px + 3px margin = 21px
  line1   7px        2 lines = 21px + 2px margin = 23px
  line2   7px        2 lines = 21px + 2px margin = 23px
  line3   6.5px      2 lines = 20px + 2px margin = 22px
  line4   6px        2 lines = 18px + 2px margin = 20px
  analogy 6px        1 line  = 9px
  TOTAL ≈ 118px  ← only valid when all lines are short

Verbose node (5 lines + analogy, abbreviations expanded, width=240):
  title   12px Syne  1 line  = 21px
  line1   7px        4-6 lines = 42–63px + 2px margin
  line2   7px        4-5 lines = 42–52px + 2px margin
  line3   6.5px      2-3 lines = 20–29px + 2px margin
  line4   6px        2-3 lines = 18–27px + 2px margin
  analogy 6px        1-2 lines = 9–18px
  TOTAL ≈ 190–230px  ← use this range when descriptions are sentence-length

Sub-node (3 lines + analogy, width=200):
  TOTAL ≈ 70–90px (short) / 120–150px (verbose)
```

### Step 3 — Verify fit. Fix row gaps if needed.

If `safe_h < content_height`: **increase the row gap** between this node and the one constraining it. Never shorten descriptions.

When you move a row down, all rows below it must also shift, all connection line y-coordinates must update, all animation cy values must update, and the viewBox height must be rechecked. Do this cascade before writing — not after.

### Step 4 — Path routing through tight gaps

When a connection path must pass between two rows and both fO boxes are close to each other:

```
gap = next_row_icon_top - prev_row_fO_bottom
      = (next_center_y - icon_offset) - (prev_center_y + fO_y + safe_h)
```

If the gap is ≥ 4px, the path can travel through it. Use `y = prev_row_fO_bottom + 2` as the routing y.

If the gap is 0 or negative, the rows are too close — increase spacing.

**Route vertical exit paths away from descriptions.** When a path must exit a node going downward and then cross horizontally:
- Exit from the icon's side edge (not center) so the vertical clears the fO x-range
- Drop past all fO bottoms in that column before turning horizontal
- Travel horizontally at a y-level confirmed clear of all fO y-ranges along the x-span

`foreignObject{overflow:hidden}` in CSS ensures content never bleeds past safe_h regardless of wrapping.

---

## Row spacing minimums

| Description depth | Approx content height | Min row gap needed |
|-------------------|-----------------------|--------------------|
| Full verbose (5 lines + analogy, abbreviations expanded) | ~190–230px | **300px** between row centers |
| Full short (5 lines + analogy, brief descriptions) | ~115–125px | 170px between row centers |
| Medium (3 lines + analogy) | ~70–90px | 120px between row centers |
| Compact (1 line + analogy) | ~45–55px | 80px between row centers |

**Default to 300px row gaps.** The abbreviation-expansion rule (required for every node) produces verbose descriptions that reliably exceed 170px. Using 170px gaps clips the constraint line and analogy on most nodes. Only drop to 170px when descriptions are genuinely brief (single short sentences per line).

For branch rows (multiple routes off one branch point): minimum 150px between each branch row center when descriptions are verbose.

**Pre-layout checklist** — run this before writing the file:
1. List all column pairs that share x-range overlap
2. For each pair, calculate safe_h and content_height
3. If safe_h < content_height anywhere, adjust row positions now
4. Re-verify all connection path y-levels don't cross fO areas
5. Confirm viewBox height covers all content

---

## Branch layout — routing nodes

Branches represent mutually exclusive paths (e.g. "if condition A do this, else do that"). They require careful vertical positioning.

### Rules for branch structure

**1. The routing node sits at the same y as the first branch.**
The node that decides the branch (e.g. a router, classifier, switch) must be positioned at exactly the y-level of the first/top branch row. This produces a straight horizontal connector from the router into the first branch — no drop. If the router is above the first branch, there is an awkward descending jog.

**2. A vertical branch spine runs from first to last branch y, at a fixed x.**
Place a vertical dashed line at `x = router_right_edge + 80–120px`. This spine connects all branch entry points. Branches exit the spine horizontally to the right.

```svg
<!-- Spine — from first branch y to last branch y -->
<line class="conn" x1="820" y1="[first_branch_y]" x2="820" y2="[last_branch_y]"
      stroke="rgba(212,160,23,.35)" stroke-width="1.2" stroke-dasharray="4,4" opacity="0"/>

<!-- Router → spine horizontal (at router y = first branch y) -->
<line class="conn" x1="[router_right]" y1="[router_y]" x2="820" y2="[router_y]"
      stroke="rgba(212,160,23,.4)" stroke-width="1.2" stroke-dasharray="4,4" opacity="0"/>

<!-- Branch exits rightward from spine -->
<path class="conn" d="M820,[branch_y] H[entry_node_left]" .../>
```

**3. All branch entry nodes start at the same x.**
Every branch's first node must be at the same x-coordinate so the branches are left-aligned when viewed as a column. This makes the depth difference visible — a shallow branch has fewer nodes to the right than a deep one.

**4. Sub-nodes within a branch are spaced at consistent intervals (300px default).**
Each subsequent node in a branch is placed 300px to the right of the previous one.

**5. Row gaps between branches follow the content height rule.**
Each branch row must have enough vertical gap above it (from the branch row above) to fit its entry node's description. Apply the safe_h calculation between every pair of stacked branch rows.

**6. Return paths converge at a single point below the last branch row.**
All branch return paths drop down to a shared convergence y, then travel to the next main node. Mark the convergence with a small open circle:

```svg
<!-- Convergence marker -->
<circle class="conn" cx="[conv_x]" cy="[conv_y]" r="3.5" fill="none"
        stroke="rgba(212,160,23,.45)" stroke-width="1.3" opacity="0"/>

<!-- Return path from a branch — drop to convergence, then continue -->
<path class="conn" d="M[branch_exit_x],[branch_y] L[branch_exit_x],[conv_y] L[conv_x],[conv_y] L[conv_x],[next_node_top]"
      stroke="rgba(212,160,23,.3)" stroke-width="1" stroke-dasharray="3,5"
      fill="none" marker-end="url(#agf)" opacity="0"/>
```

Convergence y should be: `last_branch_y + last_branch_fO_content_height + 30px`. Confirm this is below all branch descriptions before writing.

### Branch animation — full pattern

The branch animation has two simultaneous behaviors:
- **Branch dots** show all possible routes splitting at once (what the system CAN do)
- **Main dot** cycles through one complete path per loop (what happens on a given run)

This is the complete pattern:

```javascript
// In runDot(), when the main dot arrives at the branch node and pauses:
p1.add({targets:dot, cx:routerX, cy:routerY, duration:900, complete:function(){
  anime({targets:dot, opacity:0, duration:280});   // main dot fades out
  runBranchSplit(function(){                       // all branch dots animate simultaneously
    // branch dots finish → main dot reappears on the selected path for this cycle
    const p2 = anime.timeline({easing:'easeInOutQuad'});
    if(path===0){
      anime.set(dot,{cx:spineX, cy:branch1Y, opacity:0});
      p2.add({targets:dot, opacity:1, duration:280});
      p2.add({targets:dot, cx:node1X, cy:branch1Y, duration:ms(dist1)});
      // ... full route 1 trace ...
    } else if(path===1){
      // ... full route 2 trace ...
    } else {
      // ... full route 3 trace ...
    }
    // All paths end by calling runR4(p2) or equivalent to continue after convergence
  });
}});

// Cycle counter — each complete loop shows a different path
let cycle = 0;
function runDot(){
  const path = cycle % numPaths;
  cycle++;
  // ...
}
```

```javascript
function runBranchSplit(onDone){
  anime.set('#bdot1',{cx:branchX, cy:branchY, opacity:0});
  anime.set('#bdot2',{cx:branchX, cy:branchY, opacity:0});
  anime.set('#bdot3',{cx:branchX, cy:branchY, opacity:0});

  let n=0;
  function tick(){ if(++n===3) onDone(); }

  anime.timeline({easing:'easeInOutQuad', complete:tick})
    .add({targets:'#bdot1', opacity:1, duration:180})
    .add({targets:'#bdot1', cx:spineX, cy:branchY, duration:ms(spineGap)})
    .add({targets:'#bdot1', cx:entry1X, cy:branch1Y, duration:ms(entryDist)})
    .add({targets:'#bdot1', cx:entry1X, cy:branch1Y, duration:600})  // pause at node
    .add({targets:'#bdot1', opacity:0, duration:300});

  // bdot2 drops to branch2Y, bdot3 drops to branch3Y — all start at branchY
  // bdot durations use ms() for consistent speed
}
```

**SVG attribute rule:** The `cx`/`cy` attributes on branch dot SVG elements must equal the `anime.set()` starting values. If they differ, the dot renders at the wrong position on page load before animations start.

```svg
<!-- SVG attribute cy MUST match anime.set() cy -->
<circle id="bdot1" cx="[branchX]" cy="[branchY]" r="4.5" fill="#D4A017" opacity="0" filter="url(#gR)"/>
<circle id="bdot2" cx="[branchX]" cy="[branchY]" r="4.5" fill="#D4A017" opacity="0" filter="url(#gR)"/>
<circle id="bdot3" cx="[branchX]" cy="[branchY]" r="4.5" fill="#D4A017" opacity="0" filter="url(#gR)"/>
```

**Branch dot color rule:** All branch dots use gold `#D4A017` when all routes are automated. Only use purple for a branch dot if that specific route requires human action. Never assign colors decoratively. The main `tdot` is always gold.

---

## Connection lines

**`class="conn"` and `class="cdot"` rule:** Connection lines and text labels carry `class="conn"` and `opacity="0"`. Small indicator circles (cdots, convergence markers) carry `class="cdot"` and `opacity="0"`. The entrance animation targets both selectors: `.conn,.cdot`. Any element missing both classes will appear immediately on load rather than animating in.

**Opacity and dash pattern graduation** — visual hierarchy for primary vs secondary flow:

| Connection type | Stroke opacity | stroke-width | stroke-dasharray |
|----------------|---------------|-------------|-----------------|
| Primary forward path | `.45` | `1.4` | `5,4` |
| Branch forward path | `.4–.45` | `1.2` | `4,4` |
| Return / secondary path | `.22–.3` | `1.0–1.3` | `3,5` |
| Sub-branching (thin) | `.3` | `1.0` | `3,4` |

Lower opacity and tighter dashes recede visually — return paths carry less semantic weight than the forward paths that drive the animation.

```svg
<!-- Primary forward (automated) -->
<line class="conn" x1="[from_right]" y1="[y]" x2="[to_left]" y2="[y]"
      stroke="rgba(212,160,23,.45)" stroke-width="1.4" stroke-dasharray="5,4"
      marker-end="url(#agf)" opacity="0"/>
<circle class="cdot" cx="[from_right]" cy="[y]" r="3" fill="#D4A017" opacity="0"/>

<!-- Return path (lower opacity, tighter dash) -->
<path class="conn" d="M[x],[y] L[x],[conv_y] L[conv_x],[conv_y]"
      stroke="rgba(212,160,23,.3)" stroke-width="1" stroke-dasharray="3,5"
      fill="none" marker-end="url(#agf)" opacity="0"/>
```

Use `url(#af)` + purple stroke for YOU connections. Use `url(#agr)` + green stroke for STORED connections.

### Connection label texts

Add a small text label above key connection lines to name the protocol, data type, or action being carried. Use fill `#282828` (very dark, nearly invisible against the background — visible but not distracting):

```svg
<text class="conn" x="[midpoint_x]" y="[line_y - 9]" text-anchor="middle"
      font-family="DM Mono" font-size="8" fill="#282828" opacity="0">label text</text>
```

Place the `<text>` element immediately after the `<line>` it labels. Examples: "HTTPS POST", "CLI input", "SSE stream", "File I/O", "MCP stdio", "manual carry", "Write tool". Only label connections where the transport mechanism adds meaning — don't label every line.

**Routing rule:** Paths must not cross other nodes' foreignObject description areas. Before placing a horizontal segment at y-level X, check every node whose fO x-range intersects the path x-range — if X falls within that node's fO y-range, reroute. Clearance options:
- Drop below: route at fO_bottom + 5px
- Lift above: route at fO_start - 5px (must also be above icon tops)
- Thread the gap: the 4px window between adjacent row fO bottoms and icon tops is valid

A vertical exit from a node's icon bottom passing through that same node's description is accepted convention. It must not pass through any other node's description.

---

## Animated traveling dot

Constant speed regardless of segment length:
```javascript
const SPD = 520/3000;
function ms(px){ return Math.round(px/SPD); }
```

Chain segments with `anime.timeline`, one step per node in pipeline order:
```javascript
const dot = document.getElementById('tdot');
function runDot(){
  anime.set(dot,{opacity:0,cx:startX,cy:startY});
  const tl = anime.timeline({easing:'easeInOutQuad'});
  tl.add({targets:dot,opacity:1,duration:300});
  tl.add({targets:dot,cx:x1,cy:y1,duration:ms(dist1)});
  tl.add({targets:dot,cx:x2,cy:y2,duration:ms(dist2)});
  // ...
  tl.add({targets:dot,opacity:0,duration:500,
          complete:function(){setTimeout(runDot,1200);}});
}
setTimeout(runDot,1500);
```

For branching systems, cycle through paths and show branch splits simultaneously using parallel timelines that each call a shared `tick()` counter.

Dot element (place at end of SVG):
```svg
<circle id="tdot" cx="[startX]" cy="[startY]" r="5.5" fill="#D4A017" opacity="0" filter="url(#gR)"/>
```

---

## Stats grid (optional)

Include below the SVG when the system has useful at-a-glance metrics — counts, protocols, session steps, knowledge state. Three equal-ish columns.

```html
<div class="sg">
  <div class="sp">
    <div class="sp-h">Panel Title</div>
    <div class="sp-row"><span class="sp-dot" style="background:#D4A017"></span><span style="font-size:clamp(5.5px,0.64vh,8.5px);color:#F5F2EE">Item</span><span style="font-size:clamp(5px,0.58vh,7.5px);color:#555">&nbsp;→ detail</span></div>
    <div class="sp-note">Footer note</div>
  </div>
  <!-- repeat for each panel -->
</div>
```

```css
.sg{display:grid;grid-template-columns:1fr 1.5fr 1fr;gap:clamp(3px,0.45vh,6px);height:clamp(80px,10vh,130px);min-height:0;flex-shrink:0}
.sp{background:#141414;border:1px solid rgba(255,255,255,.06);border-radius:5px;padding:clamp(4px,0.55vh,7px) clamp(6px,0.7vw,10px);overflow:hidden;display:flex;flex-direction:column;gap:clamp(1px,0.16vh,2px)}
.sp-h{font-family:'Syne',sans-serif;font-weight:800;font-size:clamp(6px,0.72vh,9.5px);color:#444;letter-spacing:.18em;text-transform:uppercase;margin-bottom:clamp(1px,0.18vh,3px);flex-shrink:0}
.sp-row{display:flex;gap:5px;align-items:flex-start;flex-shrink:0;line-height:1.3;margin-bottom:clamp(0px,0.1vh,1px)}
.sp-dot{width:5px;height:5px;min-width:5px;border-radius:50%;margin-top:2px}
.sp-note{font-family:'DM Mono',monospace;font-size:clamp(4.5px,0.54vh,7px);color:#444;margin-top:auto;padding-top:clamp(1px,0.16vh,2px);border-top:1px solid rgba(255,255,255,.04);flex-shrink:0}
```

Use colored `sp-dot` elements matching the semantic system — gold for automated protocols, green for file I/O, purple for manual steps. Numbered step lists use inline colored `①②③` spans styled in the semantic color.

---

## Entrance animations

```javascript
anime({targets:'.sp',opacity:[0,1],translateY:[3,0],delay:anime.stagger(30,{start:200}),easing:'easeOutQuad',duration:320});
anime({targets:'.nd',opacity:[0,1],translateY:[14,0],delay:anime.stagger(50,{start:70}),easing:'easeOutExpo',duration:580});
anime({targets:'.conn,.cdot',opacity:[0,1],delay:anime.stagger(25,{start:650}),easing:'easeOutQuad',duration:220});
```

### Persistent node animations — "alive" feel

Key nodes can have always-on looping animations that run independently of the traveling dot. These fire inside `window.onload` alongside the entrance animations. Assign an `id` to the outer group of any node that needs custom targeting:

```svg
<g id="claudeNode" transform="translate(1740,80)"><g class="nd" opacity="0">...</g></g>
```

**LLM / brain node — breathing outer ring + blinking dots:**
```javascript
// Outer circle breathes
anime({targets:'#claudeNode > g > circle:first-of-type',
  r:[20,24,20],opacity:[1,0.4,1],
  easing:'easeInOutSine',duration:2800,loop:true});
// Three inner dots blink in staggered sequence
anime({targets:['#srv1','#srv2','#srv3'],
  opacity:[1,0.2,1],delay:anime.stagger(400),
  easing:'easeInOutSine',duration:1200,loop:true});
```

Where to use persistent animations:
- The central intelligence node (LLM, AI engine) — breathing/pulse conveys active thinking
- Any node that represents a continuously running process (server, daemon, optimizer)
- Avoid on human/YOU nodes and static storage nodes — those should feel inert by contrast

The traveling dot handles "what's happening right now." Persistent animations handle "what's always running."

---

CDN fallback — always `window.onload`:
```javascript
window.onload=()=>{
  if(typeof anime==='undefined'){
    document.querySelectorAll('.sp,.nd,.conn,.cdot').forEach(e=>{e.style.opacity=1;});
    return;
  }
  // all animations here
};
```

---

## SVG text overlap detector

Embed before `</body>` in every output:
```html
<script>
window.addEventListener('load',()=>{setTimeout(()=>{
  const allText=[...document.querySelectorAll('svg text')];
  function nodeGroup(el){let p=el.parentElement;while(p&&p.tagName!=='svg'){if(p.classList.contains('nd'))return p;p=p.parentElement;}return null;}
  const items=allText.map(t=>({el:t,r:t.getBoundingClientRect(),g:nodeGroup(t)}));
  let count=0;
  for(let i=0;i<items.length;i++){for(let j=i+1;j<items.length;j++){
    if(items[i].g&&items[i].g===items[j].g)continue;
    const a=items[i].r,b=items[j].r;
    if(a.left<b.right&&a.right>b.left&&a.top<b.bottom&&a.bottom>b.top){
      items[i].el.setAttribute('fill','#E8351A');items[j].el.setAttribute('fill','#E8351A');count++;
    }
  }}
  if(count>0){const w=document.createElement('div');w.style.cssText='position:fixed;top:10px;left:50%;transform:translateX(-50%);background:#E8351A;color:#fff;padding:6px 14px;font-family:monospace;font-size:11px;border-radius:4px;z-index:9999;pointer-events:none';w.textContent=count+' SVG text overlap'+(count>1?'s':'')+' — highlighted red';document.body.appendChild(w);}
},2500);});
</script>
```

Detects overlapping SVG badge text (icon labels) only — foreignObject HTML is not checked. Zero overlaps = icon badges are clear.

**`addEventListener` vs `window.onload` — do not mix up:** The overlap detector must use `window.addEventListener('load', ...)`. The animation block must use `window.onload = () => {...}`. Both live in the same `<script>` tag and coexist without conflict because they use different registration methods. If you write both as `window.onload`, the second assignment silently overwrites the first and the overlap detector never fires.

---

## Tracked checklist — dynamic TodoWrite list

Use the `TodoWrite` tool to create a **live, updating task list** at the start of work. This is not a mental checklist — it is a real list that the user can see updating as each task is ticked off. Mark each item `completed` with a follow-up `TodoWrite` call immediately after finishing it. Do not report the task complete until every item shows as done.

Create this list before deriving the architecture:

```
1. Derive system architecture from code / plan
2. Calculate safe_h for every x-column pair
3. Verify all row gaps sufficient — cascade-adjust if needed
4. Map all connection paths — confirm no fO crossings
5. Write HTML file to /outputs/
6. Confirm file readable at localhost:9000
7. Verify semantic colors: purple=YOU, gold=AUTOMATED, green=STORED
8. Confirm red (#E8351A) used only by overlap detector — nowhere else
9. Confirm legend at top, footer at bottom
10. Confirm all abbreviations expanded in node descriptions
11. Confirm no description text cut off (overlap detector shows zero)
12. Confirm traveling dot plays at consistent speed through every node
```

The response after completion must contain only: file path, server URL, one confirmation line. No HTML in the response body at any point.
