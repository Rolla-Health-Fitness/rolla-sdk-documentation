# Branding & Modules

This section covers custom branding options and module configuration.

## Custom Branding (Optional)

```swift
let branding = RollaBranding(
    appName: "Your App Name",
    primaryColor: .systemBlue,
    secondaryColor: .systemGray,
    accentColor: .systemGreen,
    brightness: "light",  // or "dark"
    defaultThemeMode: "system",  // "light", "dark", or "system"
    defaultLocale: "en",  // Optional
    headerLogoAsset: nil,  // Optional: partner logo asset path (provided by Rolla)
    termsUrl: "https://example.com/terms",  // Optional
    privacyUrl: "https://example.com/privacy"  // Optional
)

let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    branding: branding
)
```

## Branding Assets

Image assets used by the SDK (such as partner logos referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. This means any custom logos, icons, or images need to be provided to Rolla in advance so they can be included in your SDK build.

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG format preferred) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

## Rolla Band References

The SDK can refer to the paired wearable either generically ("fitness device") or specifically as the "Rolla Band" throughout its UI. This is controlled by the `removeRollaBandReferences` flag on `RollaConfiguration`:

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    removeRollaBandReferences: true  // Default — generic "fitness device" wording
)
```

- `true` (**default**) — the SDK uses generic "fitness device" wording. This is the right choice for most partner apps, which pair with their own-branded or third-party wearables rather than a Rolla-branded band.
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

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Apple Health Integration](06-apple-health.md) | **Home:** [README](README.md)
