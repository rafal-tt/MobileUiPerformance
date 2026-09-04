# Rich text display without fragile WebView sizing

**Status:** Draft  
**Targets:** Correct, stable layout for HTML/rich content; fewer platform workarounds.

## Problem

ThinkTime must show rich text. `WebView` does not naturally expose **content-driven size** to the native layout system. Existing workarounds (measure HTML height via JS, resize host, etc.) are imperfect.

## Constraints any solution must meet

- Height (and sometimes width) must follow content inside ScrollViews / list cells.
- Must not defeat list virtualization (expensive or async measure per cell is risky).
- Acceptable fidelity for the HTML/subset we actually render.
- Must not worsen leak story (WebView is a heavy native peer).

## Direction options (to evaluate)

| Direction | Pros | Cons |
| --- | --- | --- |
| Keep WebView + harden measure protocol | Familiar; full HTML | Still fighting the control model; flaky layout |
| Native attributed / spannable text | Intrinsic size; lighter | Limited HTML; more mapping work |
| Skia / DrawnUI text + limited markup | Fast create; one surface | Markup feature gap; a11y/input |
| Hybrid: plain/native for lists, WebView only for full-screen docs | Isolates sizing pain | Two pipelines |

## Measure

- Layout stability (no jump after load) on sample rich items in a list and on a detail page.
- Time to final correct height; scroll hitch rate when cells contain rich text.
- Instance counts of WebView / native web peers after navigate away.

## Open

- What HTML subset is actually required in-product?
- Which screens need inline rich text vs full-page only?
