# Skia or WebView surface for hotspots

**Status:** Draft  
**Targets:** Specific pages or list regions where native control fidelity is optional.

## Hypothesis

Replacing a dense MAUI subtree with one surface (Skia canvas or WebView) removes most handler creation. Baseline Single Grid: SkiaSharp **~126 ms**, WebView **~135 ms** vs MAUI programmatic **~1548 ms**.

## When to use

- Read-heavy or custom-drawn schedules/grids.
- Prototype to prove budget is achievable before investing in DrawnUI fidelity.
- WebView when HTML/CSS layout is acceptable, interaction is limited, and the host can use a **known size** (note: **first** open much slower; warm path is fast).
- **Not** a default fix for product rich text that must **size to content**—that is tracked under [rich-text-display.md](rich-text-display.md).

## Measure

- Same UiPerformanceTest scenarios (already implemented in the suite).
- ThinkTime hotspot page: time-to-first-contentful vs interaction completeness.

## Risks

- Interaction model (gestures, accessibility, text input).
- WebView cold start and memory.
- Dual UI stacks (MAUI + HTML/Skia) increase maintenance.
- Grids Layout with SkiaSharp still **~1320 ms** in suite—multi-region layouts need careful design (one canvas vs many).
