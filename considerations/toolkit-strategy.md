# Toolkit strategy

Canonical options write-up: [options.md](options.md).

## Scope

- **Out of scope (for now):** full app migration (Flutter, Avalonia, Uno, native rewrite of the whole UI).
- **Possible but not considered:** those full-migration paths — documented for later; no active planning.
- **Currently considered:** stay on MAUI; WebView / Blazor / SkiaUi / composition; optional **partial** Flutter add-to-app backup.

## Currently considered (summary)

1. MAUI incremental — WebView pages, Blazor Hybrid, HTML regions only, Skia-drawn UI / SkiaUi
2. Skia + WebView + MAUI composition
3. Light MAUI hygiene (already insufficient alone)
4. **Backup:** Flutter add-to-app islands (partial — not full rewrite)

## Parking lot (full migration)

- Flutter full/large migration
- Avalonia full/large migration
- Uno / native full rewrites

Revisit only if MAUI-hosted strategies cannot meet goals **and** product reopens scope.

## Constraints

- Existing ThinkTime codebase: `/Users/rkukla/proj/thinktime/thinktime-xamarin` — see [app.md](app.md)
- Dependency notes (useful if scope reopens): [migration-dependencies.md](migration-dependencies.md)
- Team: 3 engineers, strong C# / MAUI, open to learning — see [team.md](team.md)
- Release cadence, shared business logic, offline/sync requirements.

## Decision inputs needed

- Target ms for page open and scroll on reference devices.
- Which screens are in scope for non-MAUI **islands** (not full rewrite).
- Acceptable fidelity and accessibility bar (Figma).
