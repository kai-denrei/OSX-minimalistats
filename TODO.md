# TODO — Minimalist Stats Fork

Living task doc. Companion to `STATS_FORK_HANDOFF.md` (the detailed design brief).
Goal: a minimalist, mostly label-free set of menubar stats — CPU, RAM, Disk — identified by visual form.

---

## ✅ Done

- [x] Cloned `exelban/stats` into this dir — on branch `master` @ **v3.0.5** (full history; handoff doc preserved untracked).
- [x] Verified handoff file map against live source — all claimed files exist; `widget_t` enum confirmed at `Kit/module/widget.swift:14`.
- [x] Installed **vanilla Stats v3.0.5** (official notarized DMG, byte-for-byte the same release as our source) to `/Applications` and launched it — running in the menubar now. This is our rollback reference.

---

## ⛔ Blocker

- [ ] **Xcode is not installed** (only Command Line Tools). Required to *build* any source change.
      Building `Stats.xcodeproj` needs full **Xcode 16+** (~15 GB, free, needs your Apple ID).
      Until then we can only configure the *installed* app — which covers a surprising amount (see below).

---

## Key insight: no-code vs. code

Two of the handoff's four phases need **zero Xcode** — they're just app Settings (or `defaults write`):

| Work | Needs Xcode? | How |
|---|---|---|
| Disable GPU/Net/Sensors/BT/Battery/Clock/Remote | ❌ No | Settings → toggle modules off |
| Set CPU → bar/line chart, Disk → bar/mini, RAM → tachometer | ❌ No | Settings → each module → Widget picker |
| Turn off text labels / value text | ❌ No | Already default-off on charts; toggle in Widget settings |
| **New "Reservoir" RAM widget** | ✅ Yes | New file `Kit/Widgets/Reservoir.swift` + enum + config |
| **Custom accent colors (amber/teal)** | ✅ Yes | Edit `SColor` in `Kit/helpers.swift` |

So we can reach ~70% of the minimalist look **right now**, no build required, and reserve Xcode for the two net-new bits.

---

## Phase 0 — Baseline ✅ (done above)

## Phase 1 — Strip to CPU / RAM / Disk  *(no-code)* ✅
- [x] Disabled GPU, Network, Sensors, Bluetooth, Battery, Clock, Remote — via `defaults write eu.exelban.Stats <Name>_state -bool false`, applied on quit→relaunch (Store caches the domain at launch).
- [x] CPU/RAM/Disk left enabled. Verified keys persist across relaunch.
- [x] **Functionally removed Net / Bluetooth / Clock / Remote** from the app (`Stats/AppDelegate.swift` `modules` array + imports). They no longer load/instantiate. Built + ran clean with CPU/RAM/Disk/GPU/Sensors/Battery. Their code/frameworks still ship — see "Deferred footprint reduction" below.

## Phase 2 — Diegetic widget config  *(no-code)* ✅ (first pass)
- [x] CPU → `bar_chart` **per-core** (10 bars: 4 P + 6 E on this M4). `CPU_widget = "bar_chart"`, `CPU_usagePerCore = true`, `CPU_bar_chart_color = "cluster"` (P vs E distinct colors). Alt: `CPU_clustersGroup = true` → just 2 bars (E/P clusters).
- [x] Disk → `bar_chart` (quiet bar). `Disk_widget = "bar_chart"`.
- [x] RAM → `tachometer` (gauge preview). `RAM_widget = "tachometer"`.
- [ ] Confirm visually & decide final per-widget choices (see visual-ideas section). Charts default label/value OFF; toggle if any text shows.

### How to reconfigure live (recipe)
```bash
osascript -e 'quit app "Stats"'; sleep 2; pkill -x Stats
defaults write eu.exelban.Stats <Module>_widget -string "<rawValue>"   # mini|line_chart|bar_chart|pie_chart|tachometer|memory|...
defaults write eu.exelban.Stats <Name>_state -bool true|false
open -a /Applications/Stats.app
```
Module Names: CPU, RAM, Disk, GPU, Network, Battery, Bluetooth, Clock, Remote, Sensors.

## Phase 3 — New Reservoir RAM widget  *(needs Xcode)*
- [ ] Read `Kit/Widgets/Tachometer.swift` fully; mirror its `WidgetWrapper` API.
- [ ] Add `Kit/Widgets/Reservoir.swift` — fixed-ceiling fill = `used / totalSize` (from `Modules/RAM/readers.swift`).
- [ ] Register `.reservoir` in `widget_t` (`Kit/module/widget.swift`).
- [ ] Offer it in `Modules/RAM/config.plist` + instantiate in `Modules/RAM/widget.swift`; wire data.
- [ ] Verify fill tracks real memory & bottoms out at a fixed wall (cross-check Activity Monitor — note: raw "used" reads high due to cache; may switch fill basis to memory *pressure*).

## Phase 4 — Aesthetic pass  *(needs Xcode)*
- [ ] Add amber + teal accents to `SColor` in `Kit/helpers.swift` (OKLCH→sRGB at author time); set as widget defaults.
- [ ] Tune `Constants.Widget` spacing/margins only if cramped; keep `height = 22`.
- [ ] Optional: monochrome Disk so it recedes.

---

## ⭐ OUR visual ideas — "slightly different visuals" (TO FILL IN)

> You mentioned the screenshots gave you ideas for visuals that differ from the handoff's
> strict diegetic concept. Capture them here as we discuss — e.g.:
>
**Confirmed by user (2026-06-29):**
- [x] Layout confirmed: **column · gauge · column**, no text. CPU = 10 per-core bars (keep).
- [x] **CPU bars:** removed the **white background** (it was the box fill — white in dark mode). `CPU_bar_chart_box = false`.
- [x] **RAM gauge:** **earth tones**; orange→ochre, pink/red→terracotta. (3 segments recolored.)
- [x] **Disk bar:** earth-toned (clay); box off. ⚠️ GOTCHA: Disk widget keys use title **`SSD`** (from `Modules/Disk/config.plist` `Widgets.bar_chart.Title=SSD`), NOT `Disk`. So per-widget keys are `SSD_bar_chart_color` / `SSD_bar_chart_box`, while module-level keys (`Disk_state`, `Disk_widget`) use `Disk`. Module-name keys ≠ widget-title keys.
- [ ] **Disk dark-grey background** → ⚠️ NEEDS CODE (box fill is hardcoded white/black; no `defaults` key). Pending Xcode (Phase 4).

**DESIGN PIVOT (2026-06-29, "RAM is the real bottleneck"):**
- [x] **CPU** — keep as-is (10 per-core bars, earth tones).
- [x] **RAM → numeric %** (not a gauge). Now `mini` widget, white (`monochrome`), no label → shows "33%". `RAM_widget=mini`, `RAM_mini_color=monochrome`, `RAM_mini_label=false`.
- [x] **Disk → text** used/total. `Disk_widget=text`, `Disk_textWidgetValue=$capacity.used/$capacity.total`.
- [x] ✅ BUILT & LIVE (Xcode 26.6, ad-hoc signed, installed over `/Applications/Stats.app`): **RAM threshold color** — `.threshold` SColor + `Double.thresholdColor(0.8,0.9)`, in `Mini`. `RAM_mini_color=threshold`.
- [x] ✅ BUILT & LIVE: **CPU bars white + per-core escalation** — `.threshold` added to `BarChart` too (`partitionValue.value.thresholdColor()`). `CPU_bar_chart_color=threshold` → each core white<80 / orange 80–90 / red ≥90.
- [x] ✅ BUILT & LIVE: **Disk "123/245GB"** — `DiskSize.gigabytesString` + `$capacity.compact` token (integer, one GB, no decimals). `Disk_textWidgetValue=$capacity.compact`.
- [ ] Disk dark-grey background — DROPPED for now (disk is clean text; user chose transparent/minimal).

## Build recipe (works)
```bash
xcodebuild -project Stats.xcodeproj -scheme Stats -configuration Debug \
  -derivedDataPath <scratch>/DerivedData \
  CODE_SIGN_IDENTITY="-" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=YES DEVELOPMENT_TEAM="" build
# then: cp -R <derived>/Build/Products/Debug/Stats.app /Applications/  (quit Stats first)
```
Ad-hoc signing disables hardened runtime (fine for local). Build products kept in scratchpad, NOT the repo.
- [x] ~~Phase 3 Reservoir tank widget~~ — **SUPERSEDED**: user prefers numeric % over a fill gauge for RAM. Drop unless revisited.

### Earth-tone palette (applied, no-code via `custom:#hex`)
Key discovery: SColor supports `custom:#RRGGBBFF` values via `defaults` — **no Xcode needed for arbitrary colors** (`SColor.custom`, `Kit/types.swift:213`; parsed by `NSColor(hex:)`, `Kit/extensions.swift:495`).
| Name | Hex | Used for |
|---|---|---|
| Sage | `#7D8471` | CPU E-cores (6), RAM app segment |
| Ochre | `#B08D57` | CPU P-cores (4), RAM wired segment (was orange) |
| Terracotta | `#A56A4F` | RAM compressed segment (was pink/red) |
| Clay | `#9C7B5B` | Disk bar |

Keys: `CPU_eCoresColor`, `CPU_pCoresColor`, `RAM_appColor`, `RAM_wiredColor`, `RAM_compressedColor`, `Disk_bar_chart_color` (all `custom:#…FF`). Tune any with `defaults write eu.exelban.Stats <key> -string "custom:#RRGGBBFF"` then quit→relaunch.

> Note: RAM gauge's faint grey "empty" arc is hardcoded (`Charts.swift:1060`) — not removable no-code.

**Still to discuss:**
> - [ ] (gauge style — keep tachometer dial, or move to the fill/"tank" Reservoir sooner?)
> - [ ] (spacing / how minimal — icons or pure glyphs?)
> - [ ] (light/dark adaptiveness)

---

## Size notes
- Installed **app = 17 MB** (small). Breakdown: `Frameworks` 11 MB (`Kit` 3.7 MB + 10 per-module frameworks ~6.7 MB) · `Assets.car` 1.8 MB · `WidgetsExtension.appex` 1.3 MB. Binary is **universal (arm64+x86_64)**.
- Source **repo folder = 84 MB**, of which **`.git` history = 61 MB** (this is what "feels big" in Finder, not the app).
- Future slimming (all Xcode-gated): remove 7 unused module frameworks (Net/Sensors/GPU/Remote/Clock/Bluetooth/Battery ≈ 5 MB) + thin to arm64-only ≈ halve binaries → app could drop to ~8–10 MB. Repo: a shallow clone (`--depth 1`) avoids the 61 MB history if we don't need it.

## Deferred footprint reduction (NOT done yet — goal: smallest possible footprint, step by step)
Functional removal is done (the 4 modules don't load), but their **code + frameworks still ship**. To actually shrink, in rough order of value/safety:
- [ ] **Delete the 4 module targets + source** (Net, Bluetooth, Clock, Remote) from `Stats.xcodeproj`. Tooling ready: `xcodeproj` gem installed at `~/.gem` (Ruby 2.6). ⚠️ `Net` is `import`ed by the **WidgetsExtension** (`Widgets/widgets.swift:18`) — must de-couple it there or drop the extension first.
- [ ] **Drop the WidgetsExtension** (macOS desktop / Notification-Center widgets — a separate surface from the menubar; ~1.3 MB). Removes the Net coupling automatically. *(User chose "keep for now"; revisit when going for minimum size.)*
- [x] ~~Thin universal → arm64-only~~ — **N/A: already arm64-only.** Debug builds default to `ONLY_ACTIVE_ARCH=YES`; every binary verified arm64 (no x86_64 slice). Real app-shrink levers instead: a **Release build** (strips Debug symbols, ~19→~11 MB) and **removing the 4 dead module frameworks** (Net/Remote/Clock/Bluetooth ~3.7 MB, still embedded after functional-only removal).
- [ ] **Remove SMC + Helper targets** — the privileged fan-control helper we never build. Fan/temperature *monitoring* (Sensors) works without it; only fan *control writes* need it.
- [x] **Strip localizations** — DONE (commit 582f3c9): en-only; removed 39 project refs + ~49 `.lproj` + the `.xcloc`; `knownRegions=en`; build green (no storyboards/Base, so safe). Also dropped the 1024px + 2 dup app icons. Repo 9.7→7.4 MB.
- [ ] **Re-evaluate GPU/Sensors/Battery** if never enabled — they're off by default (Battery also self-disables on the Mini). Kept for now per user.

Target: app **~17 MB → ~8 MB**, far fewer moving parts. Each step is: edit → build → run → commit (so any regression is bisectable).

## Build & run reminders (from handoff)
- **Never `make build`** — that's the signed-release pipeline; it will fail locally.
- Build via Xcode ⌘R, or `xcodebuild -project Stats.xcodeproj -scheme Stats -configuration Debug build`.
- **Don't build the SMC helper** (`SMC/Helper`) — sensors are out of scope; exclude it from the scheme.
- Work on a branch (`diegetic-minimal` or similar); commit one phase at a time; don't force-push `master`.
