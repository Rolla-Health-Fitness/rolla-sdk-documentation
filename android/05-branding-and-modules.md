# Branding & Modules

Customize the SDK appearance through branding configuration and manage which modules are enabled in your integration.

## Custom Branding (Optional)

Colors are passed as ARGB integers (e.g. `0xFF6750A4.toInt()`):

```kotlin
val branding = RollaBranding(
    appName = "Your App Name",
    primaryColor = 0xFF1976D2.toInt(),         // Blue
    secondaryColor = 0xFF625B71.toInt(),       // Gray
    accentColor = 0xFF7D5260.toInt(),          // Accent
    brightness = "light",                       // or "dark"
    defaultThemeMode = "system",               // "light", "dark", or "system"
    defaultLocale = "en",                      // Optional
    headerLogoAsset = null,                    // Optional: partner logo asset path (provided by Rolla)
    termsUrl = "https://example.com/terms",    // Optional
    privacyUrl = "https://example.com/privacy" // Optional
)

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    branding = branding
)
```

> **Branding assets:** Image assets (such as partner logos) must be pre-bundled inside the SDK — they cannot be transferred from the host app at runtime. During onboarding, coordinate with Rolla to supply your partner logo (SVG format preferred) and any other brand assets. Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

## Module Configuration

The SDK is organized into modules. Currently, all modules are always enabled — pass `null` for the `modules` parameter (or omit it entirely).

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    modules = null  // All modules enabled (currently the only supported option)
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

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
