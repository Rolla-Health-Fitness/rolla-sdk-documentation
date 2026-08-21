# Configuration

Everything you can shape about the SDK — branding, language, modules, data sources — is set through `RollaConfiguration`, the object you pass when creating a `Rolla` instance. This page is the complete configuration reference: the full parameter table first, then a closer look at each configurable surface.

> **Configuration is read at engine start.** Apart from tokens, which you can push live with `updateToken` (see [Token Management](06-token-management.md)), a `RollaConfiguration` applies for the engine's lifetime. To change branding, language, modules, or data sources, call `Rolla.destroyEngine()` and create a new `Rolla` with the new configuration — see [Engine Lifecycle](07-engine-lifecycle.md).

## RollaConfiguration

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `String` | Yes | — | JWT access token from `POST /api/login` |
| `partnerId` | `String` | Yes | — | Partner identifier provided by Rolla |
| `refreshToken` | `String?` | No | `null` | Refresh token for automatic credential renewal |
| `tokenExpiresIn` | `Int?` | No | `null` | Token lifetime in seconds. Note: iOS uses `TimeInterval` (Double) for this parameter |
| `userId` | `String?` | No | Extracted from JWT | User identifier for local data namespacing (per-user storage isolation); defaults to the `sub` claim in the JWT if not provided. Not sent as a request header |
| `environment` | `String` | No | `"rnd"` | Target environment: `"production"` or `"rnd"`. See [Code Integration](04-code-integration.md#environment-values) |
| `language` | `RollaLanguage?` | No | `null` (profile-driven) | Forces the SDK UI language for the engine's lifetime. See [Language](#language) |
| `disabledModules` | `Set<RollaDisabledModule>` | No | `emptySet()` (nothing disabled) | Modules whose entire UI is hidden across the SDK. See [Module Configuration](#module-configuration) |
| `disabledDataSources` | `Set<RollaDataSource>` | No | `emptySet()` (all offered) | Data sources whose connect option is hidden wherever the user picks a source to connect. See [Data Source Configuration](#data-source-configuration) |
| `branding` | `RollaBranding?` | No | `null` | Visual identity overrides — every field optional, per-field merge. See [Custom Branding](#custom-branding-optional) |
| `showOptionsButton` | `Boolean` | No | `true` | Render a three-dot options action at the trailing edge of the Home top app bar. Tapping it opens an "Options" bottom sheet with shortcuts to Data Sources, Edit Goals, Leaderboards, and FAQ (rows follow your configuration — e.g. disabling the Leaderboards module hides its row). Defaults to true because most partners need this entry point |
| `showGoalsSection` | `Boolean` | No | `false` | Show the user's goals at the bottom of the Home screen, with an edit action. See [Goals on Home](#goals-on-home) |

For the identity and auth essentials (`token`, `partnerId`, `environment`) and a minimal setup example, see [Code Integration](04-code-integration.md).

## Custom Branding (Optional)

Every `RollaBranding` field is optional: a set field overrides the SDK's built-in default individually, and an unset field keeps it. Passing no branding at all keeps the complete default look. Colors are passed as ARGB integers (e.g. `0xFF6750A4.toInt()`):

```kotlin
val branding = RollaBranding(
    hostAppName = "Your App Name",              // names your app in SDK copy — see below
    primaryColor = 0xFF1976D2.toInt(),          // seeds the SDK's entire color scheme
    themeMode = RollaThemeMode.SYSTEM,          // SYSTEM, LIGHT, or DARK
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
- **`themeMode`** — the theme the SDK UI runs in: light, dark, or following the device setting (system).
- **`headerLogoAsset`** — path of your logo inside the SDK bundle (see [Branding Assets](#branding-assets) below).
- **`privacyUrl`** — your privacy policy, linked from the consent screen's "privacy policy" text. Unset keeps the SDK's default policy link.
- **`removeRollaBandReferences`** — whether the SDK uses generic "fitness device" wording or Rolla Band-specific naming. See [Rolla Band References](#rolla-band-references) below.

> **Android notification channels are brand-neutral.** The notification channels the SDK creates appear under your app's name in the system settings with neutral names ("Important Alerts", "Engagement Tips") — see [Permissions → Notification Channels](03-permissions.md#notification-channels).

## Branding Assets

Image assets used by the SDK (such as partner logos referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. This means any custom logos, icons, or images need to be provided to Rolla in advance so they can be included in your SDK build.

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG format preferred) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

## Rolla Band References

The SDK can refer to the paired wearable either generically ("fitness device") or specifically as the "Rolla Band" throughout its UI. This is controlled by the `removeRollaBandReferences` flag on `RollaBranding`:

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    branding = RollaBranding(
        removeRollaBandReferences = true  // Default — generic "fitness device" wording
    )
)
```

- `true` (**default**, also when unset) — the SDK uses generic "fitness device" wording. This is the right choice for most partner apps, which pair with their own-branded or third-party wearables rather than a Rolla-branded band.
- `false` — the SDK shows Rolla Band-specific references (naming, imagery, and copy that call out the Rolla Band by name).

## Language

By default the SDK renders in the language of the user's backend profile (see the [Profile guide](../sdk-auth-api/03-profile.md)). To force a specific UI language instead, set `language`:

```kotlin
import com.rolla.sdk.wrapper.config.RollaLanguage

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    language = RollaLanguage.GERMAN
)
```

- **Authoritative when set.** The configured language wins for the engine's lifetime — persisted in-SDK picks and the backend profile language cannot override it.
- **Applied at engine start.** Changing the language requires destroying and recreating the engine, like any configuration change — see [Engine Lifecycle](07-engine-lifecycle.md).
- **Kept in sync with the backend.** When the configured language differs from the user's profile, the SDK writes it to the profile at startup, so backend-generated content (goal labels, insights) arrives in the same language as the SDK UI.
- **Unset keeps the profile-driven behavior.** With `language` unset, the SDK follows the profile's language — which your app can set via [`POST /api/setprofile`](../sdk-auth-api/03-profile.md) if you manage language selection server-side.

### RollaLanguage

| Value | Language |
|-------|----------|
| `RollaLanguage.ENGLISH` | English |
| `RollaLanguage.GERMAN` | German (Deutsch) |
| `RollaLanguage.SPANISH` | Spanish (Español) |
| `RollaLanguage.CROATIAN` | Croatian (Hrvatski) |
| `RollaLanguage.BOSNIAN` | Bosnian (Bosanski) |
| `RollaLanguage.SERBIAN_LATIN` | Serbian — Latin script (Srpski) |
| `RollaLanguage.SERBIAN_CYRILLIC` | Serbian — Cyrillic script (Српски) |
| `RollaLanguage.ARABIC` | Arabic (العربية) |

## Module Configuration

By default every module is enabled. To hide a module's entire UI everywhere it appears in the SDK, pass its `RollaDisabledModule` value in the `disabledModules` set (or omit the parameter / pass `emptySet()` to keep everything enabled):

```kotlin
import com.rolla.sdk.wrapper.config.RollaDisabledModule

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    disabledModules = setOf(
        RollaDisabledModule.WEIGHT,
        RollaDisabledModule.BLOOD_PRESSURE
    )
)
```

### RollaDisabledModule

`disabledModules` accepts the following `RollaDisabledModule` values. These are the modules that can currently be turned off per integration:

| Value | Hides |
|-------|-------|
| `RollaDisabledModule.WEIGHT` | The Weight tracking module (weight logging, BMI, and targets) |
| `RollaDisabledModule.BLOOD_PRESSURE` | The Blood Pressure tracking and manual-logging module |
| `RollaDisabledModule.LEADERBOARDS` | The Leaderboards module (weekly/monthly competitive rankings) |
| `RollaDisabledModule.INSIGHTS` | The Insights module (the insights feed, the Home screen's Insights entry, and its unread badge) |

Leaderboards let users compare their Health Score or Active Points against other users in your tenant over weekly and monthly periods, with join/leave controls per challenge type. Disable the module to hide competitive rankings everywhere in the SDK UI.

Insights are short personalized reads generated from the user's own health data, refreshed as new data syncs. The Home screen's Overview section shows an Insights entry carrying the unread count; it opens the insights feed, and leaving the feed returns to Home. Disable the module to remove the feed and every path to it from the SDK UI.

Additional modules will become disable-able in future releases. If there is a module you need to hide that isn't listed yet, contact Rolla during onboarding and we will prioritize adding it to `RollaDisabledModule`.

## Goals on Home

Off by default. With `showGoalsSection = true`, the bottom of the Home screen shows a Goals section — the user's selected goals, with an Edit action that opens the goals editor, or a select-goals call to action when none are selected yet:

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    showGoalsSection = true
)
```

The section is the bottom-most element of the Home scroll. It is particularly useful together with `showOptionsButton = false`, where it becomes the user's way to view and edit goals directly from Home. Users who reach the SDK with goals never selected are asked to choose them once, right after their first data-source connect — see [Goal selection after the first data-source connect](../sdk-auth-api/03-profile.md#goal-selection-after-the-first-data-source-connect).

## Data Source Configuration

By default the SDK offers every data source the user can connect (Rolla Band, Garmin, Oura, Apple Health, Health Connect). To hide specific sources, pass their `RollaDataSource` values in the `disabledDataSources` set (or omit the parameter / pass `emptySet()` to offer everything):

```kotlin
import com.rolla.sdk.wrapper.config.RollaDataSource

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
- **Band-only skips the picker.** If the Rolla Band is the only source left enabled, there is no data-source selection screen — onboarding takes the user straight to the band pairing screen (the gate becomes band pairing instead of source choosing), and the Data Sources entry is hidden from the Options sheet. Band status and unpairing remain available from the band button on the Home screen.

### RollaDataSource

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
