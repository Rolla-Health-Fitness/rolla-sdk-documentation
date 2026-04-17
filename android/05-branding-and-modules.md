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

The SDK is organized into the following modules. Module names are **camelCase** strings and must match exactly if passed in the `modules` list.

**Health & Fitness**

| Module | Description |
|--------|-------------|
| `metrics` | Health metrics dashboard (heart rate, HRV, steps, calories, distance) |
| `weight` | Weight tracking with BMI and targets |
| `bloodPressure` | Blood pressure tracking and manual logging |
| `goals` | Daily health goals (steps, sleep, active points) |
| `insights` | AI-generated personalized health insights |
| `scores` | Readiness and activity scores |

**Activity Tracking**

| Module | Description |
|--------|-------------|
| `activityTracking` | Live activity tracking with GPS and heart rate |
| `activityReview` | Activity history and detailed review |

**Device Management**

| Module | Description |
|--------|-------------|
| `bandPairing` | Bluetooth band discovery and pairing |
| `bandSync` | Band data synchronization |
| `bandFirmware` | Band firmware update management |

**User & Settings**

| Module | Description |
|--------|-------------|
| `profile` | User profile management |
| `settings` | App settings (theme, language, units, permissions) |
| `authentication` | User authentication and session management |
| `consent` | User consent management |
| `onboarding` | User onboarding flow and initial setup |
| `permissions` | Runtime permissions management |

**UI & Navigation**

| Module | Description |
|--------|-------------|
| `home` | Home dashboard |
| `fabMenu` | Floating action button menu |
| `branding` | App branding and theming |
| `support` | Support and help |

**Integrations & Diagnostics**

| Module | Description |
|--------|-------------|
| `integrations` | External integrations (Apple Health, Garmin, Oura) |
| `debugLogs` | Band diagnostic logs |

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
