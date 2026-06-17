# Branding & Modules

Customize the SDK appearance through branding configuration and control which modules are disabled in your integration.

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

## Rolla Band References

The SDK can refer to the paired wearable either generically ("fitness device") or specifically as the "Rolla Band" throughout its UI. This is controlled by the `removeRollaBandReferences` flag on `RollaConfiguration`:

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    removeRollaBandReferences = true  // Default — generic "fitness device" wording
)
```

- `true` (**default**) — the SDK uses generic "fitness device" wording. This is the right choice for most partner apps, which pair with their own-branded or third-party wearables rather than a Rolla-branded band.
- `false` — the SDK shows Rolla Band-specific references (naming, imagery, and copy that call out the Rolla Band by name).

## Module Configuration

By default every module is enabled. To hide a module's entire UI everywhere it appears in the SDK, pass its `RollaDisabledModule` value in the `disabledModules` set (or omit the parameter / pass `emptySet()` to keep everything enabled):

```kotlin
import com.rolla.sdk.wrapper.RollaDisabledModule

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    disabledModules = setOf(
        RollaDisabledModule.WEIGHT,
        RollaDisabledModule.BLOOD_PRESSURE
    )
)
```

### Disable-able Modules

`disabledModules` accepts the following `RollaDisabledModule` values. These are the modules that can currently be turned off per integration:

| Value | Hides |
|-------|-------|
| `RollaDisabledModule.WEIGHT` | The Weight tracking module (weight logging, BMI, and targets) |
| `RollaDisabledModule.BLOOD_PRESSURE` | The Blood Pressure tracking and manual-logging module |

Additional modules will become disable-able in future releases. If there is a module you need to hide that isn't listed yet, contact Rolla during onboarding and we will prioritize adding it to `RollaDisabledModule`.

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
