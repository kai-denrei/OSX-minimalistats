---
role: arch
owner: Gerald
status: active
last-updated: 2026-06-29
---

# Arch

## Scope
Module set, project structure, what stays vs. goes in the minimal fork.

## Decisions
| Date | Decision | Rationale | Linked roles |
|---|---|---|---|
| 2026-06-29 | Keep CPU/RAM/Disk/GPU/Sensors/Battery; drop Net/Bluetooth/Clock/Remote | Itemized each module vs. the operator's two Macs; dropped cloud (Remote), redundant (Clock), and low-value (Net/Bluetooth) | [[pm]] |
| 2026-06-29 | Functional removal now; full target+source removal deferred & documented in TODO.md | "Step by step, test everything works"; full removal needs pbxproj edits + WidgetsExtension de-coupling from Net | [[dev]] [[devops]] |
| 2026-06-29 | Keep Sensors despite the "unmaintained fan control" worry | Read-only temp/power/RPM monitoring works WITHOUT the privileged SMC helper; only fan-control writes need it | |

## Dead Ends
| Date | What was tried | Why it failed / was rejected |
|---|---|---|
| 2026-06-29 | Workflow agent tasked with mapping BarChart background controls (heavy schema) | Hit the StructuredOutput retry cap (failed); recovered by reading the source directly. Heavy schemas can starve a single agent. |

## Lessons
- Module "removal" has two depths: functional (drop from the `modules` array — safe, no build risk) vs. structural (delete the Xcode target+source — needs pbxproj surgery). Do functional first, verify, then structural. — from 2026-06-29
- Sensors' value (temps/power/fan RPM) is independent of the unmaintained fan-control helper; don't conflate "fan control unmaintained" with "sensor monitoring unavailable". — from 2026-06-29

## Open Questions
- [ ] Drop the WidgetsExtension entirely for menubar-only minimalism? It's the only thing coupling to Net. — owner: Gerald — since: 2026-06-29

## Assumptions
- [GPU module covers both M4 and M5 GPUs] — status: validated (ANE power tables include m4 + m5) — since: 2026-06-29
- [The 6-module kept set is final] — status: untested — since: 2026-06-29

## Dependencies
Blocked by: none
Feeds into: [[devops]] footprint reduction

## Session Log
- 2026-06-29 — itemized 7 modules; dropped 4 (functional); kept 6
