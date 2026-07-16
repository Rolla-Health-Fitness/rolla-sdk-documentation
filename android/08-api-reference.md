# API Reference

Complete reference for the Rolla SDK classes, methods, interfaces, error types, and close reasons.

## Native API Reference (Android)

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `Rolla(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var listener: RollaListener?` | Set the listener for callbacks |
| `val isPresenting: Boolean` | Whether the SDK is currently visible |
| `show(activity: Activity)` | Present the SDK from an Activity |
| `show(fragment: Fragment)` | Present the SDK from a Fragment |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token, refreshToken?, expiresIn?, callback?)` | Push fresh credentials to the SDK. The `callback` is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. |
| `clearSession(callback?)` | Clear all persisted session data |
| `warmUpEngine(context, callback?)` | Start and configure the engine ahead of time, without any UI. See [Headless Methods](#headless-methods) |
| `getBandBatteryLevel(context, callback)` | Headless live battery read from the paired Rolla band. See [Headless Methods](#headless-methods) |
| `getPairedBandInfo(context, callback)` | Headless paired-band query — zero Bluetooth. See [Headless Methods](#headless-methods) |
| `syncHealthData(context, includeSamples = false, callback)` | Headless sync of the user's primary data source. See [Headless Methods](#headless-methods) |
| `companion fun destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaListener Interface

| Method | Description |
|--------|-------------|
| `onRollaClosed(rolla, reason)` | Called when the SDK UI is dismissed |
| `onRollaError(rolla, error)` | Called when an error occurs |
| `onTokenRefreshed(rolla, token, refreshToken?, expiresIn?)` | Called when the SDK refreshes tokens internally |
| `onTokenExpired(rolla)` | Called when the host app must provide new tokens |
| `onSyncHealthDataCompleted(rolla, result)` | Called when a headless `syncHealthData` reaches a terminal outcome, with the same `RollaSyncResult` the callback receives |
| `onActivityStarted(rolla, activity)` | A live tracking session started — `RollaStartedActivity.origin` distinguishes a fresh start from a crash-recovery resume. See [Host Events](#host-events) |
| `onActivityCompleted(rolla, activity)` | An activity reached a lifecycle phase: `FINISHED` (saved in-SDK), then `UPLOADED` or `UPLOAD_FAILED`. Key idempotency on `(activityId, phase)` |
| `onActivityRemoved(rolla, activity)` | An activity's record was removed without a kept result — `reason` is `CANCELED` (crash-recovery discard; the "activity canceled" case) or `DELETED` (user deleted it, backend-confirmed) |
| `onUiSyncCompleted(rolla, result)` | A sync completed inside the SDK UI (auto-sync on open, return from background, manual refresh) |
| `onBandPaired(rolla, band)` | The user paired a band inside the SDK UI |
| `onBandUnpaired(rolla, band)` | The user unpaired the band inside the SDK UI (backend-confirmed) |
| `onBandConnected(rolla, band)` | The paired band established a live BLE link. See [Host Events](#host-events) for the link-event caveats |
| `onBandDisconnected(rolla, band)` | The paired band lost its live BLE link (debounced a few seconds) |
| `onPrimarySourceChanged(rolla, change)` | The user's primary data source changed |
| `onGoalsChanged(rolla, change)` | The user saved goal changes inside the SDK UI (backend-confirmed) |
| `onProfileUpdated(rolla, update)` | The user updated profile data inside the SDK UI — carries only the changed fields |

All methods have default empty implementations, so existing integrations compile unchanged — implement only the ones you need.

## Host Events

The event callbacks above (`onActivityStarted` through `onProfileUpdated`) let your app observe what happens inside the SDK without polling. Delivery semantics:

- **Engine-scoped, engine-lifetime delivery.** Events are armed by any of `show()`, `warmUpEngine`, `getBandBatteryLevel`, `getPairedBandInfo`, or `syncHealthData`, and keep flowing after the SDK UI closes — an upload that completes moments after dismissal still reports. Delivery stops only at `destroyEngine()`. Nothing fires while the engine is cold, and nothing is delivered retroactively.
- **Main thread.** Like all SDK callbacks, events arrive on the main thread.
- **Activity lifecycle.** Every started activity terminates in a `FINISHED` completion or a removal — possibly in a *different app session* if the app dies in between (crash recovery resolves on the next launch, re-firing `onActivityStarted` with origin `CRASH_RECOVERY`). Two exceptions are cleaned up silently, without an event: a session abandoned mid-tracking for over a day, and an interrupted session neither resumed nor discarded before the user starts their next activity. Dedupe on `activityId`, and treat `(activityId, phase)` as the idempotency key for completions — `UPLOADED`/`UPLOAD_FAILED` can re-fire across retries. Manually logged activities enter the lifecycle at `FINISHED` (no started event); pause/resume inside a session fires nothing.
- **Band link events are not a proximity signal.** `onBandConnected`/`onBandDisconnected` report genuine BLE link transitions of the user's own band only: connect fires immediately, disconnect only after the BLE supervision timeout plus a ~3-second debounce (a drop with an immediate reconnect reports nothing). They are orthogonal to paired/unpaired — an unpair or logout drops the physical link too, so a disconnect legitimately accompanies those. Use `getPairedBandInfo` for the pairing state.
- **`syncedData` on UI syncs.** On a successful band / Health Connect UI sync, `RollaSyncResult.syncedData` carries the same per-stream summary as the headless result (samples never included). It is `null` when there is nothing attributable to report — failures, Garmin/Oura content-only refreshes, syncs that recorded nothing, or overlapping syncs — never wrong or double-reported data.

## Headless Methods

Four methods run **headlessly** — no SDK UI needs to be opened. Each starts the engine automatically on first use; `warmUpEngine(context, callback?)` only moves that one-time cost off the first call (a common pattern is warming up right after login so the first `show()` presents instantly). Because there is no UI to prompt from, **your app owns OS permissions**: when one is missing, the methods fail fast with a typed reason instead of prompting.

### `syncHealthData(context, includeSamples, callback)`

Runs a full sync of the user's primary data source (band over BLE, or Health Connect) and resolves to a typed `RollaSyncResult` — the call never throws, and the same result is also delivered to `onSyncHealthDataCompleted(rolla, result)`:

| Field | Meaning |
|-------|---------|
| `outcome` | `SUCCESS`, `SKIPPED` (expectedly did nothing — see `skipReason`), or `FAILURE` (see `error`) |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `BAND`, `APPLE_HEALTH`, `HEALTH_CONNECT`, `GARMIN`, `OURA` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device — together they give the sync duration. `startedAt` is `null` for `SKIPPED` (nothing ran) and on overlapping syncs; `lastSyncAt` is present only on success |
| `skipReason` | `NO_BAND_PAIRED` (no band on the account), `BAND_NOT_CONNECTED` (a band is paired but couldn't be reached right now), `ALREADY_IN_PROGRESS`, `SERVER_SIDE_SOURCE` (Garmin/Oura sync server-side), `BLUETOOTH_PERMISSION_REQUIRED`, `BLUETOOTH_UNAVAILABLE`, `APPLE_HEALTH_PERMISSION_REQUIRED`, `HEALTH_CONNECT_PERMISSION_REQUIRED`, `NOT_INITIALIZED`, `OFFLINE` |
| `syncedData` | Per-stream summary of what was uploaded; pass `includeSamples = true` to also receive raw sample arrays |

### `getBandBatteryLevel(context, callback)`

A **live BLE read** from the paired Rolla band — the band must be reachable. Resolves to a typed `RollaBatteryResult`: a percentage when `status` is `AVAILABLE`, otherwise a documented reason (`NO_BAND_PAIRED`, `BAND_NOT_CONNECTED` — a band is paired but couldn't be reached, `BLUETOOTH_UNAVAILABLE`, `BLUETOOTH_PERMISSION_REQUIRED`, `UNKNOWN_ERROR`). Never a stale value reported as live.

### `getPairedBandInfo(context, callback)`

Answers "does this account currently have a Rolla band?" with **zero Bluetooth** — no scan, no connect, no BLE permission; works with Bluetooth off. Resolves to a typed `RollaPairedBandResult`:

| Status | Meaning |
|--------|---------|
| `BAND_PAIRED` | A band is paired — `band` carries its MAC address (always present) plus the last cached battery/firmware/serial, each possibly null if the SDK hasn't read the band recently |
| `NO_BAND_PAIRED` | The user's profile confirms no band is paired |
| `UNKNOWN` | Could not be determined (offline with no local record) — reported instead of guessing |

The lookup is network-first: the profile is the authoritative pairing record, so a band unpaired remotely from another device is reported correctly. This is a pairing-state query, not a link-state one — live connect/disconnect transitions arrive via `onBandConnected`/`onBandDisconnected`.

```kotlin
rolla.getPairedBandInfo(context) { result ->
    result.onSuccess { info ->
        if (info.isPaired) println("Band: ${info.band!!.macAddress}")
    }
}
```

## RollaConfiguration

The `RollaConfiguration` class defines all parameters for SDK initialization. See [Code Integration](04-code-integration.md) for usage examples.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `String` | Yes | — | JWT access token from `POST /api/login` |
| `partnerId` | `String` | Yes | — | Partner identifier provided by Rolla |
| `refreshToken` | `String?` | No | `null` | Refresh token for automatic credential renewal |
| `tokenExpiresIn` | `Int?` | No | `null` | Token lifetime in seconds. Note: iOS uses `TimeInterval` (Double) for this parameter |
| `userId` | `String?` | No | Extracted from JWT | User identifier for local data namespacing (per-user storage isolation); defaults to the `sub` claim in the JWT if not provided. Not sent as a request header |
| `environment` | `String` | No | `"rnd"` | Target environment. See [Code Integration](04-code-integration.md) for available values |
| `disabledModules` | `Set<RollaDisabledModule>` | No | `emptySet()` (nothing disabled) | Modules whose entire UI is hidden across the SDK. See [Branding and Modules](05-branding-and-modules.md#module-configuration) and the [`RollaDisabledModule`](#rolladisabledmodule) values below |
| `disabledDataSources` | `Set<RollaDataSource>` | No | `emptySet()` (all offered) | Data sources whose connect option is hidden wherever the user picks a source to connect. See [Branding and Modules](05-branding-and-modules.md#data-source-configuration) and the [`RollaDataSource`](#rolladatasource) values below |
| `branding` | `RollaBranding?` | No | `null` | Visual identity: `hostAppName`, `primaryColor`, `defaultThemeMode` (`RollaThemeMode`), `headerLogoAsset`, `privacyUrl`, `removeRollaBandReferences` — every field optional; set fields override the SDK defaults individually. See [Branding and Modules](05-branding-and-modules.md) |
| `showSettingsButton` | `Boolean` | No | `true` | Render a Settings button on the Home screen, below the Metrics list. Tapping it opens a bottom sheet with shortcuts to Data Sources and Goals. Defaults to true because most partners need this button. |

### RollaDisabledModule

`disabledModules` accepts a set of `RollaDisabledModule` values. Each value passed hides that module's entire UI everywhere it appears in the SDK. Currently supported:

| Value | Hides |
|-------|-------|
| `RollaDisabledModule.WEIGHT` | The Weight tracking module |
| `RollaDisabledModule.BLOOD_PRESSURE` | The Blood Pressure tracking module |

More modules will become disable-able in future releases. Pass `emptySet()` (or omit the parameter) to keep every module enabled.

### RollaDataSource

`disabledDataSources` accepts a set of `RollaDataSource` values. Each value passed hides that source's connect option wherever the user picks a data source to connect (the Data Sources screen and the onboarding data-source step). A source the user has already connected stays visible for viewing/disconnecting; only new connections are suppressed. If you disable every source, the Rolla Band remains available as a floor.

| Value | Hides |
|-------|-------|
| `RollaDataSource.BAND` | The Rolla Band pairing option |
| `RollaDataSource.GARMIN` | Garmin Connect |
| `RollaDataSource.OURA` | Oura |
| `RollaDataSource.APPLE_HEALTH` | Apple Health (iOS only) |
| `RollaDataSource.HEALTH_CONNECT` | Health Connect |

Pass `emptySet()` (or omit the parameter) to offer every data source.

## Error Handling

The SDK provides detailed error information through RollaError:

```kotlin
sealed class RollaError(val code: String, val message: String) : Exception(message) {
    class EngineFailedToStart
    class InitializationFailed(details: String)
    class FlutterError(errorCode: String, errorMessage: String)
    class AlreadyPresenting
    class InvalidContext
    class Underlying(throwable: Throwable)
    class Unknown(details: String?)
}
```

## Close Reasons

The SDK provides close reasons through RollaCloseReason:

```kotlin
sealed class RollaCloseReason {
    data class FlutterRequested(val reason: String?)
    data object HostNavigationBack
    data object HostModalDismiss
    data object Programmatic
    data object HostStackReplaced
    data object Unknown
}
```

## Error Recovery Guide

Recommended host app actions for each `RollaError` subclass:

| Error Class | Code | Meaning | Host App Recovery |
|-------------|------|---------|-------------------|
| `EngineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `InitializationFailed(details)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `FlutterError(errorCode, errorMessage)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `AlreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `InvalidContext` | `INVALID_CONTEXT` | Activity/Fragment not in a valid state | Ensure the Activity is resumed or the Fragment is attached before calling `show()`. |
| `Underlying(Throwable)` | `UNDERLYING_ERROR` | Wraps a platform-native error | Inspect the wrapped throwable. Handle based on underlying cause. |
| `Unknown(details?)` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## Close Reason Reference

When each `RollaCloseReason` subclass is triggered:

| Close Reason | When Triggered |
|-------------|----------------|
| `FlutterRequested(reason)` | SDK's internal UI initiated the close (e.g., user tapped close/done). Optional `reason` may provide context. |
| `HostNavigationBack` | User pressed the system back button. |
| `HostModalDismiss` | User dismissed the modal via gesture or system action. |
| `Programmatic` | Host app called `dismiss()` programmatically. |
| `HostStackReplaced` | Host app replaced the navigation/activity stack while SDK was presenting. This can occur if the host app finishes the current Activity, launches a new task, or clears the back stack while the SDK is on screen. |
| `Unknown` | Close reason could not be determined. |

---

**Previous:** [Engine Lifecycle](07-engine-lifecycle.md) | **Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
