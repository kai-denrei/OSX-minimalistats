---
role: pm
owner: Gerald
status: active
last-updated: 2026-06-29
---

# PM

## Scope
Goals, scope, sequencing for the minimal fork.

## Decisions
| Date | Decision | Rationale | Linked roles |
|---|---|---|---|
| 2026-06-29 | Goal: smallest-footprint, minimal menubar monitor across M4 Mini + M5 MBP | Operator's stated direction | [[arch]] [[ux]] |
| 2026-06-29 | Defer Xcode-dependent slimming; document it explicitly | "Step by step"; no-code/no-build wins first | [[devops]] |

## Dead Ends
| Date | What was tried | Why it failed / was rejected |
|---|---|---|

## Lessons

## Open Questions
- [ ] Target footprint number? (operator expected ~20 MB; the `.psd` cut already reached 9.6 MB) — owner: Gerald — since: 2026-06-29
- [ ] How deep to go: delete 4 module targets + source, drop WidgetsExtension, arm64-only, strip localizations? — owner: Gerald — since: 2026-06-29

## Assumptions (brief challenged at init — untested)
- [The 6-module kept set (CPU/RAM/Disk/GPU/Sensors/Battery) is final and won't grow back] — status: untested — since: 2026-06-29
- [Operator will actually *enable/use* GPU/Sensors/Battery, vs. just keeping them available — if not, they're dead weight to cut] — status: untested — since: 2026-06-29
- [One shared repo/config serves both Macs without per-machine divergence] — status: untested — since: 2026-06-29
- [Public visibility is acceptable long-term for this fork (it carries upstream's MIT code)] — status: untested — since: 2026-06-29
- [The deferred footprint work is worth the effort vs. the current 9.6 MB "good enough"] — status: untested — since: 2026-06-29
- [Severing upstream is fine — no intent to pull future Stats fixes/security updates] — status: untested — since: 2026-06-29

## Dependencies
Blocked by: none
Feeds into: all roles

## Session Log
- 2026-06-29 — INIT + first SYNC; brief challenged (6 assumptions surfaced)
