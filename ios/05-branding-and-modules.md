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

## Module Configuration

The SDK is organized into modules. Currently, all modules are always enabled — pass `nil` for the `modules` parameter (or omit it entirely).

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    modules: nil  // All modules enabled (currently the only supported option)
)
```

Selective module enablement will be available in a future release. Once you see the full feature set, you can tell us which modules you want enabled or disabled and we will implement per-partner module configuration.

### Available Modules

The SDK is organized into the following modules. Module names are **camelCase** strings and must match exactly if passed in the `modules` array.

| Module | Description |
|--------|-------------|
| `metrics` | Health metrics dashboard (heart rate, HRV, steps, calories, distance) |
| `weight` | Weight tracking with BMI and targets |
| `bloodPressure` | Blood pressure tracking and manual logging |
| `activityTracking` | Live activity tracking with GPS and heart rate |
| `activityReview` | Activity history and detailed review |
| `goals` | Daily health goals (steps, sleep, active points) |
| `insights` | AI-generated personalized health insights |
| `scores` | Readiness and activity scores |
| `profile` | User profile management |
| `bandPairing` | Bluetooth band discovery and pairing |
| `bandSync` | Band data synchronization |
| `bandFirmware` | Band firmware update management |
| `settings` | App settings (theme, language, units, permissions) |
| `authentication` | User authentication and session management |
| `branding` | App branding and theming |
| `consent` | User consent management |
| `fabMenu` | Floating action button menu |
| `home` | Home dashboard |
| `onboarding` | User onboarding flow and initial setup |
| `permissions` | Runtime permissions management |
| `support` | Support and help |
| `integrations` | External integrations (Apple Health, Garmin, Oura) |
| `debugLogs` | Band diagnostic logs |

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Apple Health Integration](06-apple-health.md) | **Home:** [README](README.md)
