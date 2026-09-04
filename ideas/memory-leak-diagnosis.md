# Memory leak diagnosis (managed ↔ native)

**Status:** Draft  
**Targets:** Findable retention; fewer long-lived platform views after leave page / recycle cell.

## Problem

Each platform view has a native counterpart. We often cannot tell whether a managed instance stays alive **because of** a native peer (or handler/listener), or the native peer stays alive because of managed references. Leaks are hard to locate across that bridge.

## Working practices (to institutionalize)

1. **Pair counts:** After N open/close cycles of a page, compare managed view/handler counts vs native view hierarchy / Instruments / Android Studio native heap.
2. **Unsubscribe & clear:** Events, `MessagingCenter`/weak messengers, bindings, gestures—especially on cells and WebViews.
3. **Handler lifecycle:** Confirm handlers disconnect when visual elements leave the tree; watch custom handlers and workarounds.
4. **WebView / rich text hosts:** Explicit destroy/navigate-null patterns; avoid retaining JS bridge callbacks.
5. **Reproduce with a minimal page** that only has the suspected control before blaming the full screen.

## Tooling notes

- Managed: `dotnet-gcdump` / Visual Studio memory, MAUI handler diagnostics if available.
- Android: Android Studio Memory Profiler (Java/Kotlin peers), LeakCanary for native Activity/Fragment leaks where applicable.
- iOS: Instruments Allocations / Leaks; watch `UIView` / `WKWebView` retention.

Cross-heap correlation is manual today—document suspected pairs when filing findings.

## Success signal

- Stable memory after repeated navigation/scroll scenarios on reference devices.
- Documented “how we proved the root” for each fixed leak class (so the next one is faster).
