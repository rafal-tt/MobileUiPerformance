---
confluence_page_id: "4394024967"
confluence_url: "https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/4394024967/Mobile+app+UI+performance+improving+ideas"
github_repo: "https://github.com/rafal-tt/MobileUiPerformance"
published: "2026-09-04"
html_mirror: "4394024967-body.html"
---

# Mobile app UI performance improving ideas

> Markdown mirror of the Confluence page for diffs. Markers: `sync-begin:<id>` / `sync-end:<id>`. See [SYNC.md](SYNC.md).

## Sync metadata

Live page mirrored in GitHub for diffs. Edit Confluence or this mirror; reconcile with section markers.

- GitHub workspace: https://github.com/rafal-tt/MobileUiPerformance
- Mirror: `confluence/4394024967-Mobile-app-UI-performance-improving-ideas.md`
- Confluence page ID: `4394024967`
- Published: 2026-09-04

Related: [Mobile app UI toolkits performance comparison](https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/3702128641/Mobile+app+UI+toolkits+performance+comparison)

---

sync-begin:scope

## Scope

| Status | Meaning |
| --- | --- |
| **Out of scope (for now)** | Full app migration off MAUI (rewrite the whole UI in Flutter, Avalonia, Uno, native, etc.) |
| **Possible / not considered** | Those full-migration paths — documented for later; no active spikes or planning |
| **Currently considered** | Stay on MAUI as host; improve hotspots via WebView / Blazor / Skia-drawn UI / composition; optional partial Flutter add-to-app backup |

**Already tried and not enough:** visual-tree reduction and list virtualization (continue as hygiene only).

sync-end:scope

---

sync-begin:constraints

## App constraints

| Constraint | Detail |
| --- | --- |
| Scale | Large, complex MAUI app (multi-project; substantial UI surface) |
| Platforms | Android + iOS; .NET MAUI 10 |
| Offline | Full offline mode; local data in Realm |
| Look & feel | **Same look on iOS and Android** (not stock native Material vs UIKit) |
| Design | Every component look and page layout is **strictly defined in Figma** — solution must allow styling or drawing controls to that spec |
| Team | 3 software engineers; deep C# / .NET MAUI; willing to learn new tech when justified |
| Critical 3rd-party UI | PDF viewer, Rich Text Editor, Image editor — consolidating on **Syncfusion** (Telerik being removed) |
| Other notable deps | Syncfusion ListView and related controls, custom WebView for HTML content, Firebase push, BLE, multi-customer overlays/branding |

**Design implication:** Prefer approaches with high style control (own renderer or HTML/CSS design tokens). Stock platform defaults and unthemed vendor chrome are a poor fit. Third-party controls are acceptable only if theming can match Figma.

sync-end:constraints

---

sync-begin:problems

## Problems to solve

1. **Long UI creation time** — Opening complex pages is too slow. Dominant cost is creating control handlers and initial layout (not CLR microbenchmarks). Visual-tree reduction alone is already practiced and insufficient.

2. **Non-fluent list scrolling** — Same root cause: creating UI for new cells exceeds the frame budget. List virtualization is already practiced and insufficient while cells remain expensive.

3. **Rich text display (WebView sizing)** — Backend/content often HTML. System WebView does not adapt size to content well inside native layout. Workarounds exist but are imperfect (sizing races, jumps, nested scroll issues).

4. **Hard-to-locate memory leaks** — Each platform view has a native peer; retention can be owned from either side, which makes root-cause analysis across the managed ↔ native bridge difficult.

**Evidence (toolkit comparison, Galaxy S9):** MAUI 10 programmatic Single Grid ~1548 ms vs Flutter ~134 ms, SkiaSharp-in-MAUI ~126 ms, WebView ~135 ms (warm). Language/framework microbenchmarks do not explain the UI gap.

sync-end:problems

---

sync-begin:options-considered

## Options currently considered (MAUI-hosted)

### A. Full-page UI in WebView (HTML/JS)

Render most of a page as HTML/CSS/JS; C# hosts shell, auth, offline bridge.

| Pros | Cons |
| --- | --- |
| Fast create/runtime in our tests (warm path) | System WebView variance across OEM/OS versions |
| Natural fit for HTML from backend | Dynamic UI pushes work into JS/TS; harder debugging for this team |
| Can ship page-by-page | Icon font → images/webfont migration; first-open cost; memory/leak surface |
| Reuses existing WebView/auth patterns | Content-sized WebViews in lists remain hard |

### B. Blazor Hybrid

MAUI shell hosts BlazorWebView; UI authored in C# + Razor inside the WebView.

| Pros | Cons |
| --- | --- |
| Stays closer to .NET skills than raw JS | Still system WebView–bound (create model ≠ Flutter/Skia for huge lists) |
| Incremental page conversion; Core/services reusable | Dual toolkit (XAML + Razor); first paint / warm-up care needed |
| Better debugging than a JS SPA for this team | Does not fix auto-height WebViews in lists by itself |

### C. HTML regions only

Keep MAUI chrome/lists; use WebView only for rich HTML bodies (evolve current approach).

| Pros | Cons |
| --- | --- |
| Lowest effort; closest to today | **Does not** fix dense page-open / scroll jank |
| Best fit for rich-text *display* correctness | Many WebViews ⇒ memory/leak risk; sizing workarounds remain delicate |
| Leaves Syncfusion editors on MAUI | CSS vs MAUI style drift if not tokenized |

### D. Skia-drawn UI (custom layer / SkiaUi)

MAUI hosts one (or few) GPU Skia surfaces; measure/layout/paint/hit-test in C# for dense subtrees.

| Pros | Cons |
| --- | --- |
| Fast when the whole dense subtree is drawn (suite-class wins) | Building a control/layout framework (front-loaded effort) |
| All C# — maintain, debug, extend, leak RCA aligned with the team | Does not solve HTML rich text alone |
| Strong Figma + same iOS/Android look | Partial “draw some controls, keep MAUI layout” (e.g. DrawnUI-style) was only partly faster in tests |
| Incremental after bridge + primitives exist | Must still host Syncfusion PDF/RTE/ImageEditor as MAUI islands |

### E. Composition: Skia + WebView + MAUI Syncfusion

Skia for dense lists/grids; WebView for HTML bodies; Syncfusion MAUI for PDF / RTE / Image Editor.

| Pros | Cons |
| --- | --- |
| Right tool per problem; matches evidence + product pain | Clear ownership rules needed (multiple UI paradigms) |
| Keeps critical commercial controls on MAUI | Team must document and enforce boundaries |

### F. Backup — Flutter add-to-app islands (partial)

Keep MAUI app; embed Flutter only for worst screens. **Not** a full rewrite.

| Pros | Cons |
| --- | --- |
| Flutter create/scroll where it hurts most | Two UI stacks (Dart + C#); navigation/lifecycle glue |
| No full-app rewrite | No first-class “Flutter inside MAUI” product — custom native wiring |
| Can prove Flutter on one pilot screen | Harder long-term maintainability; use only if MAUI hybrids miss targets |

sync-end:options-considered

---

sync-begin:options-parking

## Possible but not currently considered (full migration)

Kept for later if MAUI-hosted strategies cannot meet goals. **No active planning.**

### Flutter — full / large migration

| Pros | Cons |
| --- | --- |
| Excellent UI create/scroll; hot reload; large ecosystem | Very large convert/test effort; learn Dart |
| Same pixels iOS/Android; good Figma fit | Core/API rewrite or dual maintain; overlays/BLE/push/SSO rework |
| Syncfusion Flutter PDF Viewer available | Syncfusion Flutter: no RTE, no Image Editor (alts TBD) |

### Avalonia — full / large migration

| Pros | Cons |
| --- | --- |
| Much faster UI create than MAUI in our suite; C# / possible Core reuse | Full UI convert upfront; mobile maturity/docs weaker than MAUI |
| Own Skia-style rendering; Figma-friendly | Commercial control coverage TBD; platform integrations reworked |

Also parking lot: Uno full migration; Compose/SwiftUI islands (poor fit for same look + Figma unless double-styled).

sync-end:options-parking

---

sync-begin:recommendation

## Recommendation

**Primary:** Skia-drawn UI layer for dense pages/lists (all C#; Figma-friendly; suite-class performance when the whole dense subtree is drawn).

**Parallel (required, lower effort):** Harden HTML-region WebViews for rich-text display (fixed/locked height; prefer detail WebView over many auto-height cells; aggressive dispose).

**Keep as MAUI islands:** Syncfusion PDF viewer, Rich Text Editor, Image Editor.

**Target end shape:** composition — Skia for density, WebView for HTML bodies, Syncfusion for editors.

| Not primary | Why |
| --- | --- |
| Full Flutter / Avalonia | Out of scope; rewrite cost |
| Full-page HTML/JS WebView as default | Strong create perf; weaker maintain/debug/extend for this team |
| Blazor Hybrid as default | Better than JS for C#, still WebView-taxed; weaker for huge recycled lists vs Skia |
| MAUI-only further tuning | Already insufficient |
| Flutter add-to-app | Backup only if Skia + HTML regions miss targets |

### Suggested next steps

1. Time-box Skia bridge + few primitives → one real product hotspot styled to Figma; measure create, scroll, memory.
2. In parallel, ship HTML-region sizing/lifecycle fixes.
3. If proven, grow Skia control set and roll more dense screens; compose with WebView + Syncfusion.
4. Only then consider Flutter add-to-app backup or reopen full-migration parking lot.

sync-end:recommendation

---

sync-begin:footer

## Working notes

Detailed working notes: https://github.com/rafal-tt/MobileUiPerformance

Related measurement write-up: https://ssi.atlassian.net/wiki/spaces/PEPRODUCTS/pages/3702128641/Mobile+app+UI+toolkits+performance+comparison

sync-end:footer
