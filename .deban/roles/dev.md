---
role: dev
owner: Gerald
status: active
last-updated: 2026-06-29
---

# Dev

## Scope
Swift changes to the Stats fork: widgets, color logic, module wiring, build.

## Decisions
| Date | Decision | Rationale | Linked roles |
|---|---|---|---|
| 2026-06-29 | Add `.threshold` SColor + `Double.thresholdColor(0.8,0.9)`; wire into `Mini` and `BarChart` | One mechanism = white-by-default + amber@80/red@90 for RAM % and CPU/GPU bars alike | [[ux]] |
| 2026-06-29 | Disk unit-less number via `DiskSize.gigabytesString` + `$capacity.compact` token | `getReadableMemory` always adds units; the template regex absorbs adjacent literals, so a dedicated token is the robust path | [[ux]] |
| 2026-06-29 | Functional module removal = delete from `AppDelegate.modules` + imports | Safe, testable, no pbxproj surgery (deferred) | [[arch]] |

## Dead Ends
| Date | What was tried | Why it failed / was rejected |
|---|---|---|
| 2026-06-29 | Earth-toned Disk via `Disk_bar_chart_color` / `Disk_bar_chart_box` | No effect — the Disk widget's title is "SSD" (config.plist Title override), so keys are `SSD_bar_chart_*`. Per-widget keys are by widget TITLE, not module name. |
| 2026-06-29 | No-code threshold via built-in `.utilization` color | Hardcodes blue (not white) below the low zone and zones aren't user-configurable — needed a custom color fn |

## Lessons
- Stats keys per-widget settings by widget TITLE, not module name; resolve `\(title)_\(type)_…` to the concrete title (Disk's is "SSD"). — from dead end on 2026-06-29
- Arbitrary colors are reachable with no code via `custom:#RRGGBBFF` SColor values (no Xcode). — from 2026-06-29

## Open Questions
- [ ] GPU/RAM threshold escalation visually verified under real load? (CPU verified via stress test only) — owner: Gerald — since: 2026-06-29

## Assumptions
- [staged code compiles] — status: validated (built green, Xcode 26.6) — since: 2026-06-29

## Dependencies
Blocked by: Xcode toolchain (now installed)
Feeds into: [[devops]]

## Session Log
- 2026-06-29 — threshold colors + disk-compact built & live; 4 modules functionally removed; GPU enabled as 4th glyph
