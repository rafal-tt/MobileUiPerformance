# Problem statement

## Product context

ThinkTime’s mobile client is on **.NET MAUI 10**. See [findings/top-issues.md](findings/top-issues.md) for the canonical list of product issues.

## Top issues

1. **Long UI creation time** — complex page open is too slow; dominated by handler creation and initial layout.
2. **Non-fluent list scrolling** — same cause: creating UI for new cells exceeds the frame budget.
3. **Rich text display** — `WebView` does not adapt size to content well; product workarounds exist but are imperfect.
4. **Hard-to-locate memory leaks** — every platform view has a native peer; unclear whether managed or native retention keeps the pair alive.

## Observed bottlenecks (perf)

From local profiling and the cross-toolkit suite:

1. **Control handler creation** — mapping MAUI elements to native platform views dominates wall time when building dense UIs.
2. **Initial layout** — measuring/arranging large trees adds significant cost on first paint and when recycling fails to keep up during scroll.
3. **Not primarily CLR / BCL** — language and framework microbenchmarks show .NET is competitive enough; the gap vs Flutter / native / Avalonia on **UI creation** is toolkit-specific.

## Symptoms to improve

| Symptom | User impact | Likely technical driver |
| --- | --- | --- |
| Slow page open | Perceived lag / blank or incomplete UI | Many handlers + full layout of complex trees |
| Janky list scroll | Frame drops when new cells appear | Per-cell handler/layout cost exceeds frame budget |
| High cost even without bindings/styles | Optimization of XAML/MVVM alone is insufficient | Native view creation path itself is expensive |
| Rich text layout wrong/unstable | Jumping heights, broken lists, fragile hacks | WebView not designed for content-intrinsic sizing |
| Memory growth over session | Sluggishness, risk of kills; slow RCA | Managed ↔ native reference cycles / unclear ownership |

## Goals (working)

- Reduce **time-to-interactive** for complex pages while **keeping MAUI as the app host** (full migration out of scope for now).
- Make **scrolling** meet fluent frame budgets for typical list densities used in ThinkTime.
- Display **rich text** with reliable content-driven sizing (and fewer WebView workarounds).
- Make **leak diagnosis** tractable across the managed/native boundary; keep memory stable under navigation/scroll.
- Prefer solutions that are **measurable** against the existing UiPerformanceTest scenarios (Single Grid, Grids Layout, Absolute Layout, and product-like pages).
- Preserve **same look on iOS and Android** and the ability to style (or draw) UI to **Figma-defined** components and pages ([considerations/app.md](considerations/app.md)).

## Non-goals (for now)

- **Full app migration** off MAUI (Flutter, Avalonia, Uno, native rewrite of the entire UI) — possible later, **not currently considered**; see [considerations/options.md](considerations/options.md).
- Optimizing language microbenchmarks (already not the limiting factor for UI creation).

## Success signals

- Lower average ms on UiPerformanceTest MAUI scenarios (especially programmatic Single Grid / Grids Layout).
- Measurable improvement on ThinkTime page-open and scroll traces (handler + layout share of time down).
- Rich text regions reach a correct final size without visible jump (or with an accepted, short placeholder strategy).
- Memory stable after repeated open/close and scroll stress; leak class documented when fixed.
- Clear documented trade-offs (fidelity, maintainability, platform parity) for the chosen path.
