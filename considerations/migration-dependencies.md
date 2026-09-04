# Dependencies to reconsider outside MAUI

Inventory from `thinktime-xamarin` PackageReferences and notable platform usage.

**Note:** Full app migration is **out of scope for now**. This matrix stays as reference if Flutter / Avalonia / other non-MAUI hosts are ever reconsidered ([options.md](options.md) parking lot). For current work, focus on MAUI-hosted hybrids and Syncfusion-on-MAUI.

**Legend — migration risk**

| Risk | Meaning |
| --- | --- |
| **Blocker** | No clear equivalent or large rewrite / data migration |
| **High** | Commercial or platform feature; must validate vendor + feature parity before committing |
| **Medium** | Ecosystem exists; still port/integration work |
| **Low** | Portable .NET or trivial platform API swap |

Availability notes below are directional (verify at decision time)—especially commercial **Syncfusion** SKUs per toolkit.

**Planned vendor change:** **Telerik is being removed**; remaining Telerik usage (rich text editor and any other Rad\* controls) will be **replaced with Syncfusion**. Treat migration planning as **Syncfusion-centric**, not dual-vendor.

---

## 1. Must-have product capabilities (stated)

| Dependency / capability | Today (MAUI) | Flutter | Avalonia (or other .NET UI) | Risk |
| --- | --- | --- | --- | --- |
| Offline DB | **Realm** 20.x (.NET) | Realm Flutter / Dart SDK (schema + sync model rewrite) | Realm .NET often reusable if data layer stays C# | **High** (schema/API + sync semantics) |
| PDF viewer | Syncfusion MAUI PdfViewer | Syncfusion Flutter PDF Viewer — verify feature parity + theming | Syncfusion or other .NET PDF viewer if offered for target UI | **High** |
| Rich text editor | **→ Syncfusion** RTE (Telerik RadRichTextEditor being removed; WebView sizing issues may follow the replacement) | Syncfusion Flutter **RTE: not available**; alts TBD | Syncfusion Avalonia or embed WebView/HTML editor | **High** (fidelity + sizing) |
| Image editor | Syncfusion MAUI ImageEditor | Syncfusion Flutter Image Editor **not available**; alts TBD | .NET image editor control or custom Skia | **High** |

These three UI controls plus Realm are the first gate for any toolkit switch.

---

## 2. Heavy MAUI-bound UI libraries (beyond the big three)

Widely used in XAML/code-behind—not optional “nice to have” without a replacement plan.

| Package / area | Role in app | Non-MAUI note | Risk |
| --- | --- | --- | --- |
| **Syncfusion.Maui.ListView** | Primary list control across many screens | Need Flutter/Avalonia list with similar grouping/templates | **High** |
| Syncfusion Calendar, Picker, Popup, PullToRefresh, Carousel, Buttons, Charts, Gauges | Date UX, charts, chrome | Syncfusion has multi-platform suites—confirm SKU | **High** |
| Syncfusion **SignaturePad** | Custom form signatures | Common elsewhere; re-integrate | **Medium** |
| **Telerik.UI.for.Maui** (legacy) | e.g. RadRichTextEditor, `RadCollectionView`, `RadPieChart` | **Planned removal** — replace with Syncfusion; do not plan Flutter/Avalonia around Telerik | **N/A** (exiting) |
| **FFImageLoading.Maui** | Cached images in lists/forms | `cached_network_image` (Flutter); Avalonia image cache libs | **Medium** |
| **CommunityToolkit.Maui** | Behaviors, popups, helpers | Rewrite against target toolkit idioms | **Medium** |
| **CommunityToolkit.Maui.MediaElement** | Video preview | `video_player` / Avalonia media | **Medium** |
| **Indiko.Maui.Controls.Markdown** + **Markdig** | Markdown UI | Markdig portable; UI control is not | **Medium** |
| **SkiaSharp** (+ Views.Maui) | Drawing / forced latest Skia | SkiaSharp OK on Avalonia; Flutter uses Skia natively via engine | **Low–Medium** |
| Custom **WebViewExtended** / auth WebViews / auto-size workarounds | Rich HTML display, KB, comments, embedded web modules | Re-implement platform WebView bridges; sizing issue follows | **High** |
| Embedded **web app** surfaces (Scheduling, T&A, Statistics, etc.) | MAUI WebView hosts existing web UI | Same idea on any toolkit—auth cookies/deep links must port | **Medium** |

---

## 3. Data, sync, and portable .NET (Avalonia-friendly; Flutter = rewrite or FFI)

| Package / area | Role | Flutter | Avalonia / .NET UI | Risk |
| --- | --- | --- | --- | --- |
| **Realm** | Offline store, ~dozens of Realm model types | Port models to Dart Realm or keep .NET via embedding (unusual) | Likely keep C# Realm layer | **High** / **Medium** |
| **Newtonsoft.Json** | API/serialization | `json` / codegen | Keep | **Low** |
| **Mapster** + codegen (`generate_maps.sh`) | DTO ↔ domain mapping | Manual / Dart mappers | Keep | **Medium** (Flutter) / **Low** (Avalonia) |
| **FastEnum** | Enum ↔ string for Realm/API | Dart enums | Keep | **Low–Medium** |
| **System.IdentityModel.Tokens.Jwt** | Tokens | Flutter JWT pkgs | Keep | **Low** |
| **HtmlAgilityPack** / **HtmlSanitizer** | HTML processing | Dart HTML libs | Keep | **Low–Medium** |
| **CsvHelper** | CSV | Dart CSV | Keep | **Low** |
| **MetadataExtractor** | Photo EXIF/metadata | Dart/native metadata libs | Keep | **Medium** |
| **System.Reactive** | Rx patterns | `rxdart` etc. | Keep | **Medium** / **Low** |
| **Binbin.Linq.PredicateBuilder** / Dynamic LINQ | Query building | Redesign queries | Keep | **Medium** / **Low** |
| NSwag-generated **ThinkTime.Api** clients | HTTP API surface | Regenerate OpenAPI for Dart **or** keep .NET service process | Keep clients in C# | **High** (Flutter) / **Low** (Avalonia) |

**Implication:** Avalonia (or any .NET UI) can reuse much of `ThinkTime.Core` / `ThinkTime.Api`. Flutter implies rewriting or isolating that layer.

---

## 4. Platform / device integrations

| Dependency | Role | Non-MAUI note | Risk |
| --- | --- | --- | --- |
| **Xamarin.Firebase.Messaging** (+ `google-services.json`) | Android push | `firebase_messaging` / native | **Medium** |
| iOS push (`PushNotificationProvider`, APNs) | Push | Flutter firebase / native | **Medium** |
| **Xamarin.ShortcutBadger** | Android launcher badges | Platform channel / plugin | **Medium** |
| **Plugin.BLE** | BLE (e.g. temperature probes in custom forms) | `flutter_blue_plus` etc.; Avalonia needs .NET BLE | **High** |
| **NET-iOS.GMImagePicker** | iOS multi-image pick | `image_picker` / PHPicker | **Medium** |
| Android custom JARs (`render-thread-animators`, `hiddenapibypass`) | Animation / hidden API workaround | Likely MAUI/Android-specific; re-validate need | **Medium** |
| SSO / SAML / external browser auth | Login | Same OAuth/SAML flows; WebView/browser glue differs | **High** |
| Deep linking | Navigation into features | Per-toolkit routing | **Medium** |
| Permissions, camera, file share | Attachments / media | Standard plugins exist | **Medium** |

---

## 5. Observability & ops

| Dependency | Role | Non-MAUI note | Risk |
| --- | --- | --- | --- |
| **Sentry.Maui** | Crash/perf | Sentry Flutter / Sentry .NET for Avalonia | **Low–Medium** |
| Multi-**Overlay** customer builds | Branding/config | Rebuild pipeline for new toolkit | **High** (process) |
| DevOps APK/IPA upload, Google Storage scripts | Release | Keep; change build targets | **Medium** |

---

## 6. Decision cheat-sheet

```text
Full app migration off MAUI?
│
├─ Currently: OUT OF SCOPE (possible / not considered)
│  • Flutter full, Avalonia full, Uno/native — see options.md §1/§2/§4.3
│  • Reopen only if MAUI-hosted strategies fail product goals
│
└─ Stay on MAUI (currently considered)
   • WebView / Blazor / SkiaUi / composition for hotspots
   • Optional partial Flutter add-to-app backup (not full rewrite)
   • Keeps Syncfusion/Realm on MAUI; aligns with current scope
```

## 7. Scan sources (for refresh)

- `ThinkTime.App/ThinkTime.App.csproj`, `ThinkTime.Core/ThinkTime.Core.csproj`
- `ThinkTime.Android/ThinkTime.Android.csproj`, `ThinkTime.iOS/ThinkTime.iOS.csproj`
- `Version.props` (`SyncfusionVersion`, `MauiForcedVersion`; `TelerikVersion` until removal completes)
- Usage of Syncfusion RTE/PdfViewer/ImageEditor/ListView, residual `RadRichTextEditor*`, `WebViewExtended`, `Realms`, `Plugin.BLE`, Firebase providers

Re-run a PackageReference grep when refreshing this doc after major upgrades.
