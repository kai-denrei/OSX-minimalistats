# Minimal Stats — a label-free macOS menubar monitor

A stripped-down fork of [exelban/stats](https://github.com/exelban/stats). Three glyphs in the
menubar, **no text labels**, unified threshold coloring:

| Glyph | Shows | Form |
|---|---|---|
| **CPU** | per-core load | bars (10 on an M4) |
| **RAM** | usage | a numeric `%` |
| **Disk** | used / total | compact text, e.g. `13/245GB` |

All three share one **threshold palette**: low‑key **white** under 80%, **amber** at 80–90%,
**red** at ≥90%. Clicking any glyph opens the full detailed popup (unchanged from upstream).

GPU, Sensors, and Battery modules are bundled but **off by default** (enable in Settings;
Battery self‑disables on machines without a battery). The Network, Bluetooth, Clock, and
Remote modules have been removed from the app.

## Build & run

Requires **Xcode 16+**. This fork is for **personal / local use** (ad‑hoc signed, no
notarization):

```bash
xcodebuild -project Stats.xcodeproj -scheme Stats -configuration Debug \
  CODE_SIGN_IDENTITY="-" CODE_SIGNING_REQUIRED=NO CODE_SIGNING_ALLOWED=YES DEVELOPMENT_TEAM="" build
# then run the built Stats.app (Build/Products/Debug/Stats.app)
```

Do **not** run `make build` — that's the upstream signed‑release pipeline and will fail locally.
The privileged SMC helper is intentionally not built (only fan *control* needs it; fan/temperature
*monitoring* works without it). See `TODO.md` for the design log and deferred footprint‑reduction work.

## Credits & license

Fork of **[exelban/stats](https://github.com/exelban/stats)** by **Serhiy Mytrovtsiy**, used under
the **MIT License** — see [`LICENSE`](LICENSE); all upstream copyright is retained. This fork trims
the app to a minimal menubar set and adds threshold‑based widget coloring.
