# API Reference

The public Dart surface of `rolla_sdk`. Everything below is exported from the package root.

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

> **Note:** Unlike the React Native bridge (a flat `Rolla.*` namespace over a hidden Flutter engine), the Flutter package *is* the SDK. You call static methods on `RollaSDK` and render the `RollaSdkHome` widget directly in your tree.

## `RollaSDK`

Static entry point. Initialize once, then render `RollaSdkHome`.

### `RollaSDK.initializeWithToken(...)`

Initializes the SDK with credentials your backend minted. Must complete before you render `RollaSdkHome`. Calling it again disposes the previous instance and re-initializes.

```dart
static Future<void> initializeWithToken({
  required String accessToken,
  required String userId,
  required String partnerId,
  RollaEnvironment environment = RollaEnvironment.production,
  String? baseUrl,
  String? refreshToken,
  TokenRefreshCallback? onTokenExpired,
  VoidCallback? onLogout,
  VoidCallback? onRequestDismiss,
  Duration? tokenExpiresIn,
  bool hideBottomNavigation = false,
  bool showBackButton = false,
  bool showSettingsButton = true,
  bool showAccountSettings = false,
  bool? isProfileComplete,
  Set<RollaDisabledModule> disabledModules = const {},
  Map<RollaModuleType, RollaModuleConfig>? moduleConfigs,
  Branding? branding,
})
```

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `accessToken` | `String` | *required* | JWT access token from your auth service. |
| `userId` | `String` | *required* | User id from your user system (pass the same value to `RollaSdkHome`). |
| `partnerId` | `String` | *required* | Your partner id, provided by Rolla. |
| `environment` | `RollaEnvironment` | `production` | Backend environment. Must match where the token was issued. |
| `baseUrl` | `String?` | `null` | Override the environment's base URL. Leave `null` to use `environment.baseUrl`. |
| `refreshToken` | `String?` | `null` | Refresh token, if your backend issues one. |
| `onTokenExpired` | `TokenRefreshCallback?` | `null` | Invoked on a `401`. Return a `TokenRefreshResult`, or `null` to force logout. See [Token Management](06-token-management.md). |
| `onLogout` | `VoidCallback?` | `null` | Invoked when the user logs out from inside the SDK. Tear down your session / navigate to login. |
| `onRequestDismiss` | `VoidCallback?` | `null` | Invoked when the SDK's back button (see `showBackButton`) asks the host to dismiss it. **Pure-Flutter hosts must supply this** — typically `() => Navigator.of(context).pop()`. Without it the back button does nothing. (Native add-to-app hosts leave it `null` and use the `rolla_sdk/init` method channel instead.) |
| `tokenExpiresIn` | `Duration?` | `null` | Lifetime of the access token, for proactive refresh. |
| `hideBottomNavigation` | `bool` | `false` | Hide the HOME and PROFILE tabs, leaving only the activity (PLUS) button. |
| `showBackButton` | `bool` | `false` | Render a back button in the SDK's top app bar that calls `onRequestDismiss`. Pure-Flutter hosts that embed `RollaSdkHome` on their own `Navigator` set this `true`. |
| `showSettingsButton` | `bool` | `true` | Show a Settings button on Home that opens a sheet with Data Sources and Goals. Set `false` if you surface those elsewhere. |
| `showAccountSettings` | `bool` | `false` | Show credential-management screens (change/reset password, change email, delete account). Keep `false` unless your app delegates credential management to the SDK. |
| `isProfileComplete` | `bool?` | `null` | Hint about onboarding: `true` skips the SDK's completeness check, `false` forces onboarding, `null` lets the SDK check via API. |
| `disabledModules` | `Set<RollaDisabledModule>` | `{}` | Modules whose UI is hidden everywhere. See [Branding & Modules](05-branding-and-modules.md). |
| `moduleConfigs` | `Map<RollaModuleType, RollaModuleConfig>?` | `null` | Advanced per-module configuration. Rarely needed. |
| `branding` | `Branding?` | `null` | Host branding (colors, app name, theme, assets). See [`Branding`](#branding) below. |

```dart
await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,
  tokenExpiresIn: Duration(seconds: session.expiresIn),
  userId: user.id,
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.production,
  branding: myBranding,
  showBackButton: true,
  onRequestDismiss: () => Navigator.of(context).pop(),
  onTokenExpired: () async => TokenRefreshResult(
    accessToken: (await refreshMyToken()).accessToken,
  ),
);
```

### `RollaSDK.updateToken(...)`

Push fresh credentials to the SDK out-of-band (e.g. after your own refresh, outside the `onTokenExpired` flow).

```dart
static Future<void> updateToken({
  required String accessToken,
  String? refreshToken,
  Duration? expiresIn,
})
```

### `RollaSDK.logout()`

Clear stored tokens and dispose the SDK instance. Call when your user logs out.

```dart
static Future<void> logout()
```

### `RollaSDK.isInitialized`

```dart
static bool get isInitialized
```

`true` once `initializeWithToken` (or `initialize`) has completed. Guard `RollaSdkHome` rendering on this.

### `RollaSDK.handleInsightGenerationPush(...)`

Forward a Rolla data-only FCM push (`{ trigger, status }`) so the insights feed refreshes when `status == 'complete'`. Safe no-op otherwise. Only relevant if you own Firebase Messaging and wire Rolla pushes.

```dart
static Future<void> handleInsightGenerationPush({
  required String trigger,
  required String status,
})
```

> **Advanced:** `RollaSDK.initialize({ required RollaSDKConfig config })` lets you supply a custom `RollaAuthProvider` instead of a token. Most partners should use `initializeWithToken`; `disabledModules` is only supported via `initializeWithToken`.

## `RollaSdkHome`

The single widget that renders the entire SDK experience. It builds its **own** `MaterialApp.router` (GoRouter, theming, BLoC providers, lifecycle), so **do not wrap it in another `MaterialApp`** — render it directly.

```dart
class RollaSdkHome extends StatefulWidget {
  /// User id from the host app. Pass the same value used at init.
  final String userId;

  /// True only for native add-to-app modal presentation. Pure-Flutter
  /// hosts leave this false (the default).
  final bool isNativeModal;

  const RollaSdkHome({ super.key, required this.userId, this.isNativeModal = false });
}
```

```dart
// After initializeWithToken() completes:
return RollaSdkHome(userId: user.id);
```

See [Code Integration](04-code-integration.md) for the full launch screen pattern.

## Types & enums

### `RollaEnvironment`

```dart
enum RollaEnvironment {
  production('https://ross.rolla.cloud'),
  rnd('https://ross-rnd.rolla.cloud');

  final String baseUrl;
}
```

Must match the environment your token was issued for. `rnd` is research-and-development (used by the demos and during onboarding); `production` is the live backend. `initializeWithToken` defaults to `production`. Read the base URL via `environment.baseUrl`.

### `TokenRefreshResult`

Returned from `onTokenExpired`. Return `null` instead to force a logout.

```dart
class TokenRefreshResult {
  final String accessToken;
  final String? refreshToken;
  final Duration? expiresIn;   // note: a Duration, not seconds

  const TokenRefreshResult({
    required this.accessToken,
    this.refreshToken,
    this.expiresIn,
  });
}

typedef TokenRefreshCallback = Future<TokenRefreshResult?> Function();
```

> **Heads up:** `expiresIn` is a `Duration`, unlike the RN/native `expiresIn` which is seconds. If your backend returns seconds, wrap it: `Duration(seconds: expiresIn)`.

### `RollaDisabledModule`

Modules you may disable today. A disabled module's overview cards, detail pages, settings entries, and cross-module references are hidden; its DI still runs so cross-module dependencies stay safe.

```dart
enum RollaDisabledModule {
  weight,
  bloodPressure,
}
```

These are the **only** disablable modules in this version. The broader internal `RollaModuleType` enum lists every shipped module but is not a disable surface.

### `Branding`

Host branding passed to `initializeWithToken`. The five required fields drive theme; the rest are optional and fall back to the SDK's bundled defaults when `null`. **Asset paths must resolve inside the SDK package's bundle** — see [Branding & Modules](05-branding-and-modules.md).

```dart
class Branding {
  // Required
  final String appName;
  final ThemeMode defaultThemeMode;   // dart:ui ThemeMode
  final Color primaryColor;           // dart:ui Color, e.g. Color(0xFF21C55E)
  final Color secondaryColor;
  final Color accentColor;
  final Brightness brightness;        // Brightness.light | .dark

  // Optional
  final String? partnerId;            // Partner-ID header value
  final String? termsUrl;
  final String? privacyUrl;
  final String? headerLogoAsset;
  final String? onboardingImageAsset;
  final String? signUpImageAsset;
  final String? authBackgroundAsset;
  final String? authBackgroundVideoAsset;
  final String? authBackgroundPosterAsset;
  final String? authTitle;
  final String? authSubtitle;
  final Map<String, String>? authTitleI18n;      // per-locale, e.g. {'en': 'Welcome'}
  final Map<String, String>? authSubtitleI18n;
  final Locale? defaultLocale;

  const Branding({ /* fields above */ });
}
```

```dart
const myBranding = Branding(
  appName: 'My App',
  defaultThemeMode: ThemeMode.system,
  primaryColor: Color(0xFF21C55E),
  secondaryColor: Color(0xFF0F0F0F),
  accentColor: Color(0xFF21C55E),
  brightness: Brightness.dark,
  partnerId: 'your-partner-id',
  termsUrl: 'https://example.com/terms',
  privacyUrl: 'https://example.com/privacy',
);
```

### `RollaRoutes`

Route-path constants for the SDK's internal GoRouter. You rarely need these — `RollaSdkHome` drives its own navigation — but they're exported for deep-linking and debugging.

```dart
// Core
RollaRoutes.home               // '/'
RollaRoutes.onboarding         // '/onboarding'
RollaRoutes.consentGate        // '/consent-gate'
RollaRoutes.dataSourceGate     // '/datasource-gate'
RollaRoutes.bandPairing        // '/band-pairing'
RollaRoutes.bandStatus         // '/band-status'
RollaRoutes.fabActionsAll      // '/fab-actions-all'

// Activity
RollaRoutes.activitySummary    // '/activity/summary'
RollaRoutes.activityReview     // '/activity/review'
RollaRoutes.activityHistory    // '/activity/history'
RollaRoutes.activityStart(id)  // '/activity/<catalogId>'  (function)

// Manual activity
RollaRoutes.manualActivity        // '/manual-activity'
RollaRoutes.manualActivitySelect  // '/manual-activity/select'

// Profile
RollaRoutes.profile            // '/profile'
RollaRoutes.personalDetails    // '/profile/personal-details'

// Settings
RollaRoutes.settings                // '/settings'
RollaRoutes.settingsSecurity        // '/settings/security'
RollaRoutes.settingsChangeEmail     // '/settings/change-email'
RollaRoutes.settingsChangePassword  // '/settings/change-password'
RollaRoutes.settingsResetPassword   // '/settings/reset-password'
RollaRoutes.settingsDeleteAccount   // '/settings/delete-account'
RollaRoutes.settingsTheme           // '/settings/theme'
RollaRoutes.settingsLanguage        // '/settings/language'
RollaRoutes.settingsPermissions     // '/settings/permissions'
RollaRoutes.settingsIntegrations    // '/settings/integrations'
RollaRoutes.settingsWithdrawConsent // '/settings/withdraw-consent'
RollaRoutes.settingsDownloadMyData  // '/settings/download-my-data'
RollaRoutes.settingsPrivacy         // '/settings/privacy'

// Other
RollaRoutes.goals      // '/goals'
RollaRoutes.faq        // '/faq'
RollaRoutes.contactUs  // '/contact-us'
```

## Modules

The package exports four module classes plus their type/config enums. The SDK initializes and renders all modules itself — these are exported for advanced reads (e.g. fetching the latest weight) after init.

| Export | Purpose |
| --- | --- |
| `WeightModule` | Weight tracking (BMI, trends, targets). |
| `BloodPressureModule` | Blood-pressure history and manual logging. |
| `ProfileModule` | User profile management. |
| `GoalsModule` | Daily health goals. |
| `RollaModuleType` | Enum naming every module the SDK ships. |
| `RollaDisabledModule` | The subset you can disable (see above). |
| `RollaModuleConfig` | Base type for `moduleConfigs` entries. |

Access a module after init via the instance:

```dart
final weight = RollaSDK.instance.getModule<WeightModule>(RollaModuleType.weight);
```

## Auth, config & UI helpers (also exported)

| Export | Purpose |
| --- | --- |
| `RollaSDKConfig` | Config object for the advanced `RollaSDK.initialize`. |
| `RollaAuthProvider` / `TokenAuthProvider` | Implement a custom auth provider for `initialize`. |
| `RollaSnackbar` | SDK-styled snackbar, usable in host screens after init. |
| `Goal`, `Profile`, and failure types | Domain entities returned by module reads. |

> **Note:** There is no `RollaCloseEvent` / event-listener API like the RN bridge. In Flutter you observe lifecycle through the callbacks you pass to `initializeWithToken` (`onLogout`, `onRequestDismiss`, `onTokenExpired`) and through your own `Navigator`. For the canonical native error/enum lists (and the values the native wrappers serialize `RollaDisabledModule` to), see the [iOS API Reference](../ios/10-api-reference.md) and [Android API Reference](../android/08-api-reference.md).

---

**Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
