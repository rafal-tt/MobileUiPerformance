# DrawnUI or custom handlers

**Status:** Draft  
**Targets:** Dense grids/lists that must stay inside the MAUI app.

## Hypothesis

Per-control native handlers dominate cost. Drawing (or sharing) platform surfaces for many logical controls cuts create/layout time while remaining in the MAUI process.

## Evidence from baseline (MAUI 10)

| Approach | Single Grid (ms) | Grids Layout (ms) |
| --- | ---: | ---: |
| MAUI programmatic | 1548 | 1946 |
| DrawnUI (Skia-drawn controls) | 646 | 1338 |
| SkiaSharp custom | 126 | 1320 |

DrawnUI helps Single Grid substantially; multi-grid layout cost remains high—investigate layout strategy and whether Absolute Layout / custom measure is needed (MAUI 11 DrawnUI Absolute Layout looked strong in published table).

## Levers

- Adopt DrawnUI (or similar) for hot screens/lists.
- Custom handlers that batch children onto one native view.
- Handler pooling / reuse experiments if applicable to our MAUI version.

## Risks

- Accessibility, input, focus, and styling parity.
- Dependency and upgrade cost; team skill with Skia drawing models.
- Incomplete coverage of existing control set.
