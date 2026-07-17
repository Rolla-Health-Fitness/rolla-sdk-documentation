# Public API Reference

The complete public API of the Rolla SDK on Android: the `Rolla` class, the `RollaListener` interface and its host events, the headless methods, and the error and close-reason types. `RollaConfiguration` and its option enums are documented on the [Configuration](05-configuration.md) page.

**On this page:** [Rolla Class](#rolla-class) · [RollaListener Interface](#rollalistener-interface) · [Host Events](#host-events) · [Headless Methods](#headless-methods) · [RollaError](#rollaerror) · [RollaCloseReason](#rollaclosereason)

## Rolla Class

One instance per configuration. Create it, assign the listener, and present:

```kotlin
val rolla = Rolla(configuration)
rolla.listener = rollaListener
rolla.show(activity)
```

### Presentation

| Method / Property | Description |
|-------------------|-------------|
| <code>Rolla(configuration:&nbsp;RollaConfiguration)</code> | Create an instance — see [Configuration](05-configuration.md) for every option |
| <code>var&nbsp;listener:&nbsp;RollaListener?</code> | Receives every callback — see [RollaListener](#rollalistener-interface) |
| <code>val&nbsp;isPresenting:&nbsp;Boolean</code> | `true` from `show()` until the SDK UI closes |
| <code>show(activity:&nbsp;Activity)</code> | Present the SDK UI from an Activity |
| <code>show(fragment:&nbsp;Fragment)</code> | Present the SDK UI from a Fragment (`androidx.fragment.app.Fragment`) |
| `dismiss()` | Dismiss the SDK UI; the engine stays alive — see [Engine Lifecycle](07-engine-lifecycle.md) |

### Session & Tokens

| Method | Description |
|--------|-------------|
| <code>updateToken(token,&nbsp;refreshToken?,&nbsp;expiresIn?,&nbsp;callback?)</code> | Push fresh credentials to the SDK. The `callback` is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. See [Token Management](06-token-management.md) |
| `clearSession(callback?)` | Purge all persisted session data (tokens, auth metadata) — call on logout |

### Headless & Engine

> **Headless** means callable without the SDK UI ever being opened — no `show()` needed, no screen presented. The SDK runs its engine invisibly in the background and hands your app a typed result.

| Method | Description |
|--------|-------------|
| <code>warmUpEngine(context,&nbsp;callback?)</code> | Start and configure the engine ahead of time, without any UI — see [Headless Methods](#warmupengine) |
| <code>syncHealthData(context,&nbsp;includeSamples&nbsp;=&nbsp;false,&nbsp;callback)</code> | Headless sync of the user's primary data source — see [Headless Methods](#synchealthdata) |
| <code>getBandBatteryLevel(context,&nbsp;callback)</code> | Headless live battery read from the paired Rolla band — see [Headless Methods](#getbandbatterylevel) |
| <code>getPairedBandInfo(context,&nbsp;callback)</code> | Headless paired-band query, zero Bluetooth — see [Headless Methods](#getpairedbandinfo) |
| <code>companion&nbsp;fun&nbsp;destroyEngine()</code> | Fully tear down the Flutter engine and free its memory — see [Engine Lifecycle](07-engine-lifecycle.md) |

## RollaListener Interface

All sixteen methods have default empty implementations — override only the ones you need, and existing integrations compile unchanged when new methods are added. The interface has two halves:

- **Presentation & token callbacks** (below) — the SDK needs your app to react: dismissal, errors, token exchange.
- **[Host events](#host-events)** — the SDK tells your app what happened inside it: syncs, activities, band, profile. Purely observational.

### Presentation & Token Callbacks

| Callback | Method | Called when |
|----------|--------|-------------|
| SDK&nbsp;closed | <code>onRollaClosed(rolla,&nbsp;reason)</code> | The SDK UI was dismissed — see [RollaCloseReason](#rollaclosereason) |
| Error&nbsp;occurred | <code>onRollaError(rolla,&nbsp;error)</code> | An error occurred — see [RollaError](#rollaerror) |
| Token&nbsp;refreshed | <code>onTokenRefreshed(rolla,&nbsp;token,&nbsp;refreshToken?,&nbsp;expiresIn?)</code> | The SDK refreshed tokens internally — store them for future use |
| Token&nbsp;refresh&nbsp;needed | `onTokenExpired(rolla)` | The SDK could not refresh the token — fetch new tokens from your backend and call `updateToken` |

## Host Events

Twelve listener methods push SDK events to your app, so you never have to poll. Two rules apply to all of them:

- **Engine-scoped, engine-lifetime delivery.** Events are armed by any of `show()`, `warmUpEngine()`, or any headless call, and keep flowing after the SDK UI closes — an upload that completes moments after dismissal still reports. Delivery stops only at `destroyEngine()`. Nothing fires while the engine is cold, and nothing is delivered retroactively.
- **Main thread.** Like all SDK callbacks, events arrive on the main thread.

### Sync Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Headless&nbsp;sync&nbsp;completed | <code>onSyncHealthDataCompleted(rolla,&nbsp;result)</code> | A headless [`syncHealthData`](#synchealthdata) reaches a terminal outcome — with the same `RollaSyncResult` the callback receives |
| UI&nbsp;sync&nbsp;completed | <code>onUiSyncCompleted(rolla,&nbsp;result)</code> | A sync completes inside the SDK UI (auto-sync on open, return from background, manual refresh) |

**`syncedData` on UI syncs.** On a successful band / Apple Health / Health Connect UI sync, `RollaSyncResult.syncedData` carries the same per-stream summary as the headless result (samples never included). It is `null` when there is nothing attributable to report — failures, Garmin/Oura content-only refreshes, syncs that recorded nothing, or overlapping syncs — never wrong or double-reported data.

### Activity Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Activity&nbsp;started | <code>onActivityStarted(rolla,&nbsp;activity)</code> | A live tracking session starts — `RollaStartedActivity.origin` distinguishes a fresh start from a crash-recovery resume |
| Activity&nbsp;completed | <code>onActivityCompleted(rolla,&nbsp;activity)</code> | An activity reaches a lifecycle phase: `FINISHED` (saved in-SDK), then `UPLOADED` or `UPLOAD_FAILED` |
| Activity&nbsp;removed | <code>onActivityRemoved(rolla,&nbsp;activity)</code> | An activity's record is removed without a kept result — `reason` is `CANCELED` (crash-recovery discard) or `DELETED` (user deleted it, backend-confirmed) |

**Lifecycle guarantees.** Every started activity terminates in a `FINISHED` completion or a removal — possibly in a *different app session* if the app dies in between (crash recovery resolves on the next launch, re-firing `onActivityStarted` with origin `CRASH_RECOVERY`). Two exceptions are cleaned up silently, without an event: a session abandoned mid-tracking for over a day, and an interrupted session neither resumed nor discarded before the user starts their next activity. Dedupe on `activityId`, and treat `(activityId, phase)` as the idempotency key for completions — `UPLOADED`/`UPLOAD_FAILED` can re-fire across retries. Manually logged activities enter the lifecycle at `FINISHED` (no started event); pause/resume inside a session fires nothing.

### Band Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Band&nbsp;paired | <code>onBandPaired(rolla,&nbsp;band)</code> | The user pairs a band inside the SDK UI |
| Band&nbsp;unpaired | <code>onBandUnpaired(rolla,&nbsp;band)</code> | The user unpairs the band inside the SDK UI (backend-confirmed) |
| Band&nbsp;connected | <code>onBandConnected(rolla,&nbsp;band)</code> | The paired band establishes a live BLE link |
| Band&nbsp;disconnected | <code>onBandDisconnected(rolla,&nbsp;band)</code> | The paired band loses its live BLE link (debounced a few seconds) |

**Link events are not a proximity signal.** `onBandConnected`/`onBandDisconnected` report genuine BLE link transitions of the user's own band only: connect fires immediately, disconnect only after the BLE supervision timeout plus a ~3-second debounce (a drop with an immediate reconnect reports nothing). They are orthogonal to paired/unpaired — an unpair or logout drops the physical link too, so a disconnect legitimately accompanies those. Use [`getPairedBandInfo`](#getpairedbandinfo) for the pairing state.

### Profile & Settings Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Primary&nbsp;source&nbsp;changed | <code>onPrimarySourceChanged(rolla,&nbsp;change)</code> | The user's primary data source changes |
| Goals&nbsp;changed | <code>onGoalsChanged(rolla,&nbsp;change)</code> | The user saves goal changes inside the SDK UI (backend-confirmed) — one call per save |
| Profile&nbsp;updated | <code>onProfileUpdated(rolla,&nbsp;update)</code> | The user updates profile data inside the SDK UI — carries only the changed fields |

## Headless Methods

Four methods run **headlessly** — no SDK UI needs to be opened. Each starts the engine automatically on first use. Because there is no UI to prompt from, **your app owns OS permissions**: when one is missing, the methods fail fast with a typed reason instead of prompting.

### warmUpEngine

```kotlin
fun warmUpEngine(context: Context, callback: ((Result<Unit>) -> Unit)? = null)
```

Starts and configures the engine ahead of time so the headless calls — and the first `show()` — have zero start-up latency. Optional: the methods below warm the engine themselves if needed; this only moves the one-time cost to a moment you control (a common pattern is right after login). Safe to call repeatedly — a repeat call for the same user is a no-op that preserves the session. See [Engine Lifecycle](07-engine-lifecycle.md#warming-up-the-engine).

### syncHealthData

```kotlin
fun syncHealthData(context: Context, includeSamples: Boolean = false, callback: (Result<RollaSyncResult>) -> Unit)
```

Runs a full sync of the user's primary data source (band over BLE, or Health Connect) and resolves to a typed `RollaSyncResult` — the call never throws, and the same result is also delivered to `onSyncHealthDataCompleted(rolla, result)`:

| Field | Meaning |
|-------|---------|
| `outcome` | `SUCCESS`, `SKIPPED` (expectedly did nothing — see `skipReason`), or `FAILURE` (see `error`) |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `BAND`, `APPLE_HEALTH`, `HEALTH_CONNECT`, `GARMIN`, `OURA` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device — together they give the sync duration. `startedAt` is `null` for `SKIPPED` (nothing ran) and on overlapping syncs; `lastSyncAt` is present only on success |
| `skipReason` | `NO_BAND_PAIRED` (no band on the account), `BAND_NOT_CONNECTED` (a band is paired but couldn't be reached right now), `ALREADY_IN_PROGRESS`, `SERVER_SIDE_SOURCE` (Garmin/Oura sync server-side), `BLUETOOTH_PERMISSION_REQUIRED`, `BLUETOOTH_UNAVAILABLE`, `APPLE_HEALTH_PERMISSION_REQUIRED`, `HEALTH_CONNECT_PERMISSION_REQUIRED`, `NOT_INITIALIZED`, `OFFLINE` |
| `syncedData` | Per-stream summary of what was uploaded; pass `includeSamples = true` to also receive raw sample arrays |

### getBandBatteryLevel

```kotlin
fun getBandBatteryLevel(context: Context, callback: (Result<RollaBatteryResult>) -> Unit)
```

A **live BLE read** from the paired Rolla band — the band must be reachable. Resolves to a typed `RollaBatteryResult`: a percentage when `status` is `AVAILABLE`, otherwise a documented reason (`NO_BAND_PAIRED`, `BAND_NOT_CONNECTED` — a band is paired but couldn't be reached, `NOT_ROLLA_DEVICE` — reserved for forward compatibility, not currently returned, `BLUETOOTH_UNAVAILABLE`, `BLUETOOTH_PERMISSION_REQUIRED`, `UNKNOWN_ERROR`). Never a stale value reported as live.

### getPairedBandInfo

```kotlin
fun getPairedBandInfo(context: Context, callback: (Result<RollaPairedBandResult>) -> Unit)
```

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

## RollaError

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

Recommended host app actions for each subclass:

| Error Class | Code | Meaning | Host App Recovery |
|-------------|------|---------|-------------------|
| `EngineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `InitializationFailed(details)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `FlutterError(errorCode, errorMessage)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `AlreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `InvalidContext` | `INVALID_CONTEXT` | Activity/Fragment not in a valid state | Ensure the Activity is resumed or the Fragment is attached before calling `show()`. |
| `Underlying(Throwable)` | `UNDERLYING_ERROR` | Wraps a platform-native error | Inspect the wrapped throwable. Handle based on underlying cause. |
| `Unknown(details?)` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## RollaCloseReason

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

When each reason is triggered:

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
