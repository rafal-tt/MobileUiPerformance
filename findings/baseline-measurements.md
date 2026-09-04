# Baseline measurements

Source of truth for published numbers: [Mobile app UI toolkits performance comparison](https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/3702128641/Mobile+app+UI+toolkits+performance+comparison).

Local artifacts: `/Users/rkukla/proj/cs/maui/UiPerformanceTest/binaries` (`Results*.md`, `Results*.xlsx`, APKs).

**Device:** Samsung Galaxy S9 (Exynos 9810, 4 GB RAM, 1440×2960).  
**Method:** Average of timed attempts after discarding the first (warm-up).

## UI creation — headline comparisons

Approximate **Single Grid** (create UI programmatically, no bindings/styles) averages:

| Toolkit | Average (ms) |
| --- | ---: |
| Flutter | 134 |
| SkiaSharp in MAUI 10 (text/images via Skia) | 126 |
| WebView in MAUI 10 (HTML generated in C#) | 135 *(first run ~1000 ms)* |
| Native Java | 226 |
| Avalonia | 338 |
| DrawnUI (MAUI 10, Skia-drawn controls) | 646 |
| MAUI 11 preview (programmatic) | 1330 |
| MAUI 9 (programmatic) | 1074 |
| **MAUI 10 (programmatic)** | **1548** |

MAUI 10 **Grids Layout** programmatic ≈ **1946 ms**; XAML / BindableLayout paths are often slower still (e.g. BindableLayout ≈ **2662 ms** on MAUI 10).

### Interpretation

- Classic MAUI control trees are **~5–12× slower** than Flutter / carefully drawn Skia / WebView on the same device for this scenario.
- **Drawn / Skia / WebView** approaches inside MAUI already show large wins on Single Grid—evidence that **avoiding per-control native handlers** (or deferring them) is the high-leverage lever.
- MAUI 11 preview improves some programmatic paths vs MAUI 10 but remains far from Flutter/Avalonia; BindableLayout on MAUI 11 is worse in the published table (**3420 ms**).

## Language / framework microbenchmarks

From the same suite (averages, ms) — useful to show the bottleneck is **not** “C# is slow”:

| Area | Java | Compose | Flutter | Avalonia | MAUI 10 | MAUI 11 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Language “Run All” | 473 | 408 | 496 | 1006 | 1079 | 591 |
| Framework “Run All” | 1142 | 1492 | 425 | 852 | 970 | 377 |

.NET 11 (MAUI 11 preview host) narrows language/framework gaps; that does **not** close the UI-creation gap for standard controls.

## Implication for ThinkTime

Invest in strategies that **avoid 1:1 native handlers** for dense UI (drawn/custom renderers, hybrid WebView/Blazor surfaces, or toolkit change)—not in micro-optimizing loops or JSON deserialize. Visual-tree reduction and list virtualization are already in use and do not close the measured gap.
