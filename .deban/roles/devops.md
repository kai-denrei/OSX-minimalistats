---
role: devops
owner: Gerald
status: active
last-updated: 2026-06-29
---

# DevOps

## Scope
Build, signing, git history, repo publishing, footprint.

## Decisions
| Date | Decision | Rationale | Linked roles |
|---|---|---|---|
| 2026-06-29 | Ad-hoc Debug build (`CODE_SIGN_IDENTITY="-"`, no team) | No Apple Developer team; ad-hoc runs on Apple Silicon; never `make build` (signed-release pipeline, fails locally) | [[dev]] |
| 2026-06-29 | Flatten history to a single `main` commit; remove origin + 343 tags | Operator wants a minimal repo; upstream recoverable from GitHub if needed | [[arch]] |
| 2026-06-29 | Public repo `kai-denrei/OSX-minimalistats` | MIT fork, simplest to clone onto the M5 | [[pm]] |
| 2026-06-29 | Delete 16 MB of design `.psd` files; amend + prune + force-push | Unreferenced design source — the real repo bloat. Folder 38→9.6 MB, `.git` 15→3 MB. | |

## Dead Ends
| Date | What was tried | Why it failed / was rejected |
|---|---|---|
| 2026-06-29 | Expecting history-purge alone to reach ~20 MB | Purge took `.git` 61→15 MB but total stayed ~38 MB — the working tree carried 16 MB of `.psd` mockups. Measure the TREE, not just `.git`. |
| 2026-06-29 | `git gc --prune=now --aggressive` after amending away the `.psd` | Didn't reclaim — the orphaned 14 MB blob stayed reachable via reflog/ORIG_HEAD. `.git` sat at 15 MB. |
| 2026-06-29 | `sudo xcode-select` run by the agent | Needs admin password; operator ran the 3 sudo setup commands manually |

## Lessons
- To actually shrink `.git` after removing a big file: rewrite the commit (amend), then `git reflog expire --expire=now --expire-unreachable=now --all`, `rm .git/ORIG_HEAD`, `git repack -ad`, `git prune --expire=now`. Plain `gc --prune=now` leaves reflog-/ORIG_HEAD-reachable blobs. — from dead end on 2026-06-29
- Repo size = working tree + `.git`; purging history only touches `.git`. Large unreferenced files in the tree must be deleted AND pruned from history to reclaim. — from 2026-06-29
- Ad-hoc signing (`CODE_SIGN_IDENTITY="-"`) is enough to build + run locally on Apple Silicon without an Apple team. — from 2026-06-29
- At ~10 MB the repo is dominated by NON-code: app-icon PNGs 2.3 MB (incl. a 246 KB identical dup `icon_512x512 1.png` + a 984 KB 1024px icon), a vendored LevelDB lib (`Kit/lldb/libleveldb.a`, 832 KB, backs chart-history persistence; 4 pbxproj refs), and 51 localizations (~1.4 MB). Actual Swift source is only ~2 MB. Further cuts, easiest→hardest: dup icon (free), localizations (~1.4 MB, build-friction risk), 1024px icon (~700 KB), then LevelDB (needs ripping out history persistence). Practical floor ~3–4 MB. — from 2026-06-29

## Open Questions
- [ ] Replicate the exact menubar config onto the M5 — Stats Settings export/import vs. baking code defaults? — owner: Gerald — since: 2026-06-29

## Assumptions
- [force-push safe] — status: validated (fresh solo repo, no collaborators/clones) — since: 2026-06-29

## Dependencies
Blocked by: Xcode (installed)
Feeds into: footprint backlog (TODO.md)

## Session Log
- 2026-06-29 (sync) — size deep-dive: heavy dep = vendored LevelDB (832 KB); icon 2.3 MB + 51 localizations dominate; code only ~2 MB
- 2026-06-29 — built; flattened to single commit; published public repo; cut 16 MB .psd (folder → 9.6 MB)
