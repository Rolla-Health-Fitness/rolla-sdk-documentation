# Branding & Modules

This section covers custom branding options and module configuration.

## Custom Branding (Optional)

Every `RollaBranding` field is optional: a set field overrides the SDK's built-in default individually, and an unset field keeps it. Passing no branding at all keeps the complete default look.

```swift
let branding = RollaBranding(
    hostAppName: "Your App Name",              // names your app in SDK copy — see below
    primaryColor: .systemBlue,                 // seeds the SDK's entire color scheme
    defaultThemeMode: .system,                 // RollaThemeMode: .system, .light, or .dark
    headerLogoAsset: nil,                      // partner logo asset path (provided by Rolla)
    privacyUrl: "https://example.com/privacy"  // privacy link on the consent screen
)

let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    branding: branding
)
```

- **`hostAppName`** — your app's display name. When set, SDK copy that refers to the app names it explicitly — the consent screen's legal intro and the battery-optimization / motion-permission prompts — in every SDK language. Unset keeps the generic "this app" wording.
- **`primaryColor`** — seeds the whole SDK color scheme (buttons, navigation, inputs, charts, share cards) in both light and dark themes; it is not just an accent.
- **`defaultThemeMode`** — the theme the SDK UI starts in, until the user picks a theme inside SDK settings.
- **`headerLogoAsset`** — path of your logo inside the SDK bundle (see Branding Assets below).
- **`privacyUrl`** — your privacy policy, linked from the consent screen's "privacy policy" text. Unset keeps the SDK's default policy link.
- **`removeRollaBandReferences`** — whether the SDK uses generic "fitness device" wording or Rolla Band-specific naming. See [Rolla Band References](#rolla-band-references) below.

## Branding Assets

Image assets used by the SDK (such as partner logos referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. This means any custom logos, icons, or images need to be provided to Rolla in advance so they can be included in your SDK build.

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG format preferred) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

## Rolla Band References

The SDK can refer to the paired wearable either generically ("fitness device") or specifically as the "Rolla Band" throughout its UI. This is controlled by the `removeRollaBandReferences` flag on `RollaBranding`:

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    branding: RollaBranding(
        removeRollaBandReferences: true  // Default — generic "fitness device" wording
    )
)
```

- `true` (**default**, also when unset) — the SDK uses generic "fitness device" wording. This is the right choice for most partner apps, which pair with their own-branded or third-party wearables rather than a Rolla-branded band.
- `false` — the SDK shows Rolla Band-specific references (naming, imagery, and copy that call out the Rolla Band by name).

## Module Configuration

By default every module is enabled. To hide a module's entire UI everywhere it appears in the SDK, pass its `RollaDisabledModule` value in the `disabledModules` set (or omit the parameter / pass `[]` to keep everything enabled):

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    disabledModules: [.weight, .bloodPressure]
)
```

### Disable-able Modules

`disabledModules` accepts the following `RollaDisabledModule` values. These are the modules that can currently be turned off per integration:

| Value | Hides |
|-------|-------|
| `.weight` | The Weight tracking module (weight logging, BMI, and targets) |
| `.bloodPressure` | The Blood Pressure tracking and manual-logging module |

Additional modules will become disable-able in future releases. If there is a module you need to hide that isn't listed yet, contact Rolla during onboarding and we will prioritize adding it to `RollaDisabledModule`.

## Data Source Configuration

By default the SDK offers every data source the user can connect (Rolla Band, Garmin, Oura, Apple Health, Health Connect). To hide specific sources, pass their `RollaDataSource` values in the `disabledDataSources` set (or omit the parameter / pass `[]` to offer everything):

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    disabledDataSources: [.garmin, .oura, .appleHealth]
)
```

A hidden source's connect option is suppressed everywhere the user picks a source to connect — the Data Sources screen and the onboarding data-source step. This is useful when you want to route users toward a specific source: disabling everything except the band, for example, sends users straight to the "Pair your band" flow.

### Behavior notes

- **Deny-list semantics.** An empty set (the default) offers every source. Each value present hides that source. This matches `disabledModules`.
- **Already-connected sources stay visible.** If a user has already connected a source that you later disable, it still appears on the Data Sources screen so they can view or disconnect it — only offering a *new* connection is suppressed.
- **The band is a safety floor.** At least one source is always connectable. If you disable *every* source, the SDK keeps the Rolla Band available so onboarding never dead-ends.
- **Band-only skips the picker.** If the Rolla Band is the only source left enabled, there is no data-source selection screen — onboarding takes the user straight to the band pairing screen (the gate becomes band pairing instead of source choosing), and the Data Sources entry is hidden from Settings. Band status and unpairing remain available from the band button on the Home screen.

### Disable-able Data Sources

`disabledDataSources` accepts the following `RollaDataSource` values:

| Value | Hides |
|-------|-------|
| `.band` | The Rolla Band pairing option |
| `.garmin` | Garmin Connect |
| `.oura` | Oura |
| `.appleHealth` | Apple Health |
| `.healthConnect` | Health Connect (Android only) |

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Apple Health Integration](06-apple-health.md) | **Home:** [README](README.md)
