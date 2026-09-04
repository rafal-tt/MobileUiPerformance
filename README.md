# Mobile app UI performance — notes & decisioning

Working notes, considerations, and a decision tree for improving **ThinkTime** mobile UI performance on **.NET MAUI 10**.

Markdown in this repo is the source of truth. Publish to Confluence on demand:

- **Ideas / decisioning (publish target):** [Mobile app UI performance improving ideas](https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/4394024967/Mobile+app+UI+performance+improving+ideas)
- **Baseline measurements (published):** [Mobile app UI toolkits performance comparison](https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/3702128641/Mobile+app+UI+toolkits+performance+comparison)

## Top issues (short)

1. Long UI creation (handlers + initial layout).
2. Non-fluent list scrolling (same create cost on cells).
3. Rich text via WebView — poor content-driven sizing; workarounds imperfect.
4. Memory leaks hard to locate (managed ↔ native peer ownership unclear).

Details: [findings/top-issues.md](findings/top-issues.md).

## Related code & data

| What | Path / link |
| --- | --- |
| Product app (ThinkTime) | `/Users/rkukla/proj/thinktime/thinktime-xamarin` |
| Cross-toolkit UI perf suite | `/Users/rkukla/proj/cs/maui/UiPerformanceTest` |
| Local measurement artifacts | `/Users/rkukla/proj/cs/maui/UiPerformanceTest/binaries` |

## Document map

| File | Purpose |
| --- | --- |
| [problem-statement.md](problem-statement.md) | Symptoms, measured bottlenecks, goals |
| [findings/top-issues.md](findings/top-issues.md) | Canonical top product issues |
| [findings/baseline-measurements.md](findings/baseline-measurements.md) | Key numbers from the toolkit comparison |
| [decision-tree.md](decision-tree.md) | How to choose an improvement path |
| [ideas/](ideas/) | Individual improvement ideas (one file each) |
| [considerations/team.md](considerations/team.md) | Team size, skills, learning posture |
| [considerations/app.md](considerations/app.md) | App scale, offline, Figma-strict same iOS·Android look, critical UI |
| [considerations/options.md](considerations/options.md) | Options + **current recommendation** (SkiaUi primary, HTML regions parallel) |
| [considerations/migration-dependencies.md](considerations/migration-dependencies.md) | Deps to validate for Flutter / Avalonia / non-MAUI |
| [considerations/](considerations/) | Cross-cutting risks, constraints, open questions |

## Workflow

1. Capture findings and ideas as `.md` here.
2. Refine the decision tree as evidence accumulates.
3. When asked, sync selected pages to Confluence (`4394024967` and children as needed).
