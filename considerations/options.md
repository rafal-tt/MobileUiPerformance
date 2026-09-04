# Options

Working comparison of strategic paths for ThinkTime mobile UI performance.  
Tied to [top-issues](../findings/top-issues.md), [app.md](app.md), [team.md](team.md), [migration-dependencies.md](migration-dependencies.md), and baseline measurements.

## Scope

| Status | What |
| --- | --- |
| **Out of scope (for now)** | **Full app migration** off MAUI (rewrite the whole UI in Flutter, Avalonia, Uno, native, etc.) |
| **Possible but not considered** | Those full-migration paths — kept documented for later if MAUI-based approaches fail; **no active spikes or planning** |
| **Currently considered** | Stay on MAUI as host; improve hotspots via WebView / Blazor / Skia / composition (and optional partial Flutter add-to-app backup) |

**Status:** Draft — decisions within the currently considered set only; no full-migration decision in flight.

---

## Recommendation (current)

**Best overall tradeoff** under present scope (MAUI host, Figma-strict parity, 3 C# engineers, Syncfusion PDF/RTE/ImageEditor, tree/virtualization already insufficient):

### Primary: **SkiaUi for dense UI** (options §3.b → evolve toward §4.5)

Invest in a **Skia-drawn control/layout layer** for hotspot lists and complex pages (work already scoped in `/Users/rkukla/proj/cs/maui/SkiaUi`).

| Why this wins the tradeoff | |
| --- | --- |
| **Effect** | Suite class of wins (~100–150 ms create vs ~1.5 s MAUI handlers) when the **whole** dense subtree is drawn — same problem class as page-open + scroll |
| **Effort** | Higher than another MAUI tweak, **far lower** than full Flutter/Avalonia; incremental (one list/page at a time after the bridge + few primitives exist) |
| **Maintain / debug / extend** | **All C#** — matches the team; no Dart/JS dual stack; leaks stay on one mental model better than WebView+JS; extend by adding `SkUi*` controls styled to Figma |
| **Figma / iOS=Android** | Own renderer → one look; no fighting stock native chrome |

**Do not** rely on DrawnUI-style “half MAUI / half drawn” as the end state — measurements showed limited gains when layout still pays MAUI costs.

### Parallel (required, lower effort): **harden HTML regions** (§3.a″)

For rich text **display**, keep improving the existing WebView-island approach (fixed/locked height, prefer detail WebView over N auto-height cells, aggressive dispose). This is the **cheapest** path for issue #3; it will **not** fix dense create/scroll by itself.

### Keep as islands: Syncfusion on MAUI

PDF viewer, RTE, Image Editor stay MAUI Syncfusion modals/pages — do not reimplement inside Skia.

### Explicitly not the primary bet

| Option | Why not primary |
| --- | --- |
| Full Flutter / Avalonia | Out of scope; high rewrite cost |
| Full-page HTML/JS WebView (§3.a) as default | Strong create perf, weaker **maintain/debug/extend** for this team (JS/TS, WebView variance, icon/font pipeline) |
| Blazor Hybrid (§3.a′) as default | Better than JS for C# debugging, still WebView-bound; dual XAML+Razor tax; weaker for huge recycled lists vs Skia |
| MAUI tune only (§4.1) | Already insufficient |
| Flutter add-to-app (§4.2) | Keep as **backup** if SkiaUi + HTML regions miss targets — dual stack is harder to maintain |

### Suggested delivery shape

1. Time-box SkiaUi bridge + 1–2 layouts + label/list cell → **one real ThinkTime hotspot** styled to Figma; measure create + scroll + memory.
2. In parallel, ship 3.a″ sizing/lifecycle fixes for HTML bodies.
3. If hotspot proves out, grow SkiaUi control set and roll more dense screens; compose with WebView HTML + Syncfusion modals (§4.5).
4. Only then revisit §4.2 or reopen full-migration parking lot.

---

## How to read this

- **In scope:** §3 MAUI-based (3.a / 3.a′ / 3.a″ / 3.b), §4.1 hygiene, §4.5 composition, §4.2 as **partial** backup (not full rewrite).
- **Parking lot:** §1 Flutter full, §2 Avalonia full, Uno/native full rewrites in §4.3 — possible, **not currently considered**.
- **WebView family:** 3.a full-page HTML/JS, 3.a′ Blazor Hybrid, 3.a″ HTML regions only — see detailed split under §3.a.
- Effort estimates assume a **3-person** team with deep C#/MAUI and willingness to learn ([team.md](team.md)).
- Critical controls direction: **Syncfusion** for PDF / RTE / Image Editor after **Telerik removal** ([app.md](app.md)).
- **Design constraint ([app.md](app.md)):** same look on iOS and Android; component and page UI **strictly from Figma** — chosen stack must allow styling (or drawing) controls to that spec, not stock platform look.

---

## 1. Flutter (full or large migration) — possible / not considered

> **Out of scope for now.** Full (or large) app migration to Flutter is **not** under active consideration. Retained as a possible future path if MAUI-hosted options cannot meet goals. Do not start conversion planning or gap matrices as project work unless scope changes.

### Pros (stated + added)

| Pros | Notes |
| --- | --- |
| Fast UI creation / scroll | Matches our suite (Single Grid ~134 ms vs MAUI 10 ~1548 ms) |
| Strong VM + **hot reload** | Fast UI iteration |
| Good free IDE (Android Studio / VS Code) | Mature Flutter tooling |
| Large ecosystem; de-facto mobile standard | Hiring, samples, Stack Overflow density |
| Consistent pixels (own renderer) | Fewer platform layout surprises; strong fit for **same look iOS/Android** + Figma |
| Realm has a Flutter/Dart SDK | Offline path exists (still a schema/port effort) |
| Syncfusion Flutter **PDF Viewer** available | Commercial Syncfusion control; still verify feature parity + Figma theming vs MAUI |

### Cons (stated + added)

| Cons | Notes |
| --- | --- |
| Requirements gap analysis required | Especially commercial controls parity |
| Syncfusion Flutter: **no RTE**, **no Image Editor** (as of last check) | PDF Viewer exists; need third-party or custom for RTE + image edit; feature parity TBD |
| Learn **Dart/Flutter** | Acceptable per team posture, still ramp cost |
| Large convert + test effort | Even with AI assist; ~440k LOC / complex overlays |
| Dual ecosystem forever if incomplete | Or long “strangler” with MAUI + Flutter |
| Rewrite or dual-maintain Core/API/Mapster | Not a ViewModel lift-and-shift |
| BLE, push, SSO, overlays, WebView auth all re-integrated | See [migration-dependencies.md](migration-dependencies.md) |
| Does not automatically fix **product** rich-text *sizing* workflows | Different widgets; still need an HTML/display strategy |
| App size / cold start / plugin quality vary | Must measure on Galaxy-class and low-end devices |
| **Full migration currently out of scope** | Blocks adopting this as the active plan |

### Deferred checks (only if scope reopens)

- Gap matrix: Syncfusion MAUI RTE/ImageEditor vs Flutter alternatives
- Realm schema + sync/offline port plan
- Overlay/branding build story in Flutter
- Pilot one hotspot screen end-to-end

---

## 2. Avalonia (full or large migration) — possible / not considered

> **Out of scope for now.** Full (or large) app migration to Avalonia is **not** under active consideration. Retained as a possible future path. No active spikes unless scope changes.

### Pros (stated + added)

| Pros | Notes |
| --- | --- |
| Fast in our suite | Programmatic Single Grid ~338 ms (still ≫ Flutter, ≪ MAUI) |
| **C#** — potential ViewModel / Core reuse | Aligns with team skills; Mapster/JWT/Realm .NET stay viable |
| “Can use MAUI controls” (claimed / hybrid stories) | **Validate** only if scope reopens; not a free lunch |
| Own Skia-based rendering | Avoids per-control native handler tax; strong **Figma / cross-platform parity** fit |
| XAML familiarity | Closer mental model than Dart for this team |

### Cons (stated + added)

| Cons | Notes |
| --- | --- |
| Support/docs weaker than MAUI for mobile | Smaller community; fewer “ThinkTime-shaped” samples |
| Convert + test **entire** UI upfront | Less incremental than MAUI hybrid options |
| Learn Avalonia layout/controls | Non-zero ramp |
| Mobile maturity younger than Flutter/MAUI | Production risk on iOS/Android edge cases |
| Syncfusion **Avalonia** control coverage TBD | PDF/RTE/ImageEditor may force WebView or other vendors |
| Platform integrations reworked | Firebase, BLE, badges, pickers, etc. |
| Cold start still .NET-on-mobile | UI create can be fast while startup remains costly |
| Hiring/pool smaller than Flutter | Long-term staffing risk |
| **Full migration currently out of scope** | Blocks adopting this as the active plan |

### Deferred checks (only if scope reopens)

- Spike: ThinkTime-like page; create + scroll on S9-class device
- MAUI control embedding viability
- Syncfusion (or alt) PDF/RTE/ImageEditor on Avalonia mobile
- Realm + offline under Avalonia Android/iOS packaging

---

## 3. MAUI-based solutions (incremental) — currently considered


### Shared pros / cons for staying on MAUI

| Pros | Cons (fill as we learn) |
| --- | --- |
| Least rewrite: convert **critical** surfaces only | Residual MAUI handler cost on non-converted UI |
| New work can adopt new approaches without big-bang | Two UI paradigms in one app (complexity, hiring docs) |
| No new language | Risk of never finishing “critical parts” → permanent hybrid mess |
| Keeps Syncfusion MAUI stack (post-Telerik) | Syncfusion controls themselves may still be handler-heavy |
| Overlays / DevOps / SSO largely unchanged | Does not by itself fix managed↔native leak opacity |
| | MAUI 11+ may help some paths but won’t close Flutter-sized gaps alone ([baseline](../findings/baseline-measurements.md)) |

---

### 3.a Entire page UI in a WebView

Render (most of) a page as HTML/CSS/JS inside system WebView; C# hosts shell, auth, offline bridge.

| Pros | Cons |
| --- | --- |
| Fast create + runtime in suite (~135 ms warm Single Grid) | **System WebView** variance (OEM/OS version → layout/JS differences) |
| Natural fit for **HTML rich content** from backend | Icon **font → images** (or webfont) migration |
| Reuses existing web skills / possibly shared web modules | Fast dynamic UI pushes work into **JS** (TS compile into C# solution TBD) |
| Can ship page-by-page | **Hard debugging** (two runtimes, bridge bugs) |
| Already have WebViewExtended / auth / cookie patterns | Content-sized WebView in lists remains hard ([top-issues](../findings/top-issues.md)) |
| | Accessibility, focus, offline asset packaging |
| | Memory: WebView peers are heavy; leak RCA still hard |
| | First-open cost high in suite (~1s); need warm/reuse strategy |
| | Security: JS bridge attack surface |

Same **WebView family** as 3.a′ / 3.a″ below: all trade native MAUI visual trees for HTML (or Razor→HTML) inside a platform WebView. They differ mainly in **how much** of the screen is web and **who authors** the UI (JS/TS vs C#/Razor vs backend HTML).

---

### 3.a′ Blazor Hybrid (WebView family)

**Meaning:** Keep the MAUI app shell. For selected pages (or large regions), host **Blazor** inside a WebView (`BlazorWebView` / Blazor Hybrid). UI is authored in **C# + Razor** (`.razor` components), not hand-written JS. Blazor renders to the WebView DOM; C# still talks to Realm, APIs, and ViewModels via ordinary .NET references (same process).

```text
MAUI page / shell
└── BlazorWebView          ← one (or few) native WebView peer(s)
        └── Razor components (C#)
                ├── layout, lists, forms in HTML/CSS
                └── calls into ThinkTime.Core services / VMs
```

**How it differs from 3.a (full page HTML/JS)**

| | 3.a HTML/JS page | 3.a′ Blazor Hybrid |
| --- | --- | --- |
| UI language | HTML + JS/TS | C# + Razor (+ CSS) |
| Team fit | Needs web/JS pipeline | Stays closer to .NET skills |
| Debugging | Split JS + C# bridge | Mostly C#; still a WebView underneath |
| Sharing with existing web | Easier if web already owns the page | Better if mobile owns the feature in .NET |
| Perf model | Same WebView create/layout | Same WebView create/layout — **not** Flutter/Skia class unless UI is simple |

**Pros**

- Incremental: convert one page or feature at a time; shell/nav/Syncfusion modals stay MAUI.
- No Dart; ViewModels/services can be reused more directly than Flutter.
- Rich HTML/CSS layout without fighting MAUI handlers for that subtree.
- Better debugging/fixing for this team than a pure JS SPA in the WebView.
- Can still display backend HTML fragments inside Razor if needed.

**Cons**

- Still **system WebView**-dependent (OEM variance, memory, leak opacity).
- Dense, highly dynamic lists at 60 fps are a weak fit unless virtualized carefully in the DOM (or kept short).
- Blazor Hybrid startup / first paint can be heavy; need warm `BlazorWebView` reuse strategy.
- Dual UI toolkit in one app (MAUI XAML + Razor); styling and navigation conventions to define.
- Does not magically fix **content-sized** WebViews inside MAUI lists — prefer full-screen Blazor pages or fixed-height regions.
- Icon font / design tokens still need a web-friendly story.
- Offline: must design how Razor pages read Realm (inject services; avoid assuming browser-only storage).

**Good fit when**

- A whole screen is interaction + layout heavy in MAUI, but acceptable as a document/app page in HTML.
- Team prefers C# over JS for that screen’s logic.
- Perf target is “fast enough create + smooth enough scroll for that page,” not beating Flutter on huge recycled cell trees.

**Poor fit when**

- The hotspot is a long virtualized native-style list with complex cells (prefer SkiaUi / Flutter / MAUI list tuning).
- Many tiny WebViews per list row (today’s rich-text sizing pain).

---

### 3.a″ HTML regions only (WebView family)

**Meaning:** Do **not** move the whole page to web. Keep MAUI for chrome, lists, buttons, navigation. Use WebView **only** where content is already HTML (or must be rich text): article bodies, comments, task descriptions, KB sections — i.e. evolve what ThinkTime already does with `WebViewExtended` + `HtmlWebViewSource`.

```text
MAUI page
├── MAUI header / toolbar / actions
├── MAUI list / form fields          ← still native handlers
└── WebViewExtended (island)       ← HTML body only
        └── backend HTML / sanitized fragment
```

**How it differs from 3.a / 3.a′**

| | Full page WebView (3.a) | Blazor Hybrid (3.a′) | HTML regions only (3.a″) |
| --- | --- | --- | --- |
| Scope | Most/all of page | Page or large region in Razor | Small islands for rich content |
| Primary goal | Cut MAUI create cost for whole screen | Same, with C# UI authoring | Fix **rich text display** / reuse HTML; not full-page perf |
| Closest to today | Bigger jump | Medium jump | **Incremental** (already partially there) |
| Page-open of dense native UI | Can help a lot | Can help a lot | **Does not** — surrounding MAUI tree still expensive |

**Pros**

- Smallest WebView-family bet; builds on existing handlers, auth cookies, sanitization (`HtmlSanitizer` / `HtmlAgilityPack`).
- Best alignment with “backend sends HTML” and rich-text **display** needs.
- Leaves Syncfusion PDF/RTE/ImageEditor and MAUI lists where they are.
- Can harden sizing contract (fixed height, expand-on-tap, full-screen reader) without rewriting the app.

**Cons**

- Does **not** solve long UI creation or list scroll caused by many MAUI/Syncfusion cells.
- Content-sized WebView in scrolling lists remains the hard problem (workarounds imperfect — [top-issues](../findings/top-issues.md)).
- Many WebView instances ⇒ memory and leak surface.
- Inconsistent look if HTML CSS drifts from MAUI styles.
- Editing may still need Syncfusion RTE (modal); display path and edit path stay split.

**Practical directions for 3.a″ (product)**

1. **Display:** prefer one WebView per detail screen (or “open full content” page) over N auto-height WebViews in a list.
2. **Lists:** show plain/attributed preview text in MAUI; defer full HTML to detail.
3. **Sizing:** fixed height + internal scroll, or measure-once then lock height — avoid continuous reflow fights with MAUI layout.
4. **Reuse:** pool/warm a WebView where possible; destroy aggressively on page leave (leak hygiene).

**Good fit when**

- Priority is rich-text **correctness** and simpler HTML hosting, not rewriting dense native screens.
- Combined with 4.1 / 3.b / 4.5 for create/scroll hotspots.

**Poor fit when**

- Treated as the only answer to page-open and scroll jank (it won’t be).

---

### WebView family — how to choose (3.a / 3.a′ / 3.a″)

```text
Is the goal mostly rich HTML display / fewer sizing hacks?
├─ YES → start with 3.a″ (HTML regions only); harden sizing & lifecycle
└─ NO — goal is faster whole-screen UI create
         ├─ Prefer C# UI authoring? → spike 3.a′ Blazor Hybrid page
         └─ Prefer existing web/JS (or shared web app)? → spike 3.a full page WebView
```

All three still need a plan for: WebView variance, memory, first-load warm-up, icons, and not nesting auto-height WebViews in hot lists.

### 3.b Components drawn in Skia (custom framework / SkiaUi)

MAUI hosts one (or few) Skia surfaces; UI tree measure/layout/paint/hit-test in C# (see `/Users/rkukla/proj/cs/maui/SkiaUi` requirements: Flutter-like constraints, XAML-authorable `ISkUiView` tree, GPU `SKGLView`).

| Pros | Cons |
| --- | --- |
| Fast when **one** surface draws many visuals (suite SkiaSharp Single Grid ~126 ms) | Best results need **whole** subtree in Skia — DrawnUI-style hybrids were only partly faster (esp. multi-grid) |
| Flexible rendering | Essentially building a **new UI framework** (layouts, controls, a11y, focus, text) |
| All C# — easier debug / leak tracking than JS WebView | **Does not solve rich text** (HTML) by itself |
| Aligns with team language | Unclear if MAUI controls (WebView, Entry, …) can sit *inside* Skia tree — SkiaUi marks embedding **out of scope** for now |
| Incremental: wrap hot pages/lists first | Long runway before feature parity with MAUI+Syncfusion controls |
| | Input, IME, accessibility, RTL, dynamic type — easy to under-scope |
| | Must still host PDF/RTE/ImageEditor as MAUI/Syncfusion islands or reimplement |
| | Risk of competing with other long-horizon bets if scope later reopens to Flutter/Avalonia full migration |

#### Related note on DrawnUI

DrawnUI (Skia-drawn controls on MAUI) helped Single Grid in our suite but **Grids Layout** stayed expensive — agrees with “partial draw isn’t enough; structure must live in the drawn tree.”

---

## 4. Additional options to consider

Not yet “chosen,” but worth keeping on the board.

### 4.1 Stay on MAUI — optimize without new renderer

**Status:** Background hygiene only — **not a primary option**. Tree flattening and list virtualization are **already practiced** and have proven **not enough**.

Continue lightly: lighter templates, compiled bindings, Syncfusion ListView complexity checks, MAUI 11+ evaluation — but do not expect this track alone to meet create/scroll goals. Strategic bets are §1–3 and §4.2/4.5.

| Pros | Cons |
| --- | --- |
| Low cost ongoing cleanup | **Already insufficient** for product targets |
| Slight wins on some screens | Distracts from renderer/toolkit decisions if over-weighted |

### 4.2 Backup — Flutter add-to-app islands for hot screens

**Role:** **Partial** backup only (not full migration). If MAUI-hosted paths (3 / 4.5) miss targets on specific screens, embed Flutter for those hotspots while the app remains MAUI-hosted. Full Flutter rewrite (§1) stays **out of scope**.

**Meaning:** Keep ThinkTime as a **MAUI app**. Rebuild only the **slowest screens** (or parts of screens) as a **Flutter module** embedded in that app.

Flutter’s pattern is [Add-to-app](https://docs.flutter.dev/add-to-app): the host stays Android/iOS (MAUI → native shell). Flutter renders selected UI into a Flutter `View` / `Fragment` / `UIViewController` that the host shows like any other screen or panel.

**“Islands”** = isolated Flutter hotspots, not a full rewrite. Example split:

| Stays on MAUI | Moves to Flutter island |
| --- | --- |
| Login, navigation shell, sync, Realm, overlays | One dense list page, heavy dashboard, or similar hotspot |
| Syncfusion PDF / RTE / ImageEditor modals (until Flutter alts proven) | Screens where handler/layout create cost dominates |

User navigates MAUI → Flutter screen → back to MAUI. Data crosses a bridge (method channels / Pigeon / custom .NET↔Dart interop).

| Pros | Cons |
| --- | --- |
| Flutter UI create/scroll where it hurts most | **Two UI stacks** to own (Dart + C#/MAUI) |
| No full ~440k-LOC rewrite first | Navigation / lifecycle / DI glue across hosts |
| Can prove Flutter perf with a pilot screen | Plugin/push/BLE quirks in add-to-app mode |
| Possible stepping stone if full Flutter (§1) ever reopens | **No maintained first-class “Flutter inside MAUI” product** — wire Flutter into Android/iOS heads yourself (older Flutnet-style bridges are dated) |
| | Medium–high effort hybrid — not a free win |
| | Risk of permanent dual-stack if islands grow without a rewrite plan |

**When to pick this backup**

- Hotspot screens fail targets after MAUI WebView / Skia / composition spikes, **and**
- Full app migration (§1 / §2) remains **out of scope**, **and**
- Team accepts dual-stack cost for a bounded set of screens.

### 4.3 Other strangler hybrids — mostly parking lot

| Variant | Status | Idea | Pros | Cons |
| --- | --- | --- | --- | --- |
| **Native islands** (Compose / SwiftUI) | Possible / not considered | Platform UI for one feature | Best platform perf/a11y | Conflicts with same look iOS/Android + Figma unless double-styled; 3-eng cost |
| **Uno Platform** (full-ish) | Possible / not considered | C# / WinUI-ish XAML, optional Skia | Stay in .NET; Skia-class renderer | **Full migration class** — out of scope with Flutter/Avalonia; Syncfusion TBD |

### 4.4 Rich-text-specific track (orthogonal)

Regardless of toolkit choice for dense UI: native attributed text / HTML subset renderer / shared web component with a **fixed** layout contract — overlaps **3.a″**; keep so page strategy isn’t blocked solely by RTE/WebView sizing. See [ideas/rich-text-display.md](../ideas/rich-text-display.md).

### 4.5 MAUI + “islands” of Skia **and** WebView

Compose strategy: Skia for dense lists/grids; WebView for HTML bodies; Syncfusion MAUI for PDF/RTE/ImageEditor modals.

| Pros | Cons |
| --- | --- |
| Attacks each top issue with the right tool | Architectural complexity; clear ownership rules needed |
| Matches evidence from suite + product pain | Three UI stacks to teach and test |

---

## 5. Comparison snapshot (effort vs leverage)

| Option | Status | Effort (3 eng) | Perf upside | Core/.NET reuse | Rich HTML fit | Syncfusion critical controls |
| --- | --- | --- | ---: | --- | --- | --- |
| 1 Flutter full | Possible / **not considered** | Very high | Very high | Low | Good if designed in | PDF Viewer available; RTE/ImageEditor = alts |
| 2 Avalonia full | Possible / **not considered** | Very high | High | High | Via WebView/HTML | TBD per platform |
| 3.a WebView pages | **Considered** | Medium–high | High (create) | High shell | Excellent | Keep MAUI Syncfusion islands |
| 3.a′ Blazor Hybrid | **Considered** | Medium–high | High (create)* | High | Good | Keep MAUI Syncfusion islands |
| 3.a″ HTML regions only | **Considered** | Low–medium | Low for page-open; helps rich text | Full | Excellent (display) | Unchanged |
| 3.b SkiaUi | **Considered** | High (framework) | Very high if complete | High | Poor alone | Keep MAUI islands |
| 4.1 MAUI tune (hygiene only) | Hygiene | Low | Insufficient alone (already tried) | Full | Unchanged | Full |
| **4.2 Flutter add-to-app** | **Backup** (partial) | Medium–high | High on islands | Medium | Per island | Mixed / keep MAUI Syncfusion |
| 4.3 Uno / native islands | Possible / **not considered** | High | Varies | Varies | Varies | TBD |
| 4.5 Skia + WebView + MAUI | **Considered** | Medium–high | High | High | Good | Full |

---

## 6. Suggested evaluation order (proposal)

1. **Clarify success metrics** (page-open ms, scroll FPS, rich-text layout stability, memory after N cycles, **Figma visual parity** on iOS and Android).
2. **Parallel spikes (time-boxed) — currently considered only:**
   - One SkiaUi (or raw Skia) list/page vs same in MAUI (**style to a real Figma component**).
   - One **full-page** WebView (3.a) *or* **Blazor Hybrid** page (3.a′) using a real hotspot screen (CSS/tokens vs Figma).
   - Harden **HTML regions only** (3.a″): one detail WebView vs auto-height-in-list; sizing + dispose metrics.
3. **Decide** among MAUI hybrids (3.a / 3.a′ / 3.b / 4.5) using spike data + [team](team.md) capacity — not suite averages alone.
4. Keep light **4.1** hygiene and incremental **3.a″** where they help rich text — do **not** treat tree/virtualization tuning as the strategic path (already insufficient).
5. Hold **4.2 Flutter add-to-app** as **partial backup** if MAUI hybrids miss targets.
6. **Do not** plan full Flutter / Avalonia / Uno migration unless product explicitly reopens scope (§1 / §2 remain parking lot).

\*Blazor Hybrid create cost is still WebView-bound; measure first paint separately from MAUI handler create.

---

## Related

- Decision tree: [../decision-tree.md](../decision-tree.md)
- SkiaUi requirements: `/Users/rkukla/proj/cs/maui/SkiaUi/Requirements.md`
- Toolkit last-resort notes: [toolkit-strategy.md](toolkit-strategy.md)
