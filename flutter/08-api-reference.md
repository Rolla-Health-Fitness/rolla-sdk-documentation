# API Reference

The public Dart surface of `rolla_sdk`. Everything below is exported from the package root:

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

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
| `accessToken` | `String` | *required* | JWT access token from the [Auth API](../sdk-auth-api/02-authentication.md). |
| `userId` | `String` | *required* | Your user's id or user email (pass the same value to `RollaSdkHome`). The Rolla user ID is also available as the `sub` claim of the login JWT. |
| `partnerId` | `String` | *required* | Your partner id, provided by Rolla. |
| `environment` | `RollaEnvironment` | `production` | Backend environment. Must match where the token was issued. |
| `baseUrl` | `String?` | `null` | Override the environment's base URL. Leave `null` to use `environment.baseUrl`. |
| `refreshToken` | `String?` | `null` | Refresh token, if your backend issues one. |
| `onTokenExpired` | `Future<TokenRefreshResult?> Function()?` | `null` | Invoked when the SDK cannot refresh internally. Return a `TokenRefreshResult`, or `null` if refresh failed. See [Token Management](06-token-management.md). |
| `onLogout` | `VoidCallback?` | `null` | Invoked when the user logs out from inside the SDK. |
| `onRequestDismiss` | `VoidCallback?` | `null` | Invoked when the SDK's back button asks the host to dismiss it. **Pure-Flutter hosts must supply this** — typically `() => Navigator.of(context).pop()`. Without it the back button does nothing. |
| `tokenExpiresIn` | `Duration?` | `null` | Access-token lifetime, for proactive refresh. |
| `hideBottomNavigation` | `bool` | `false` | Hide the Home / Profile tabs, leaving only the activity (＋) button. |
| `showBackButton` | `bool` | `false` | Render a back button in the SDK's top app bar that calls `onRequestDismiss`. |
| `showSettingsButton` | `bool` | `true` | Show a Settings shortcut on Home (Data Sources, Goals). |
| `showAccountSettings` | `bool` | `false` | Show credential-management screens (change/reset password, change email, delete account). |
| `isProfileComplete` | `bool?` | `null` | `true` skips the SDK's profile-completeness check, `false` forces onboarding, `null` lets the SDK check via API. |
| `disabledModules` | `Set<RollaDisabledModule>` | `{}` | Modules whose UI is hidden everywhere. See [Branding & Modules](05-branding-and-modules.md). |
| `moduleConfigs` | `Map<RollaModuleType, RollaModuleConfig>?` | `null` | Reserved — has no effect in this release. Leave unset. |
| `branding` | `Branding?` | `null` | Host branding (colors, app name, theme, assets). See [`Branding`](#branding) below. |

### `RollaSDK.updateToken(...)`

Push fresh credentials to the SDK out-of-band (outside the `onTokenExpired` flow). No-op if `initializeWithToken` has never been called.

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

`true` once `initializeWithToken` has completed. Guard `RollaSdkHome` rendering on this.

### `RollaSDK.handleInsightGenerationPush(...)`

Forward a Rolla data-only FCM push (`{ trigger, status }`) so the insights feed refreshes when `status == 'complete'`. Safe no-op otherwise. Only relevant if you own Firebase Messaging and wire Rolla pushes.

```dart
static Future<void> handleInsightGenerationPush({
  required String trigger,
  required String status,
})
```

> **Advanced:** `RollaSDK.initialize(config: RollaSDKConfig(...))` lets you supply a custom `RollaAuthProvider` instead of a token. Most integrations should use `initializeWithToken`; `disabledModules` is only supported via `initializeWithToken`.

## `RollaSdkHome`

The single widget that renders the entire SDK experience. It builds its **own** `MaterialApp.router`, so do not wrap it in another `MaterialApp` — render it directly from a route.

```dart
const RollaSdkHome({
  super.key,
  required String userId,   // same value passed at init
  bool isNativeModal = false, // native add-to-app only; pure-Flutter hosts leave false
})
```

```dart
// After initializeWithToken() completes:
return RollaSdkHome(userId: user.id);
```

See [Code Integration](04-code-integration.md) for the full launch-screen pattern.

## Types & enums

### `RollaEnvironment`

```dart
enum RollaEnvironment {
  production('https://ross.rolla.cloud'),
  rnd('https://ross-rnd.rolla.cloud');

  final String baseUrl;
}
```

Must match the environment your token was issued for. `rnd` is the sandbox used during integration; `production` is the live backend. Note the default is `production`.

### `TokenRefreshResult`

Returned from `onTokenExpired`; return `null` instead to signal the refresh failed.

```dart
class TokenRefreshResult {
  final String accessToken;
  final String? refreshToken;
  final Duration? expiresIn;   // a Duration, not seconds

  const TokenRefreshResult({
    required this.accessToken,
    this.refreshToken,
    this.expiresIn,
  });
}
```

> `expiresIn` is a `Duration`, unlike the native SDKs' seconds. If your backend returns seconds, wrap it: `Duration(seconds: expiresIn)`.

### `RollaDisabledModule`

The modules you may disable. A disabled module's UI (overview cards, detail pages, settings entries) is hidden everywhere.

```dart
enum RollaDisabledModule {
  weight,
  bloodPressure,
}
```

### `Branding`

Host branding passed to `initializeWithToken`. The six required fields drive the theme; the optional fields fall back to the SDK's bundled defaults when `null`. Asset paths must resolve inside the SDK package's bundle — see [Branding & Modules](05-branding-and-modules.md).

```dart
class Branding {
  // Required
  final String appName;
  final ThemeMode defaultThemeMode;
  final Color primaryColor;
  final Color secondaryColor;
  final Color accentColor;
  final Brightness brightness;

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
  final Map<String, String>? authTitleI18n;    // per-locale, e.g. {'en': 'Welcome'}
  final Map<String, String>? authSubtitleI18n;
  final Locale? defaultLocale;
}
```

## Other exports

| Export | Purpose |
| --- | --- |
| `RollaModuleType` / `RollaModuleConfig` | Module enums and config types (the `moduleConfigs` parameter is reserved in this release). |
| `WeightModule`, `BloodPressureModule`, `ProfileModule`, `GoalsModule` | Module classes for advanced reads after init, via `RollaSDK.instance.getModule<T>(type)`. |
| `RollaRoutes` | Route-path constants for the SDK's internal router; only needed for deep-linking and debugging. |
| `RollaSDKConfig`, `RollaAuthProvider`, `TokenAuthProvider` | Advanced `RollaSDK.initialize` configuration. |
| `showRollaSnackbar(...)` / `RollaSnackbarVariant` | SDK-styled snackbar, usable in host screens after init. |

There is no event-listener API: in Flutter you observe the SDK lifecycle through the callbacks you pass to `initializeWithToken` (`onLogout`, `onRequestDismiss`, `onTokenExpired`) and through your own `Navigator`.

---

**Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
