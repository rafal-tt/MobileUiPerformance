# Decision tree — UI performance improvements

Use this when choosing where to spend effort. Update branches as experiments land.

**Scope:** Full app migration (Flutter / Avalonia / Uno / native rewrite) is **out of scope for now** — [options.md](considerations/options.md) parking lot. Decide among **MAUI-hosted** approaches.

Start from the symptom ([top-issues](findings/top-issues.md)):

```text
Which top issue?
│
├─ Rich text sizing / WebView workarounds
│  → ideas/rich-text-display.md
│    (do not assume “use WebView because create is fast in the suite”)
│    → options §3.a″ (HTML regions) / §3.a′ / §3.a as needed
│
├─ Suspected memory leak / growth over navigation
│  → ideas/memory-leak-diagnosis.md
│    (pair managed + native evidence before large rewrites)
│
└─ Slow page open and/or non-fluent scroll
   → performance subtree below
```

## Already tried (not enough)

**Visual-tree reduction** and **list virtualization** are already practiced in the product. Keep as ongoing hygiene; they do **not** close the create/scroll gap. Strategic work should assume density remains and move to cheaper rendering ([options.md](considerations/options.md) §3+).

## Performance subtree (issues 1–2)

```text
Is the screen dominated by many MAUI VisualElements
(each needing a platform handler)?
│
├─ NO → Profile again (bindings, images, I/O, GC, leaks).
│        Fix that bottleneck; re-measure.
│
└─ YES → Avoid 1:1 native views for dense UI
         (tree shrink / virtualization already insufficient)
         │
         ├─ Prefer C# drawn UI / Figma control?
         │  → SkiaUi / DrawnUI / custom handlers
         │    → ideas/drawn-or-custom-handlers.md
         │    → options §3.b
         │
         ├─ Prefer HTML/CSS or existing web content?
         │  → WebView page / Blazor Hybrid / HTML regions
         │    → ideas/skia-or-webview-surface.md
         │    → ideas/rich-text-display.md
         │    → options §3.a / 3.a′ / 3.a″
         │
         ├─ Mix by concern?
         │  → Skia lists + WebView HTML + MAUI Syncfusion modals
         │    → options §4.5
         │
         └─ MAUI hybrids still miss targets on hotspots?
            → Partial backup: Flutter add-to-app islands
              → options §4.2
            → Full Flutter/Avalonia migration remains
              possible but **not considered** unless scope reopens
              → options §1 / §2 (parking lot)
```

## Strategic options

See [considerations/options.md](considerations/options.md). **In scope:** MAUI WebView / Blazor / Skia / composition. **Backup:** Flutter add-to-app (§4.2). **Parking lot:** full Flutter / Avalonia / Uno.

## Priority order (default)

1. **Measure** the specific ThinkTime page/list (handlers vs layout vs bind vs data).
2. **Replace dense subtrees** with drawn/Skia/WebView/Blazor where Figma fidelity allows.
3. **Leak hygiene** whenever adding or retaining native peers (especially WebView / custom handlers).
4. **Framework upgrade** (MAUI 11+) as a parallel track; do not assume it alone closes the gap.
5. **Partial** Flutter add-to-app backup only if MAUI hybrids miss goals.
6. Full toolkit migration: **out of scope** — do not plan unless explicitly reopened.
7. Continue tree/virtualization hygiene only as background practice — not the strategic bet.

## Exit criteria for a branch

- Reproducible before/after on UiPerformanceTest *and* a ThinkTime scenario.
- Documented UX/fidelity (incl. Figma) and maintenance cost.
- Explicit “stop / continue” note in the idea’s markdown file.
