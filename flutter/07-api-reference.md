# API Reference

The public Dart surface of `rolla_sdk` 0.1.15. Everything below is exported from the package root:

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

**On this page:** [`RollaSDK`](#rollasdk) · [`RollaSdkHome`](#rollasdkhome) · [Host-driven navigation](#host-driven-navigation) · [Notification taps](#notification-taps) · [Headless calls](#headless-calls) · [Types & enums](#types--enums) · [Other exports](#other-exports) · [Not available in Flutter](#not-available-in-flutter)

## `RollaSDK`

Static entry point. Initialize once, then render `RollaSdkHome`.

### `RollaSDK.initializeWithToken(...)`

Initializes the SDK with the tokens obtained from the Rolla auth API. Must complete before you render `RollaSdkHome`. Calling it again disposes the previous instance and re-initializes; a different `userId` first clears the previous user's local caches. Throws `ArgumentError` when neither `userId` nor the token's `sub` claim identifies a user.

```dart
static Future<void> initializeWithToken({
  required String accessToken,
  required String userId,
  required String partnerId,
  RollaEnvironment environment = RollaEnvironment.production,
  String? baseUrl,
  String? refreshToken,
  Future<TokenRefreshResult?> Function()? onTokenExpired,
  VoidCallback? onLogout,
  VoidCallback? onSessionExpired,
  VoidCallback? onRequestDismiss,
  Duration? tokenExpiresIn,
  bool hideBottomNavigation = false,
  bool showBackButton = false,
  bool showOptionsButton = true,
  bool showGoalsSection = false,
  bool removeRollaBandReferences = true,
  bool showAccountSettings = false,
  bool? isProfileComplete,
  RollaLanguage? language,
  Set<RollaDisabledModule> disabledModules = const {},
  Set<RollaDataSource> disabledDataSources = const {},
  Branding? branding,
})
```

Every parameter is described in [Configuration → `initializeWithToken` parameters](05-configuration.md#initializewithtoken-parameters); the callbacks in [Code Integration](04-code-integration.md) and [Token Management](06-token-management.md).

### Session & Tokens

| Member | Description |
|--------|-------------|
| `static bool get isInitialized` | Whether an SDK instance exists. It turns `true` partway through `initializeWithToken` and is not reactive, so gate rendering on your own completion flag or the awaited future, not on this |
| `static Future<bool> updateToken({required String accessToken, String? refreshToken, Duration? expiresIn})` | Push fresh credentials — a proactive push, or the pair you refreshed elsewhere. Returns `false` when the SDK kept a newer pair it already held, or is not initialized. See [Token Management → Pushing a new token](06-token-management.md#pushing-a-new-token) |
| `static Future<void> logout()` | Clear stored tokens and dispose the SDK instance. Call when your user logs out of your app |

### Navigation & Headless

| Member | Description |
|--------|-------------|
| `static Future<RollaOpenScreenStatus> openScreen(RollaScreen screen)` | Open the SDK UI directly on a specific screen — see [Host-driven navigation](#host-driven-navigation) |
| `static Future<RollaSyncResult> syncHealthData({bool includeSamples = false})` | Headless sync of the user's primary data source — see [`syncHealthData`](#synchealthdata) |
| `static Future<BandBatteryResult> getBandBatteryLevel()` | Headless live battery read from the paired Rolla Band — see [`getBandBatteryLevel`](#getbandbatterylevel) |
| `static Future<PairedBandResult> getPairedBandInfo()` | Headless paired-band query, zero Bluetooth — see [`getPairedBandInfo`](#getpairedbandinfo) |
| `static Future<void> handleInsightGenerationPush({required String trigger, required String status})` | Forward a Rolla data-only FCM push (`{ trigger, status }`) so the insights feed refreshes when `status == 'complete'`. Safe no-op otherwise; only relevant if your app owns Firebase Messaging and forwards Rolla pushes |

> **Advanced:** `RollaSDK.initialize(config: RollaSDKConfig(...))` lets you supply a custom `RollaAuthProvider` instead of a token. Most integrations should use `initializeWithToken`; the configuration options above are only supported via `initializeWithToken`. The other public statics on `RollaSDK` (`instance`, `reset`, the `set…` methods, `runDeliberateLogout`, the configuration getters) are plumbing for the native wrappers and the SDK's own UI — do not call them.

## `RollaSdkHome`

The single widget that renders the entire SDK experience. It builds its **own** `MaterialApp.router`, so do not wrap it in another `MaterialApp` — render it directly from a route, as your authenticated root, or behind a `FutureBuilder` (see [Code Integration → Placing `RollaSdkHome`](04-code-integration.md#placing-rollasdkhome)).

```dart
const RollaSdkHome({
  Key? key,
  required String userId,      // must equal the userId passed to initializeWithToken
  bool isNativeModal = false,  // internal; leave false
})
```

```dart
// After initializeWithToken() completes:
return RollaSdkHome(userId: user.id);
```

`userId` must equal the value passed to `initializeWithToken`; a different value switches the SDK to another user's local storage. On mount it evaluates the mandatory startup gates (profile onboarding, consent, permissions, data-source connection) and shows whichever applies before Home. It also detects an activity interrupted by a crash and offers to resume, save or discard it.

## Host-Driven Navigation

### `openScreen`

```dart
static Future<RollaOpenScreenStatus> openScreen(RollaScreen screen)
```

Navigates the SDK UI to a specific screen — for example from your own menu entries. The opened screen becomes the **root of the SDK UI**, so back returns the user straight to your app through `onRequestDismiss`, never to an SDK Home they did not visit. Each subsequent call replaces the root with the new screen; `RollaScreen.home` restores the regular Home entry point.

The router lives inside `RollaSdkHome`, so the call needs the widget on screen. If it is not mounted yet — typically because you call `openScreen` and push your SDK route in the same tap — the request waits up to 15 seconds for `RollaSdkHome` to mount and settle its startup gates, then resolves:

```dart
final status = await RollaSDK.openScreen(RollaScreen.insights);
if (status != RollaOpenScreenStatus.opened) {
  debugPrint('Not opened: ${status.name}');
}
```

Pass `onRequestDismiss` whenever you use `openScreen`, even with `showBackButton` off: it is how the opened screen's back affordance exits to your app.

### `RollaScreen`

The screens your app can open directly — a deliberate whitelist:

| Value | Opens |
|-------|-------|
| `activityHistory` | The activity history list — every recorded activity with month/day filtering |
| `goals` | The goals editor |
| `home` | The SDK Home screen — restores Home as the root, with its usual bottom navigation and back-to-host button |
| `insights` | The insights feed — requires the insights module to be enabled (see [`RollaDisabledModule`](05-configuration.md#rolladisabledmodule)) |
| `resume` | No navigation at all: the SDK exactly as the user left it. Never `blockedByGate` or `screenDisabled`; resolves `opened` once `RollaSdkHome` is attached |

### `RollaOpenScreenStatus`

Every outcome is a typed status — the call never throws:

| Status | Meaning |
|--------|---------|
| `opened` | The SDK UI is on the requested screen |
| `notInitialized` | `initializeWithToken` has not completed. Nothing was opened |
| `screenDisabled` | The screen's module is in `disabledModules` (e.g. `insights` with the insights module disabled) — nothing was opened |
| `blockedByGate` | A mandatory startup step (onboarding, consent, permissions, data-source connection) is in front of the user. The SDK kept that step in place; the request is not retried |
| `uiUnavailable` | `RollaSdkHome` did not mount within the wait — nothing was opened. Push your SDK route right after calling |
| `superseded` | A newer `openScreen` request replaced this one while waiting for the UI — only the latest request is honored |

## Notification Taps

The SDK posts its own notifications (a workout in progress on Android, the inactivity reminder, the band battery warning, a background-location warning) and handles a tap on any of them itself — there is no API to call. On iOS this needs your `AppDelegate` to be the notification-center delegate, one line described in [Permissions & Entitlements → Notification Delegate](03-permissions.md#notification-delegate):

| Notification | When the SDK posts it | Tap leads to |
|--------------|-----------------------|--------------|
| **Background tracking disabled** | The SDK UI leaves the foreground mid-workout and *Always* location is missing | The OS app-settings page, opened by the SDK |
| **Stay on track** (inactivity reminder) | Two calendar days after the SDK was last opened, at 10:00 — every open of the SDK re-arms it | The insights feed, or Home when the insights module was disabled at the time the reminder was scheduled |
| **Battery low** (band battery warning) | At most once a day, when a battery reading before 18:00 shows the band at 20% or heading there before midnight | Home |
| **Workout in progress** / **Location Tracking** (Android only — the foreground-service notifications) | For the whole of a Bluetooth or GPS workout | The live workout, exactly as the user left it |

A screen target is routed through `openScreen` under the hood, so it is subject to the same rule: a tap that cold-launched the app is kept until `initializeWithToken` has run, and from then the SDK waits up to 15 seconds for `RollaSdkHome` to be on screen. With `RollaSdkHome` as your authenticated root ([Option B](04-code-integration.md#option-b--make-it-your-apps-root)) every tap lands; with the SDK on a pushed route, a tap opens your app and the navigation is dropped unless your app presents the SDK on launch.

> **If your app also uses `flutter_local_notifications`**, both share the plugin's single instance and its single tap callback: whichever calls `initialize` last receives all taps, the SDK's included. Running your own local notifications alongside the SDK is not supported in 0.1.15 — tell Rolla if you need it.

## Headless Calls

Three methods run **headlessly** — no `RollaSdkHome` needs to be on screen, only `initializeWithToken` must have completed (before that, `getBandBatteryLevel` reports `noBandPaired`, `getPairedBandInfo` reports `unknown` and `syncHealthData` is `skipped` with `notInitialized` — check your own init state first). Because there is no SDK UI to prompt from, **your app owns OS permissions**: when one is missing, the methods return a typed reason instead of prompting; the one prompt the SDK does raise is the notification permission, at `initializeWithToken`. They never throw; a transport failure is reported in the result.

### `syncHealthData`

```dart
static Future<RollaSyncResult> syncHealthData({bool includeSamples = false})
```

Runs a full sync of the user's primary data source (band over BLE, Apple Health, or Health Connect) and returns a typed [`RollaSyncResult`](#rollasyncresult):

| Field | Meaning |
|-------|---------|
| `outcome` | `success`, `skipped` (expectedly did nothing — see `skipReason`), or `failure` (see `error`); `partial` is reserved and not produced in 0.1.15 |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `band`, `appleHealth`, `healthConnect`, `garmin`, `oura`, or `unknown` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device. `startedAt` is `null` for `skipped`; `lastSyncAt` is set on success |
| `skipReason` | `noBandPaired`, `bandNotConnected` (a band is paired but couldn't be reached right now), `alreadyInProgress`, `serverSideSource` (Garmin/Oura sync server-side), `bluetoothPermissionRequired`, `bluetoothUnavailable`, `appleHealthPermissionRequired`, `healthConnectPermissionRequired`, `notInitialized`, `offline` |
| `error` | A `Failure` with a `message` (failure and partial only) |
| `syncedData` | Per-stream summary of what was uploaded; pass `includeSamples: true` to also receive the raw sample lists |

```dart
final result = await RollaSDK.syncHealthData();
switch (result.outcome) {
  case RollaSyncOutcome.success:
  case RollaSyncOutcome.partial:
    debugPrint('Synced ${result.source.name}: ${result.syncedData?.syncedDates}');
  case RollaSyncOutcome.skipped:
    debugPrint('Skipped because ${result.skipReason?.name}');
  case RollaSyncOutcome.failure:
    debugPrint('Failed: ${result.error?.message}');
}
```

### `getBandBatteryLevel`

```dart
static Future<BandBatteryResult> getBandBatteryLevel()
```

A **live BLE read** from the paired Rolla Band — the band must be reachable. Returns `status` and `level`: a percentage in `level` when `status` is `BandBatteryStatus.available`, otherwise a documented reason (`noBandPaired`, `bandNotConnected`, `bluetoothUnavailable`, `bluetoothPermissionRequired`, `notRollaDevice` — reserved, not currently returned — or `unknownError`). Never a stale value reported as live.

### `getPairedBandInfo`

```dart
static Future<PairedBandResult> getPairedBandInfo()
```

Answers "does this account currently have a Rolla Band?" with **zero Bluetooth** — no scan, no connect, no BLE permission; works with Bluetooth off. Returns `status` and `band`:

| Status | Meaning |
|--------|---------|
| `PairedBandStatus.bandPaired` | A band is paired — `band` carries its MAC address plus the last cached battery, firmware and serial, each `null` if the SDK hasn't read the band recently |
| `PairedBandStatus.noBandPaired` | The user's profile confirms no band is paired |
| `PairedBandStatus.unknown` | Could not be determined (offline with no local record) — reported instead of guessing |

The lookup is network-first: the profile is the authoritative pairing record, so a band unpaired remotely from another device is reported correctly. This is a pairing-state query, not a link-state one.

```dart
final paired = await RollaSDK.getPairedBandInfo();
if (paired.isPaired) {
  debugPrint('Band: ${paired.band!.macAddress}');
}
```

## Types & Enums

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

Returned from `onTokenExpired`; return `null` instead to signal that you have nothing fresher.

```dart
class TokenRefreshResult {
  final String accessToken;
  final String? refreshToken;
  final Duration? expiresIn;

  const TokenRefreshResult({
    required this.accessToken,
    this.refreshToken,
    this.expiresIn,
  });
}
```

> `expiresIn` is a `Duration`, unlike the native SDKs' seconds. If your backend returns seconds, wrap it: `Duration(seconds: expiresIn)`.

### `RollaLanguage`

`english`, `german`, `spanish`, `croatian`, `bosnian`, `serbianLatin`, `serbianCyrillic`, `arabic`. See [Configuration → Language](05-configuration.md#language).

### `RollaDisabledModule`

`weight`, `bloodPressure`, `leaderboards`, `insights`. See [Configuration → Module configuration](05-configuration.md#module-configuration).

### `RollaDataSource`

`band`, `garmin`, `oura`, `appleHealth`, `healthConnect`. See [Configuration → Data source configuration](05-configuration.md#data-source-configuration).

### `Branding`

Host branding passed to `initializeWithToken`. Six fields are required by the constructor; the SDK UI reads `appName`, `primaryColor`, `defaultThemeMode`, `defaultLocale`, `headerLogoAsset` and `privacyUrl`. A passed `Branding` replaces the SDK's defaults as a whole — see [Configuration → Custom branding](05-configuration.md#custom-branding-optional).

```dart
class Branding {
  // Required
  final String appName;
  final ThemeMode defaultThemeMode;
  final Color primaryColor;
  final Color secondaryColor;   // accepted, not used by the SDK UI
  final Color accentColor;      // accepted, not used by the SDK UI
  final Brightness brightness;  // accepted, not used by the SDK UI

  // Optional
  final String? headerLogoAsset;
  final String? privacyUrl;
  final Locale? defaultLocale;
  final String? partnerId;            // not needed — pass partnerId to initializeWithToken
  final String? termsUrl;             // not rendered in this version
  final String? onboardingImageAsset; // not rendered in this version
  final String? signUpImageAsset;     // not rendered in this version
  final String? authBackgroundAsset, authBackgroundVideoAsset, authBackgroundPosterAsset; // not rendered
  final String? authTitle, authSubtitle;                                                   // not rendered
  final Map<String, String>? authTitleI18n, authSubtitleI18n;                              // not rendered
}
```

### `RollaScreen` / `RollaOpenScreenStatus`

See [Host-driven navigation](#host-driven-navigation).

### `RollaSyncResult`

```dart
enum RollaSyncOutcome { success, partial, skipped, failure }
enum RollaSyncSource { band, appleHealth, healthConnect, garmin, oura, unknown }
enum RollaSyncSkipReason {
  noBandPaired, bandNotConnected, alreadyInProgress, serverSideSource,
  bluetoothPermissionRequired, bluetoothUnavailable,
  appleHealthPermissionRequired, healthConnectPermissionRequired,
  notInitialized, offline,
}

class RollaSyncResult {
  final RollaSyncOutcome outcome;
  final bool hasNewData;
  final RollaSyncSource source;
  final DateTime? startedAt;
  final DateTime? lastSyncAt;
  final RollaSyncSkipReason? skipReason;
  final Failure? error;                 // error?.message
  final RollaSyncedHealthData? syncedData;

  bool get didRun;                      // success or partial
}

class RollaSyncedHealthData {
  final RollaSyncSource source;
  final List<String> syncedDates;       // 'YYYY-MM-DD'
  final int? batteryLevel;
  final RollaSyncedStreamSummary? heartRate;   // count, from, to (epoch milliseconds)
  final RollaSyncedStreamSummary? hrv;
  final RollaSyncedStepsSummary? steps;        // + total
  final RollaSyncedSleepSummary? sleep;        // + blocks, minutes
  final RollaSyncedStreamSummary? weight;
  final RollaSyncedStreamSummary? bloodPressure;
  final RollaSyncedStreamSummary? workouts;
  final RollaSyncedSamples? samples;    // only with includeSamples: true
}
```

### `BandBatteryResult` / `PairedBandResult` / `RollaBandInfo`

```dart
enum BandBatteryStatus {
  available, noBandPaired, bandNotConnected, notRollaDevice,
  bluetoothUnavailable, bluetoothPermissionRequired, unknownError,
}

class BandBatteryResult {
  final BandBatteryStatus status;
  final int? level;            // 0–100, only when status == available
  bool get isAvailable;
}

enum PairedBandStatus { bandPaired, noBandPaired, unknown }

class PairedBandResult {
  final PairedBandStatus status;
  final RollaBandInfo? band;   // only when status == bandPaired
  bool get isPaired;
}

class RollaBandInfo {
  final String macAddress;
  final String? name;
  final int? rssi;
  final String? deviceType;
  final int? batteryPercent;
  final String? firmwareVersion;
  final String? serialNumber;
}
```

## Other Exports

| Export | Purpose |
| --- | --- |
| `JwtDecoder` | `extractUserId(token)` returns the JWT `sub` claim (the Rolla user ID), `extractExpiry(token)` the `exp` instant, `decode(token)` the payload map. Use it for the `userId` parameter instead of decoding the token yourself. |
| `showRollaSnackbar(...)` / `RollaSnackbarVariant` | SDK-styled snackbar (`success`, `error`, `info`), usable in host screens after init. |
| `RollaRoutes` | Route-path constants for the SDK's internal router; only needed for debugging. |
| `RollaModuleType`, `WeightModule`, `BloodPressureModule`, `ProfileModule`, `GoalsModule` | Module classes for advanced reads after init, via `RollaSDK.instance.getModule<T>(type)`. |
| `RollaSDKConfig`, `RollaAuthProvider`, `TokenAuthProvider`, `RollaAuthTokens` | Advanced `RollaSDK.initialize` configuration. |
| `Failure` and its subclasses | The error type carried by `RollaSyncResult.error` and the module read results. |

## Not Available in Flutter

The native wrappers expose a few things the Dart package does not, because the SDK runs inside your app's own Flutter engine:

- **Host event callbacks** (`rollaDidCompleteActivity`, `onBandPaired`, `onUiSyncCompleted`, …): there is no event-listener API. In Flutter you observe the SDK lifecycle through the callbacks you pass to `initializeWithToken` (`onLogout`, `onSessionExpired`, `onRequestDismiss`, `onTokenExpired`) and the typed results of the headless calls.
- **`notificationTarget`**: the SDK routes its own notification taps itself — see [Notification taps](#notification-taps).
- **`warmUpEngine` / `destroyEngine` / `clearSession`**: there is no separate engine. `initializeWithToken` is the warm-up, `RollaSDK.logout()` the session clear.
- **`RollaTransition`**: `RollaSdkHome` is a widget in your tree; the route you push it on owns the animation.

If your integration needs one of these, tell Rolla during onboarding.

---

**Previous:** [Token Management](06-token-management.md) | **Next:** [Troubleshooting](08-troubleshooting.md) | **Home:** [README](README.md)
