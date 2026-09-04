# App facts (ThinkTime mobile)

Source: product knowledge + scan of `/Users/rkukla/proj/thinktime/thinktime-xamarin` (csproj PackageReferences and usage).

## Stated product facts

| Fact | Detail |
| --- | --- |
| Scale | **Big and complex** (~4k `.cs`/`.xaml` files, ~440k lines of those files; multi-project solution) |
| Offline | Full **offline mode**; local data in **Realm** (`Realm` NuGet **20.1.0**) |
| Look & feel | **Same look and feel on iOS and Android** — not “native per platform” Material vs UIKit chrome |
| Design source of truth | Every UI **component look** and **page design** is **strictly defined in Figma** |
| Critical 3rd-party UI | **PDF viewer**, **Rich Text Editor**, **Image editor** — all on **Syncfusion** after planned **Telerik removal** |

## Design / styling constraint

Product UI is **pixel-specified**: Figma defines components and pages; engineering must reproduce that look, not adopt stock platform defaults.

**Implication for toolkit choice:** Prefer solutions where we can **fully style** (or fully draw) controls to match Figma — colors, typography, spacing, states, density — across **both** iOS and Android with one design language.

| Approach class | Fit for Figma-strict + cross-platform parity |
| --- | --- |
| Flutter / Avalonia / SkiaUi (own renderer) | Strong — one visual tree, high style control |
| WebView / Blazor Hybrid (HTML/CSS) | Strong if CSS design system mirrors Figma tokens |
| MAUI + heavily styled / custom controls | Possible, but native handlers + Syncfusion theming limits must be validated per control |
| Stock native-look MAUI / platform defaults | **Poor fit** — fights “same on iOS and Android” + Figma |
| Compose + SwiftUI islands (divergent native UI) | **Poor fit** unless both are custom-styled to the same Figma (double cost) |

Third-party controls (Syncfusion PDF / RTE / Image Editor, lists, etc.) are acceptable only if their **theming APIs** can hit Figma (or we wrap/replace the chrome). Unstyled vendor defaults are not enough.

## Stack snapshot

- **UI host:** .NET MAUI 10 (`MauiForcedVersion` 10.0.100), Android + iOS
- **Architecture:** MVVM (`CommunityToolkit.Mvvm`), layered `ThinkTime.Api` / `Core` / `App` / platform heads
- **Branding:** Many **Overlay/** customer variants (Caseys, PetSmart, Jumbo, …) — build/config matrix, not a separate app per brand; still under the same Figma/component system unless an overlay explicitly overrides
- **Min OS:** iOS 13+, Android API 28+

## Critical UI components (user-priority)

| Capability | Direction | Today (transitional) | Notes |
| --- | --- | --- | --- |
| Rich text **editing** | **Syncfusion** RichTextEditor | Still **Telerik** `RadRichTextEditor` (`Telerik.UI.for.Maui` **14.0.1**) in places | Custom `RadRichTextEditorFixed` + WebView resize/focus workarounds; **Telerik to be removed** |
| PDF **viewing** | **Syncfusion** PdfViewer | Syncfusion `Syncfusion.Maui.PdfViewer` (**34.2.5**) | Telerik PDF only in mem-leak test harness |
| Image **editing** | **Syncfusion** ImageEditor | Syncfusion ImageEditor | Used from task/photo flows |

### Vendor consolidation (planned)

**Remove all Telerik components** from the app. Anything currently on Telerik (notably rich text editing, plus any `RadCollectionView` / charts usage) will be **replaced with Syncfusion**.

After that consolidation, commercial UI dependency for migration planning is **Syncfusion-only** (+ Realm, platform plugins)—not a dual Syncfusion+Telerik stack.

## Related docs

- Dependency migration matrix: [migration-dependencies.md](migration-dependencies.md)
- Team capacity: [team.md](team.md)
- Top issues (incl. rich text / WebView sizing): [../findings/top-issues.md](../findings/top-issues.md)
