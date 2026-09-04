# MAUI platform / version tuning

**Status:** Draft  
**Targets:** Broad improvements without rewriting screens.

## Hypothesis

Some gains come from framework upgrades, layout engine changes, and handler map trimming—but **not enough alone** based on MAUI 11 preview Single Grid (~1330 ms) still ≫ Flutter/Avalonia.

## Levers

- Evaluate MAUI 11+ on UiPerformanceTest and a ThinkTime smoke page.
- Trim unused handlers / features if startup or map size matters (secondary to create cost).
- Prefer code-built UI over heavy XAML+binding for hottest paths where maintainable.
- Watch Microsoft guidance on layout performance, CollectionView, and compiled bindings.

## Measure

- Side-by-side MAUI 10 vs 11 APKs already in `UiPerformanceTest/binaries`.
- Re-run after each upgrade candidate.

## Risks

- Preview stability; breaking changes; BindableLayout regressions seen in published MAUI 11 numbers.
