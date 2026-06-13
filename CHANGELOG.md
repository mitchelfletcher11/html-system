# Changelog

## v1.2.0 — 2026-06-13
- First tagged GitHub release.
- Synced `SKILL.md` to the current authoring version.
- **Added `template.html`** — the build scaffold, previously missing from the public repo.
- Corrected install path (`~/.claude/commands/` → `~/.claude/skills/html-system/`).
- Added `SETUP.md` and this changelog.

## v1.1 — 2026-06-01
- Increased default row gap from 170px to **300px** for verbose nodes.
- Added a "verbose node" content-height estimate (190–230px) alongside the original short-description estimate (118px).
- Clarified that the abbreviation-expansion rule produces verbose descriptions by default — 300px gap is the standard.
- Updated branch row minimum from 110px to 150px for verbose descriptions.
- Analogy line color `#4a4a4a` → `#777` for legibility on dark backgrounds.

## v1.0 — initial release
- Full SVG pipeline diagram with animated traveling dot.
- Semantic three-color system (purple / gold / green).
- Safe foreignObject height calculation with column-pair overlap detection.
- Branch animation with parallel split dots.
- SVG text overlap detector embedded in every output.
- Stats grid, header KPI chips, legend, footer.
