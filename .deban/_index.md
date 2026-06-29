---
project: OSX-minimalistats
created: 2026-06-29
status: active
mode: solo
stale_threshold_days: 30
---

# OSX-minimalistats — Index

## Brief
A minimal, label-free macOS menubar monitor forked from exelban/stats (MIT). Three glyphs —
CPU per-core bars, RAM %, Disk used/total — no text labels, unified threshold coloring
(white <80% / amber 80–90% / red ≥90%). Trimmed of unneeded modules and published as a public
repo (kai-denrei/OSX-minimalistats) to share across the operator's Mac Mini M4 and MacBook Pro M5.

## Active Roles
- [[dev]] — owner: Gerald
- [[arch]] — owner: Gerald
- [[ux]] — owner: Gerald
- [[devops]] — owner: Gerald
- [[pm]] — owner: Gerald

## Key Decisions
- Unified threshold coloring (white/amber/red @80/90) — pivoted from an earth-tone palette — [[ux]]
- Functional module removal now; full target/source deletion deferred & documented — [[arch]]
- Flatten git history to one commit, sever upstream (recoverable via GitHub) — [[devops]]
- Repo bloat was 16 MB of design `.psd` files in the tree, not git history — [[devops]]

## Open Questions (cross-role)
- Will macOS hold menubar order CPU·RAM·GPU·Disk, or is a one-time ⌘-drag required? — [[ux]] [[devops]]
- How to replicate the exact menubar config onto the M5 (Stats export/import vs. code defaults)? — [[devops]]
- How deep to go on footprint (delete 4 module targets, drop WidgetsExtension, arm64-only, strip localizations)? — [[pm]] [[arch]]
