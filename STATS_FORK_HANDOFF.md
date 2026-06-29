# Handoff — Diegetic Minimalist Fork of `exelban/stats`

**Audience:** a Claude CLI agent operating locally on macOS with Xcode installed.
**Upstream:** https://github.com/exelban/stats (MIT).
**Verified against:** repo `master` as of clone date below. File paths and defaults in this doc were confirmed by reading the source — trust them, but re-check line numbers since they drift.

---

## 0. Goal in one paragraph

Fork Stats into a stripped, label-free menubar monitor showing exactly three metrics — CPU, RAM, Disk — where each metric is identified by its **visual form, not by text**. No "CPU"/"RAM"/"GB" labels in the bar. Identity is *diegetic*: the CPU widget reads as a CPU because of its shape, RAM because it fills toward a fixed ceiling, Disk because it barely moves. Clicking any of the three still opens the full detailed popup (untouched upstream behavior). Everything else — GPU, Network, Sensors, Bluetooth, Battery, Clock, Remote, all non-English localizations — is removed or disabled.

The design principle: **encode identity through form, subtract all text.**

- **CPU** → volatility + multiplicity. A sparkline (`line_chart`) or per-core bars (`bar_chart`). N segments reads as "the processor." This is the jagged one.
- **RAM** → a bounded reservoir against a known ceiling. A fill that visibly bottoms out at a hard wall (16/32/64 GB) reads as "memory." **This widget does not exist upstream and must be built** (see §4).
- **Disk** → near-static. A thin sliver or bar that barely moves reads as "storage." A time-series here is a flat line and is forbidden.

Three forms — jagged / filling-tank / static-sliver — zero text.

---

## 1. Prerequisites

- macOS 11+, Xcode 16+ with command-line tools.
- A free Apple ID is sufficient for **local** signing (automatic signing, "Run" from Xcode). Do **not** attempt notarization or Developer-ID distribution; this fork is for personal local use.
- Do **not** install or build the SMC privileged helper (`SMC/Helper`). It is only needed for temperature/fan sensors, which are out of scope. Skipping it removes the single gnarliest part of the build.

---

## 2. Repo orientation (confirmed file map)

Edit surface is small. The whole job touches maybe six files plus one new file.

| Concern | File | Notes |
|---|---|---|
| Menubar renderers | `Kit/Widgets/*.swift` | `LineChart`, `BarChart`, `Tachometer` (gauge), `Mini`, `PieChart`, `Memory`, `Dot`, `Text`. |
| Sparkline widget | `Kit/Widgets/LineChart.swift` | Label is `labelState` (**default `false`**); value text is `valueState` (**default `false`**). The label, when on, is `String(self.title.prefix(3)).uppercased()` drawn as stacked vertical chars (~line 131). A bare no-text sparkline is already the default. |
| Gauge widget (reservoir base) | `Kit/Widgets/Tachometer.swift` | Model the new Reservoir widget on this. It conforms to `WidgetWrapper`. |
| Global colors | `Kit/helpers.swift` | The `SColor` enum lives here. Add custom accent colors here. |
| Global widget metrics | `Kit/constants.swift` | `Constants.Widget` — menubar `height` is `22`, plus `margin`/`spacing`. |
| Per-module data | `Modules/<X>/readers.swift` | RAM exposes `totalSize` (`stats.max_mem`, ~line 16/29). CPU exposes per-core e/p/super clusters via `SystemKit.shared.device.info.cpu?.cores` (~line 39, 132+). The data the diegetic design needs is already collected. |
| Per-module widget offering | `Modules/<X>/widget.swift` + `Modules/<X>/config.plist` | `config.plist` has a `Widgets` dict whose keys are the available widget types. |
| Detailed-on-click view | `Modules/<X>/popup.swift` | **Do not touch.** This is the expanded dropdown; it is decoupled from the menubar widget and works as-is. |

**Available widget-type keys per module (from `config.plist`):**
- CPU: `label, mini, line_chart, bar_chart, pie_chart, tachometer`
- RAM: `label, mini, line_chart, bar_chart, pie_chart, memory, tachometer, text, state`
- Disk: `label, mini, bar_chart, pie_chart, memory, speed, network_chart, text`

Modules are **separate Xcode targets** (each has its own `Info.plist`). Disabling a module in-app stops it running; fully removing it means deleting its target from `Stats.xcodeproj`.

---

## 3. Build & run — the loop to use (READ BEFORE BUILDING)

**Do NOT run `make build`.** The Makefile `build` target is the full release pipeline (`clean next-version archive notarize sign verify prepare-dmg prepare-dSYM open`) and will fail without Developer-ID + notarization credentials.

Local dev loop instead:

```bash
# open and run interactively
open Stats.xcodeproj      # select the "Stats" scheme, then ⌘R

# or headless debug build
xcodebuild -project Stats.xcodeproj -scheme Stats -configuration Debug build
```

If automatic signing complains, set the Stats target to your personal team with automatic signing, Debug configuration. The SMC helper target should be **excluded from the scheme** (or just don't build it).

Iterate: edit → ⌘R → look at the menubar → adjust. There is no faster feedback than the running app; the widgets are pixel-tuned at 22pt height, so visual inspection is mandatory after each change.

---

## 4. Task sequence

### Phase 0 — Baseline
1. `git clone https://github.com/exelban/stats.git && cd stats`
2. Create a branch: `git checkout -b diegetic-minimal`
3. Build and run unmodified (§3). Confirm it launches and shows widgets. This is your rollback reference.

### Phase 1 — Strip
4. **Disable unwanted modules first (no-code):** run the app, in Settings turn OFF GPU, Network, Sensors, Bluetooth, Battery, Clock, Remote. Confirm only CPU/RAM/Disk remain. Disabling Sensors + Bluetooth also removes the heaviest sampling cost.
5. **Remove modules from the build (code):** in `Stats.xcodeproj`, remove the targets/groups for the modules disabled above. Build after each removal; if a removal breaks the build (shared symbol), revert that one removal and leave the module present-but-disabled. **Stop at the first module whose removal can't be cleanly resolved in two attempts** — disabled-but-present is an acceptable end state.
6. **Localization (do this LAST, lowest value):** delete every `*.lproj` under `Stats/Supporting Files/` except `en.lproj` (and the `*.xcloc` source-localization scaffolding if present), then remove the dropped regions from the project's `knownRegions`. This only shrinks the bundle; it changes no runtime behavior. If it introduces build friction, abandon it — it is not worth a debugging session.

### Phase 2 — Diegetic config (no-code wins)
7. CPU: set menubar widget to `line_chart` **or** `bar_chart`. Ensure label OFF and value OFF (defaults). For the strongest "this is a CPU" read, prefer `bar_chart` (per-core bars = unmistakable). For subtlety, `line_chart`.
8. Disk: set menubar widget to `bar_chart` or `mini`. **Never `line_chart`** for disk. Keep it visually quiet.
9. RAM: temporarily set to `tachometer` to preview a gauge feel, but the real target is the new Reservoir widget in Phase 3.

### Phase 3 — Build the Reservoir widget (the one new component)
This is the only net-new code. It is a fixed-ceiling fill: `used / totalSize`, where `totalSize` is already provided by `Modules/RAM/readers.swift`. The fixed ceiling **is** the diegesis — you never print "64GB"; the fill simply bottoms out at the wall.

10. Read `Kit/Widgets/Tachometer.swift` fully to learn the real `WidgetWrapper` conformance (init signature, `draw(_:)`, how it receives values, how it registers). **Mirror that API exactly** — the skeleton below is illustrative intent, not guaranteed-compiling code.

```swift
// Kit/Widgets/Reservoir.swift  — ILLUSTRATIVE; conform to the real WidgetWrapper API in Tachometer.swift
import Cocoa

public class Reservoir: WidgetWrapper {
    private var value: Double = 0          // 0...1, = used / total
    private var colorState: SColor = .systemAccent
    private let chartWidth: CGFloat = 8    // narrow; this is the "tank"

    // init(...) — copy Tachometer's signature & super.init(.reservoir, ...)

    public override func draw(_ dirtyRect: NSRect) {
        super.draw(dirtyRect)
        guard let ctx = NSGraphicsContext.current?.cgContext else { return }
        let h = self.frame.height
        let w = self.chartWidth
        // hard ceiling frame (the wall the fill bottoms out against)
        ctx.setStrokeColor(NSColor.tertiaryLabelColor.cgColor)
        ctx.stroke(CGRect(x: 1, y: 1, width: w, height: h-2), width: 1)
        // fill from bottom up to `value`
        let fillH = max(0, min(h-2, (h-2) * CGFloat(self.value)))
        ctx.setFillColor(self.color(for: self.value).cgColor)
        ctx.fill(CGRect(x: 1.5, y: 1, width: w-1, height: fillH))
    }

    private func color(for v: Double) -> NSColor { /* map SColor / pressure */ NSColor.controlAccentColor }

    // setValue(_:) — accept used/total from the RAM reader, store self.value, setNeedsDisplay
}
```

11. **Register the widget type.** Find the `widget_t` enum (grep the codebase for the existing `.tachometer` / `.lineChart` cases — likely in `Kit/`), add a `.reservoir` case with raw value `"reservoir"`.
12. **Offer it from RAM.** Add `reservoir` to the `Widgets` dict in `Modules/RAM/config.plist`, and instantiate it in `Modules/RAM/widget.swift` alongside the existing cases.
13. **Wire data.** In `Modules/RAM/widget.swift` (or wherever the RAM module pushes values to its active widget), pass `used / totalSize` to the Reservoir's setter. `totalSize` already exists in `Modules/RAM/readers.swift`.
14. Build, run, set RAM's menubar widget to Reservoir, verify the fill tracks memory and bottoms out at a stable wall.

### Phase 4 — Aesthetic pass
15. In `Kit/helpers.swift`, add accent colors to the `SColor` enum (amber + teal in OKLCH-derived sRGB values; convert OKLCH → sRGB at author time, store as `NSColor(red:green:blue:alpha:)`). Wire these as the default `colorState` for the three widgets.
16. In `Kit/constants.swift`, tune `Constants.Widget` spacing/margins only if the three glyphs feel cramped. Leave `height` at 22.
17. Optional: monochrome mode for true subtlety — Stats already supports a `.monochrome` `SColor` case; consider it for Disk so it recedes.

---

## 5. Dead Ends & gotchas (highest-value section — read twice)

- **`make build` is a trap.** It is the signed-release pipeline and will fail locally. Use Xcode ⌘R or `xcodebuild ... -configuration Debug`. (Restated because it's the #1 time-sink.)
- **Don't build the SMC helper.** `SMC/Helper` is a privileged tool for sensors you're not using; building/installing it invites code-signing and privilege-escalation friction for zero benefit. Exclude it from the scheme.
- **Disk sparkline is forbidden by design, not preference.** Capacity changes on the scale of hours; a `line_chart` of it is a flat, dead line. If tempted, stop — use a bar or `mini`.
- **RAM "used" is misleading raw.** macOS fills RAM with reclaimable cache, so raw used pins high. The Reservoir should fill on a meaningful basis (used vs total is acceptable for a diegetic gauge; if it reads as "always full," switch the fill basis to memory *pressure*, which the RAM reader also exposes). Verify against Activity Monitor before declaring done.
- **Menubar item order is controlled by macOS, not the app.** Order may shift after the first reboot post-install; reposition with ⌘-drag. Don't chase this in code.
- **Known upstream quirk:** the CPU menubar widget occasionally fails to render data on cold boot and needs the module toggled off/on once. If you see an empty CPU glyph after a fresh launch, that's this, not your change.
- **Label is `title.prefix(3)`, not a localized string.** If you ever see "CPU"/"RAM" reappear, it's `labelState` flipping true, not a localization issue — check the widget's stored `_label` default key.
- **Module removal can cascade.** Modules share symbols via `Kit`. If removing a target breaks the build and two fix attempts don't resolve it, leave the module present-but-disabled. Disabled modules don't sample, so the cost of leaving one in is bundle size only.
- **Localization stripping is cosmetic.** It does not improve runtime or memory. If it costs you a debugging session, it has already failed its cost/benefit; revert and move on.
- **Don't refactor `popup.swift`.** The detailed-on-click views are the part you want to keep intact. Touch only the menubar widget layer.

---

## 6. Acceptance criteria

Done when:
1. Menubar shows exactly three glyphs: a jagged CPU form, a filling RAM reservoir, a near-static Disk indicator — **with no text labels.**
2. Clicking each opens its full detailed popup, unchanged.
3. Only CPU/RAM/Disk modules are active; GPU/Net/Sensors/Bluetooth/Battery/Clock/Remote are off or removed.
4. RAM reservoir fill tracks real memory and visibly bottoms out at a fixed ceiling; verified against Activity Monitor.
5. Builds and runs via Xcode ⌘R with no SMC helper, no notarization.
6. Accent colors are the custom amber/teal (or chosen palette), applied to the three widgets.

Commit in small steps (one phase per commit) so any regression is bisectable. Leave the branch as `diegetic-minimal`; do not force-push over `master`.
