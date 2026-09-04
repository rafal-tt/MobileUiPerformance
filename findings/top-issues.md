# Top issues (ThinkTime mobile)

Product pain we are solving for. These are the primary drivers for this notes repo—not an exhaustive bug list.

## 1. Long UI creation time

Opening a new / complex page takes too long. Dominant cost is **creating control handlers** and running **initial layout**, not business logic or CLR microbenchmarks.

**Impact:** Slow navigation, delayed first paint / interactivity.  
**Related:** [baseline-measurements.md](../findings/baseline-measurements.md), [options.md](../considerations/options.md)  
**Note:** Visual-tree reduction alone is **already practiced and not enough**.

## 2. Non-fluent list scrolling

Scroll jank has the **same root cause** as (1): when new rows/cells appear, creating their UI (handlers + layout) exceeds the frame budget. List virtualization is **already practiced and not enough** while cells remain expensive to create.

**Impact:** Hitching / dropped frames on fling and when scrolling into new content.  
**Related:** [options.md](../considerations/options.md)

## 3. Rich text display (WebView sizing)

We need to show **rich text**. Platform `WebView` is a poor fit when the view must **adapt its height (or width) to content**—WebViews are not designed for content-driven intrinsic sizing inside native layout.

Workarounds exist in the app but are **imperfect** (sizing races, incorrect heights, layout thrash, scroll nesting issues, etc.).

**Impact:** Wrong/unstable layout around HTML content; fragile platform-specific hacks.  
**Tension with perf suite:** WebView can be *fast to create* for fixed-size surfaces, but that does not solve **content-sized** rich text in lists/pages.  
**Related:** [ideas/rich-text-display.md](../ideas/rich-text-display.md)

## 4. Hard-to-locate memory leaks

Every MAUI/platform view has a **native counterpart**. Retention can be owned from either side:

- managed view kept alive by a native view / handler / platform listener, or
- native view kept alive by a managed reference (handler, event, binding, parent tree).

That dual ownership makes leak hunting difficult: profilers on one heap do not show why the other side stays alive, and cycles can span the bridge.

**Impact:** Growing memory over navigation/scroll; hard root-cause analysis; possible contribution to later UI sluggishness.  
**Related:** [ideas/memory-leak-diagnosis.md](../ideas/memory-leak-diagnosis.md)

## How these connect

```text
Handler + layout cost ──► slow page open
         │
         └──► expensive cells ──► non-fluent scrolling

Rich text via WebView ──► sizing workarounds ──► more layout complexity / fragility
                         └──► extra native views ──► leak surface area

Managed ↔ native pairing ──► opaque retention ──► leaks hard to attribute
```

Issues (1) and (2) share one performance thesis. Issues (3) and (4) are product/architecture constraints that any “faster UI” path (DrawnUI, Skia, WebView, custom handlers) must explicitly address—not treat as afterthoughts.
