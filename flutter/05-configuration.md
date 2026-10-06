# Configuration

Everything you can shape about the SDK — branding, language, modules, data sources, UI chrome — is a parameter of `RollaSDK.initializeWithToken(...)`. This page is the complete configuration reference: the full parameter table first, then a closer look at each configurable surface.

> **Configuration is read at initialization.** Apart from tokens, which you can push live with `RollaSDK.updateToken()` (see [Token Management](06-token-management.md)), the options apply for the lifetime of the SDK instance. To change branding, language, modules or data sources, call `initializeWithToken` again with the new values — it disposes and rebuilds the SDK — and re-render `RollaSdkHome`.

## `initializeWithToken` parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `accessToken` | `String` | Yes | — | JWT access token from `POST /api/login` |
| `userId` | `String` | Yes | — | User identifier for local data namespacing (per-user storage isolation). Pass the JWT `sub` claim (`JwtDecoder.extractUserId`) or your own stable id. Never sent to the backend; an empty string falls back to the JWT `sub` |
| `partnerId` | `String` | Yes | — | Partner identifier provided by Rolla |
| `environment` | `RollaEnvironment` | No | `production` | Target backend: `rnd` (sandbox) or `production`. Must match where the token was issued — see [Code Integration](04-code-integration.md#environment-values) |
| `baseUrl` | `String?` | No | from `environment` | Override the backend URL; leave unset to use `environment.baseUrl` |
| `refreshToken` | `String?` | No | unset | Refresh token for the SDK's own credential renewal |
| `tokenExpiresIn` | `Duration?` | No | unset | Access-token lifetime, for proactive refresh. A `Duration`, not seconds |
| `onTokenExpired` | `Future<TokenRefreshResult?> Function()?` | No | unset | Invoked when the SDK cannot refresh internally. See [Token Management](06-token-management.md) |
| `onLogout` | `VoidCallback?` | No | unset | The user signed out from inside the SDK |
| `onSessionExpired` | `VoidCallback?` | No | unset | Every refresh path failed; the session is unrecoverable |
| `onRequestDismiss` | `VoidCallback?` | No | unset | The SDK asks your app to pop its route (back button, or back from a screen opened with `openScreen`). **Pure-Flutter hosts must supply this** to use the back button |
| `hideBottomNavigation` | `bool` | No | `false` | Hide the Home / Profile tabs, leaving only the activity (＋) button |
| `showBackButton` | `bool` | No | `false` | Back button in the SDK's top app bar that calls `onRequestDismiss` |
| `showOptionsButton` | `bool` | No | `true` | Three-dot options action at the trailing edge of the Home app bar. Tapping it opens an "Options" bottom sheet with shortcuts into the SDK's features that you haven't disabled. See [Options button](#options-button) |
| `showGoalsSection` | `bool` | No | `false` | Show the user's goals at the bottom of the Home screen, with an edit action. See [Goals on Home](#goals-on-home) |
| `showAccountSettings` | `bool` | No | `false` | Credential-management screens in Settings (change / reset password, change e-mail, delete account) — for hosts that let the SDK manage the Rolla account |
| `removeRollaBandReferences` | `bool` | No | `true` | Generic "fitness device" wording instead of Rolla Band naming. See [Rolla Band references](#rolla-band-references) |
| `isProfileComplete` | `bool?` | No | `null` | Whether the user's profile is already complete. See [Profile completeness](#profile-completeness) |
| `language` | `RollaLanguage?` | No | unset (profile-driven) | Forces the SDK UI language for the instance's lifetime. See [Language](#language) |
| `disabledModules` | `Set<RollaDisabledModule>` | No | `{}` (nothing disabled) | Modules whose entire UI is hidden across the SDK. See [Module configuration](#module-configuration) |
| `disabledDataSources` | `Set<RollaDataSource>` | No | `{}` (all offered) | Data sources whose connect option is hidden wherever the user picks a source to connect. See [Data source configuration](#data-source-configuration) |
| `branding` | `Branding?` | No | unset (SDK defaults) | Visual identity. See [Custom branding](#custom-branding-optional) |

The same options exist on the native wrappers' `RollaConfiguration`; the changelog names them by their native names. The differences in naming: the native `RollaBranding.hostAppName` is `Branding.appName`, `RollaBranding.themeMode` is `Branding.defaultThemeMode`, and `RollaBranding.removeRollaBandReferences` is the `removeRollaBandReferences` parameter here.

For the identity and auth essentials (`accessToken`, `partnerId`, `environment`) and a minimal setup example, see [Code Integration](04-code-integration.md).

## Custom branding (optional)

Pass a `Branding` instance via the `branding:` parameter. Colors are `dart:ui` `Color` values — use the `0xAARRGGBB` literal, not a hex string:

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart' show Branding;

const Branding partnerBranding = Branding(
  // These shape the SDK UI
  appName: 'Your App Name',                  // names your app in consent and permission copy
  primaryColor: Color(0xFF1976D2),           // seeds the SDK's entire color scheme, light and dark
  defaultThemeMode: ThemeMode.system,        // light | dark | system
  headerLogoAsset: null,                     // your logo, pre-bundled into the SDK by Rolla
  privacyUrl: 'https://example.com/privacy', // privacy link on the consent screen
  defaultLocale: null,                       // fallback locale when no language is configured or stored

  // Required by the constructor, not used by the SDK UI
  secondaryColor: Color(0xFF625B71),
  accentColor: Color(0xFF7D5260),
  brightness: Brightness.light,
);
```

Then hand it to the initializer (see [Code Integration](04-code-integration.md) for the full call):

```dart
await RollaSDK.initializeWithToken(
  // ... credentials, environment, and callbacks ...
  branding: partnerBranding,
);
```

- **`appName`** — your app's display name. SDK copy that refers to the app names it explicitly — the consent screen's legal intro and the battery-optimization / motion-permission prompts — in every SDK language. It is also the `MaterialApp.title` of the SDK shell.
- **`primaryColor`** — seeds the whole SDK color scheme (buttons, navigation, inputs, charts, share cards) in both light and dark themes; it is not just an accent. The loading indicator shown while the SDK starts uses it too.
- **`defaultThemeMode`** — the theme the SDK UI runs in: `ThemeMode.light`, `ThemeMode.dark`, or following the device setting (`ThemeMode.system`).
- **`headerLogoAsset`** — path of your logo inside the SDK bundle (see [Branding assets](#branding-assets) below). `null` keeps the SDK default.
- **`privacyUrl`** — your privacy policy, linked from the consent screen's "privacy policy" text.
- **`defaultLocale`** — a fallback UI locale, consulted only when `language` is unset and the user has neither a stored pick nor a profile language yet.

> **A `Branding` you pass replaces the SDK's built-in defaults as a whole**, not field by field. Unlike the native wrappers, where an unset `RollaBranding` field keeps the default, a `null` `privacyUrl` here means the consent screen has no privacy link. Set every field you care about. Passing no `branding` at all keeps the complete default look.

> **Color literals only.** `Color(0xFF1976D2)` is opaque blue — the leading `FF` is the alpha byte. Passing a bare `0x1976D2` yields a fully transparent color.

The class declares further optional fields (`termsUrl`, `onboardingImageAsset`, `signUpImageAsset`, the `auth*` text and background fields, `partnerId`) that this version of the SDK does not render. Leave them unset. The full declaration is in [API Reference → `Branding`](07-api-reference.md#branding).

## Branding assets

Image assets used by the SDK (such as the logo referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK package** at build time — they are loaded from the SDK's own asset bundle, not from your app's `pubspec.yaml` assets, and the header logo must be an **SVG** (the SDK renders it with its own SVG widget).

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `Branding`.

## Rolla Band references

The SDK can refer to the paired wearable either generically ("fitness device") or specifically as the "Rolla Band" throughout its UI. This is controlled by the `removeRollaBandReferences` parameter:

```dart
await RollaSDK.initializeWithToken(
  // ...
  removeRollaBandReferences: true, // Default — generic "fitness device" wording
);
```

- `true` (**default**) — the SDK uses generic "fitness device" wording. This is the right choice for most partner apps, which pair with their own-branded or third-party wearables rather than a Rolla-branded band.
- `false` — the SDK shows Rolla Band-specific references (naming, imagery, and copy that call out the Rolla Band by name).

## Language

By default the SDK renders in the language of the user's backend profile (see the [Profile guide](../sdk-auth-api/03-profile.md)). To force a specific UI language instead, set `language`:

```dart
await RollaSDK.initializeWithToken(
  // ...
  language: RollaLanguage.german,
);
```

- **Authoritative when set.** The configured language wins for the instance's lifetime — persisted in-SDK picks and the backend profile language cannot override it.
- **Applied at initialization.** Changing the language means calling `initializeWithToken` again, like any configuration change.
- **Kept in sync with the backend.** When the configured language differs from the user's profile, the SDK writes it to the profile at startup, so backend-generated content (goal labels, insights) arrives in the same language as the SDK UI.
- **Unset keeps the profile-driven behavior.** With `language` unset, the SDK follows the profile's language — which your app can set via [`POST /api/setprofile`](../sdk-auth-api/03-profile.md) if you manage language selection server-side. A host that lets users change the device locale, like Rolla's own white-label app, leaves it unset.

### `RollaLanguage`

| Value | Language |
|-------|----------|
| `english` | English |
| `german` | German (Deutsch) |
| `spanish` | Spanish (Español) |
| `croatian` | Croatian (Hrvatski) |
| `bosnian` | Bosnian (Bosanski) |
| `serbianLatin` | Serbian — Latin script (Srpski) |
| `serbianCyrillic` | Serbian — Cyrillic script (Српски) |
| `arabic` | Arabic (العربية), right-to-left |

## Module configuration

By default every module is enabled. To hide a module's entire UI everywhere it appears in the SDK, pass its `RollaDisabledModule` value in `disabledModules` (or omit the parameter to keep everything enabled):

```dart
await RollaSDK.initializeWithToken(
  // ...
  disabledModules: {RollaDisabledModule.weight, RollaDisabledModule.bloodPressure},
);
```

### `RollaDisabledModule`

`disabledModules` accepts the following values. These are the modules that can currently be turned off per integration:

| Value | Hides |
|-------|-------|
| `weight` | The Weight tracking module (weight logging, BMI, and targets) |
| `bloodPressure` | The Blood Pressure tracking and manual-logging module |
| `leaderboards` | The Leaderboards module (weekly/monthly competitive rankings) |
| `insights` | The Insights module (the insights feed, the Home screen's Insights entry, and its unread badge). Also makes `RollaSDK.openScreen(RollaScreen.insights)` resolve as `screenDisabled` |

Leaderboards let users compare their Health Score or Active Points against other users in your tenant over weekly and monthly periods, with join/leave controls per challenge type. Disable the module to hide competitive rankings everywhere in the SDK UI.

Insights are short personalized reads generated from the user's own health data, refreshed as new data syncs. The Home screen's Overview section shows an Insights entry carrying the unread count; it opens the insights feed, and leaving the feed returns to Home. Disable the module to remove the feed and every path to it from the SDK UI.

Additional modules will become disable-able in future releases. If there is a module you need to hide that isn't listed yet, contact Rolla during onboarding and we will prioritize adding it to `RollaDisabledModule`.

## Options button

On by default. The three-dot action at the trailing edge of the Home app bar opens an "Options" bottom sheet with shortcuts to Data Sources, Goals, Leaderboards and the FAQ — the entries for modules you disabled are left out. Pass `showOptionsButton: false` if your app reaches those screens through its own navigation, for example with `openScreen` (see [API Reference → Host-driven navigation](07-api-reference.md#host-driven-navigation)) or by letting the SDK's Profile tab do it.

## Goals on Home

Off by default. With `showGoalsSection: true`, the bottom of the Home screen shows a Goals section — the user's selected goals, with an Edit action that opens the goals editor, or a select-goals call to action when none are selected yet:

```dart
await RollaSDK.initializeWithToken(
  // ...
  showGoalsSection: true,
);
```

The section is the bottom-most element of the Home scroll. It is particularly useful together with `showOptionsButton: false`, where it becomes the user's way to view and edit goals directly from Home. Users who reach the SDK with goals never selected are asked to choose them once, right after their first data-source connect — see [Goal selection after the first data-source connect](../sdk-auth-api/03-profile.md#goal-selection-after-the-first-data-source-connect).

## Data source configuration

By default the SDK offers every data source the user can connect (Rolla Band, Garmin, Oura, Apple Health on iOS, Health Connect on Android). To hide specific sources, pass their `RollaDataSource` values in `disabledDataSources` (or omit the parameter to offer everything):

```dart
await RollaSDK.initializeWithToken(
  // ...
  disabledDataSources: {RollaDataSource.garmin, RollaDataSource.oura, RollaDataSource.appleHealth},
);
```

A hidden source's connect option is suppressed everywhere the user picks a source to connect — the Data Sources screen and the onboarding data-source step. This is useful when you want to route users toward a specific source: disabling everything except the band, for example, sends users straight to the "Pair your band" flow.

### Behavior notes

- **Deny-list semantics.** An empty set (the default) offers every source. Each value present hides that source. This matches `disabledModules`.
- **Already-connected sources stay visible.** If a user has already connected a source that you later disable, it still appears on the Data Sources screen so they can view or disconnect it — only offering a *new* connection is suppressed.
- **The band is a safety floor.** At least one source is always connectable. If you disable *every* source, the SDK keeps the Rolla Band available so onboarding never dead-ends.
- **Band-only skips the picker.** If the Rolla Band is the only source left enabled, there is no data-source selection screen — onboarding takes the user straight to the band pairing screen, and the Data Sources entry is hidden from the Options sheet. Band status and unpairing remain available from the band button on the Home screen.

### `RollaDataSource`

| Value | Hides |
|-------|-------|
| `band` | The Rolla Band pairing option |
| `garmin` | Garmin Connect |
| `oura` | Oura |
| `appleHealth` | Apple Health (iOS only) |
| `healthConnect` | Health Connect (Android only) |

## Profile completeness

The SDK's own onboarding (consent, account details, goals) runs in front of Home when the user's profile is incomplete. `isProfileComplete` tells the SDK what to assume:

| Value | Behavior |
|-------|----------|
| `null` (default) | The SDK fetches the backend profile at startup and skips its account-details onboarding when the profile already carries a username, birthdate, gender, height and weight — which your backend can set in advance via [`POST /api/setprofile`](../sdk-auth-api/03-profile.md). Weight stays mandatory when the weight module is disabled, because calorie calculations depend on it |
| `true` | Your app owns onboarding and guarantees a complete profile; the SDK skips the check without an API call |
| `false` | Always run the account-details onboarding — a deterministic hook for testing |

## Invalid values

Every option above is a Dart enum or typed value, so a misspelled module, language or data source is a compile error rather than a silent no-op. The only runtime check is on identity: `initializeWithToken` throws an `ArgumentError` when neither `userId` nor the token's `sub` claim resolves to a user — see [Code Integration](04-code-integration.md#initialize-with-a-token).

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
