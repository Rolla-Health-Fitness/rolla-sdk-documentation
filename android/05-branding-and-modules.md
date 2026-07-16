# Branding & Modules

Customize the SDK appearance through branding configuration and control which modules are disabled in your integration.

## Custom Branding (Optional)

Every `RollaBranding` field is optional: a set field overrides the SDK's built-in default individually, and an unset field keeps it. Passing no branding at all keeps the complete default look. Colors are passed as ARGB integers (e.g. `0xFF6750A4.toInt()`):

```kotlin
val branding = RollaBranding(
    hostAppName = "Your App Name",              // names your app in SDK copy — see below
    primaryColor = 0xFF1976D2.toInt(),          // seeds the SDK's entire color scheme
    defaultThemeMode = RollaThemeMode.SYSTEM,   // SYSTEM, LIGHT, or DARK
    headerLogoAsset = null,                     // partner logo asset path (provided by Rolla)
    privacyUrl = "https://example.com/privacy"  // privacy link on the consent screen
)

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    branding = branding
)
```

- **`hostAppName`** — your app's display name. When set, SDK copy that refers to the app names it explicitly — the consent screen's legal intro and the battery-optimization / motion-permission prompts — in every SDK language. Unset keeps the generic "this app" wording.
- **`primaryColor`** — seeds the whole SDK color scheme (buttons, navigation, inputs, charts, share cards) in both light and dark themes; it is not just an accent.
- **`defaultThemeMode`** — the theme the SDK UI starts in, until the user picks a theme inside SDK settings.
- **`headerLogoAsset`** — path of your logo inside the SDK bundle (see the branding-assets note below).
- **`privacyUrl`** — your privacy policy, linked from the consent screen's "privacy policy" text. Unset keeps the SDK's default policy link.

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

## Data Source Configuration

By default the SDK offers every data source the user can connect (Rolla Band, Garmin, Oura, Apple Health, Health Connect). To hide specific sources, pass their `RollaDataSource` values in the `disabledDataSources` set (or omit the parameter / pass `emptySet()` to offer everything):

```kotlin
import com.rolla.sdk.wrapper.RollaDataSource

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    disabledDataSources = setOf(
        RollaDataSource.GARMIN,
        RollaDataSource.OURA,
        RollaDataSource.HEALTH_CONNECT
    )
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
| `RollaDataSource.BAND` | The Rolla Band pairing option |
| `RollaDataSource.GARMIN` | Garmin Connect |
| `RollaDataSource.OURA` | Oura |
| `RollaDataSource.APPLE_HEALTH` | Apple Health (iOS only) |
| `RollaDataSource.HEALTH_CONNECT` | Health Connect |

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
