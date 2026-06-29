---
role: ux
owner: Gerald
status: active
last-updated: 2026-06-29
---

# UX

## Scope
Menubar visual design: glyph forms, color, layout, labels.

## Decisions
| Date | Decision | Rationale | Linked roles |
|---|---|---|---|
| 2026-06-29 | Three glyphs, no text labels: CPU per-core bars, RAM numeric %, Disk used/total | Minimalist; identity by form + position | [[pm]] |
| 2026-06-29 | Unified threshold palette: white <80% / amber 80–90% / red ≥90% | One calm baseline + escalation reads instantly; replaced earlier earth-tone scheme | [[dev]] |
| 2026-06-29 | RAM as the prominent numeric; CPU/GPU as bar clusters; Disk as quiet text | Operator's framing that RAM is the usual bottleneck | |
| 2026-06-29 | GPU as a single white bar, ordered CPU·RAM·GPU·Disk | GPU exposes only overall load (not per-core), so one bar — distinct from CPU's ten | [[dev]] |

## Dead Ends
| Date | What was tried | Why it failed / was rejected |
|---|---|---|
| 2026-06-29 | Earth-tone palette (sage/ochre/terracotta/clay) applied to RAM gauge + bars | Superseded by the numeric-% + threshold direction once RAM was reframed as the bottleneck. Worked fine; dropped by choice, not failure. |
| 2026-06-29 | RAM as a tachometer gauge; the planned vertical "Reservoir" fill widget | Dropped — operator prefers a numeric % to a gauge/fill |
| 2026-06-29 | Goal of making GPU "indistinguishable from CPU" (10 white bars) | Impossible from the data — GPU reports a single overall load, not per-core. Settled on 1 bar, which is actually clearer. |

## Lessons
- Settle the core metric framing before investing in a palette — the earth-tone work was redone once RAM became "the bottleneck" and the scheme shifted to thresholds. — from 2026-06-29
- Let the data shape the glyph: don't force a form (10 bars) the source can't feed; a single GPU bar is honest and more legible. — from 2026-06-29

## Open Questions
- [ ] Will macOS keep menubar order CPU·RAM·GPU·Disk, or does it need a one-time ⌘-drag? (separate NSStatusItems are macOS-ordered) — owner: Gerald — since: 2026-06-29
- [ ] Offer GPU as 3 bars (overall/render/tiler) for closer CPU parity? — owner: Gerald — since: 2026-06-29

## Assumptions
- [threshold escalation fires correctly] — status: untested for GPU/RAM (CPU stress-tested only) — since: 2026-06-29

## Dependencies
Blocked by: none
Feeds into: [[dev]]

## Session Log
- 2026-06-29 — earth-tone → threshold pivot; numeric RAM; GPU added as 4th glyph
