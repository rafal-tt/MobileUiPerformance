# Open questions & constraints

## Open questions

1. Which ThinkTime pages/lists dominate user-facing lag (top N by traffic × severity)?
2. Exact split on product traces: handler create vs measure/arrange vs bind vs image decode?
3. Minimum UX fidelity for “drawn” or WebView replacements on those screens?
4. Is MAUI 11 a near-term production target, or is MAUI 10 the planning baseline?
5. Device matrix beyond Galaxy S9 (lower-end Android, current iOS)?
6. What HTML/rich-text subset is required, and where must it size intrinsically (inline list vs full page)?
7. Which leak scenarios are reproducible today (page cycle, list scroll, rich-text WebView)?
8. Which **in-scope** option(s) from [options.md](options.md) §3 / §4.2 / §4.5 get time-boxed spikes next? (Full migration §1/§2 stays parking lot.)

## Constraints

- Prefer measurable experiments via `UiPerformanceTest` + product instrumentation.
- Confluence page `4394024967` starts empty — publish from this repo when asked.
- Do not assume language microbenchmark wins equal UI wins.
- Team capacity and skills: [team.md](team.md) (3 engineers; C# / MAUI deep; willing to learn).
- App scale / offline / critical controls / **Figma-strict same iOS·Android look**: [app.md](app.md); non-MAUI dependency matrix: [migration-dependencies.md](migration-dependencies.md).
- Strategic options: [options.md](options.md) — **in scope:** MAUI hybrids; **parking lot:** full Flutter/Avalonia/etc. MAUI-based cons still TBD as spikes run.
- Any spike must include a **Figma fidelity check** (can we style this control/page to match?), not only performance ms.
