# Branding & Modules

This section covers custom branding options and module configuration.

## 5. Configuration Options

### 5.1 Custom Branding (Optional)

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

### 5.2 Branding Assets

Image assets used by the SDK (such as partner logos referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. This means any custom logos, icons, or images need to be provided to Rolla in advance so they can be included in your SDK build.

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG format preferred) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

## 6. Module Configuration

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

The SDK contains 23 modules providing comprehensive fitness and health tracking functionality:

| Module | Description |
|--------|-------------|
| activity | Activity tracking, workout types (walk, run, cycling, cardio), activity history |
| metrics | Health metrics display (heart rate, HRV, steps, active energy, distance, speed, cadence, power) |
| integrations | Third-party health data integrations (Apple Health, Bluetooth devices, band sync) |
| nutrition | Nutrition tracking (meals, calories, macro/micronutrients, barcode scanning) |
| hydration | Hydration tracking and reminders |
| sleep | Sleep tracking, analysis, and recommendations |
| mindfulness | Guided meditation, breathing exercises, relaxation sessions |
| challenges | Social challenges, leaderboards, achievement tracking |
| community | Social feed, activity sharing, friend interactions |
| profile | User profile management, preferences, settings |
| devices | Connected device management (bands, watches, scale) |
| goals | Goal setting, tracking, and progress visualization |
| insights | Personalized health insights and recommendations |
| workout-details | Detailed workout analytics and breakdown |
| heart-health | Cardiovascular metrics and analysis |
| body-composition | Weight tracking, body metrics, composition analysis |
| recovery | Recovery tracking and recommendations |
| training-plans | Guided training programs and plans |
| social-sharing | Share workouts and metrics across social platforms |
| notifications | In-app and push notifications for activity alerts |
| dashboard | Customizable home dashboard and overview |
| settings | App settings and SDK configuration interface |
| onboarding | User onboarding flow and initial setup |

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Apple Health Integration](06-apple-health.md) | **Home:** [README](README.md)
