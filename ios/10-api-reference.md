# API Reference

The complete public API of the Rolla SDK on iOS: the `Rolla` class, the `RollaDelegate` protocol and its host events, the headless methods, and the error and close-reason types. `RollaConfiguration` and its option enums are documented on the [Configuration](05-configuration.md) page.

**On this page:** [Rolla Class](#rolla-class) · [RollaDelegate Protocol](#rolladelegate-protocol) · [Host Events](#host-events) · [Headless Methods](#headless-methods) · [RollaError](#rollaerror) · [RollaCloseReason](#rollaclosereason)

## Rolla Class

One instance per configuration. Create it, assign the delegate, and present:

```swift
let rolla = Rolla(configuration: configuration)
rolla.delegate = self
rolla.show(from: self)
```

### Presentation

| Method / Property | Description |
|-------------------|-------------|
| `init(configuration:)` | Create an instance — see [Configuration](05-configuration.md) for every option |
| `var delegate: RollaDelegate?` | Receives every callback — see [RollaDelegate](#rolladelegate-protocol) |
| `var isPresenting: Bool` | `true` from `show(from:)` until the SDK UI closes — the same flag behind the `.alreadyPresenting` guard |
| `show(from: UIViewController)` | Present the SDK UI modally |
| `dismiss()` | Dismiss the SDK UI; the engine stays alive — see [Engine Lifecycle](08-engine-lifecycle.md) |

### Session & Tokens

| Method | Description |
|--------|-------------|
| `updateToken(token:refreshToken:expiresIn:completion:)` | Push fresh credentials to the SDK. The `completion` handler is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. See [Token Management](07-token-management.md) |
| `clearSession(completion:)` | Purge all persisted session data (tokens, auth metadata) — call on logout |

### Headless & Engine

> **Headless** means callable without the SDK UI ever being opened — no `show(from:)` needed, no screen presented. The SDK runs its engine invisibly in the background and hands your app a typed result.

| Method | Description |
|--------|-------------|
| `warmUpEngine(completion:)` | Start and configure the engine ahead of time, without any UI — see [Headless Methods](#warmupengine) |
| `syncHealthData(includeSamples:completion:)` | Headless sync of the user's primary data source — see [Headless Methods](#synchealthdata) |
| `getBandBatteryLevel(completion:)` | Headless live battery read from the paired Rolla band — see [Headless Methods](#getbandbatterylevel) |
| `getPairedBandInfo(completion:)` | Headless paired-band query, zero Bluetooth — see [Headless Methods](#getpairedbandinfo) |
| `static destroyEngine()` | Fully tear down the Flutter engine and free its memory — see [Engine Lifecycle](08-engine-lifecycle.md) |

## RollaDelegate Protocol

All sixteen methods have default empty implementations — implement only the ones you need, and existing integrations compile unchanged when new methods are added. The protocol has two halves:

- **Presentation & token callbacks** (below) — the SDK needs your app to react: dismissal, errors, token exchange.
- **[Host events](#host-events)** — the SDK tells your app what happened inside it: syncs, activities, band, profile. Purely observational.

### Presentation & Token Callbacks

| Callback | Method | Called when |
|----------|--------|-------------|
| SDK closed | `rollaDidClose(_:reason:)` | The SDK UI was dismissed — see [RollaCloseReason](#rollaclosereason) |
| Error occurred | `rollaDidFailWithError(_:error:)` | An error occurred — see [RollaError](#rollaerror) |
| Token refreshed | `rollaDidRefreshToken(_:token:refreshToken:expiresIn:)` | The SDK refreshed tokens internally — store them for future use |
| Token refresh needed | `rollaDidRequestTokenRefresh(_:)` | The SDK could not refresh the token — fetch new tokens from your backend and call `updateToken` |

## Host Events

Twelve delegate methods push SDK events to your app, so you never have to poll. Two rules apply to all of them:

- **Engine-scoped, engine-lifetime delivery.** Events are armed by any of `show(from:)`, `warmUpEngine`, or any headless call, and keep flowing after the SDK UI closes — an upload that completes moments after dismissal still reports. Delivery stops only at `destroyEngine()`. Nothing fires while the engine is cold, and nothing is delivered retroactively.
- **Main thread.** Like all SDK callbacks, events arrive on the main thread.

### Sync Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Headless sync completed | `rollaDidCompleteHealthDataSync(_:result:)` | A headless [`syncHealthData`](#synchealthdata) reaches a terminal outcome — with the same `RollaSyncResult` the completion handler receives |
| UI sync completed | `rollaDidCompleteUISync(_:result:)` | A sync completes inside the SDK UI (auto-sync on open, return from background, manual refresh) |

**`syncedData` on UI syncs.** On a successful band / Apple Health / Health Connect UI sync, `RollaSyncResult.syncedData` carries the same per-stream summary as the headless result (samples never included). It is `nil` when there is nothing attributable to report — failures, Garmin/Oura content-only refreshes, syncs that recorded nothing, or overlapping syncs — never wrong or double-reported data.

### Activity Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Activity started | `rollaDidStartActivity(_:activity:)` | A live tracking session starts — `RollaStartedActivity.origin` distinguishes a fresh start from a crash-recovery resume |
| Activity completed | `rollaDidCompleteActivity(_:activity:)` | An activity reaches a lifecycle phase: `finished` (saved in-SDK), then `uploaded` or `uploadFailed` |
| Activity removed | `rollaDidRemoveActivity(_:activity:)` | An activity's record is removed without a kept result — `reason` is `canceled` (crash-recovery discard) or `deleted` (user deleted it, backend-confirmed) |

**Lifecycle guarantees.** Every started activity terminates in a `finished` completion or a removal — possibly in a *different app session* if the app dies in between (crash recovery resolves on the next launch, re-firing `rollaDidStartActivity` with origin `crashRecovery`). Two exceptions are cleaned up silently, without an event: a session abandoned mid-tracking for over a day, and an interrupted session neither resumed nor discarded before the user starts their next activity. Dedupe on `activityId`, and treat `(activityId, phase)` as the idempotency key for completions — `uploaded`/`uploadFailed` can re-fire across retries. Manually logged activities enter the lifecycle at `finished` (no started event); pause/resume inside a session fires nothing.

### Band Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Band paired | `rollaDidPairBand(_:band:)` | The user pairs a band inside the SDK UI |
| Band unpaired | `rollaDidUnpairBand(_:band:)` | The user unpairs the band inside the SDK UI (backend-confirmed) |
| Band connected | `rollaDidConnectBand(_:band:)` | The paired band establishes a live BLE link |
| Band disconnected | `rollaDidDisconnectBand(_:band:)` | The paired band loses its live BLE link (debounced a few seconds) |

**Link events are not a proximity signal.** `rollaDidConnectBand`/`rollaDidDisconnectBand` report genuine BLE link transitions of the user's own band only: connect fires immediately, disconnect only after the BLE supervision timeout plus a ~3-second debounce (a drop with an immediate reconnect reports nothing). They are orthogonal to paired/unpaired — an unpair or logout drops the physical link too, so a disconnect legitimately accompanies those. Use [`getPairedBandInfo`](#getpairedbandinfo) for the pairing state.

### Profile & Settings Events

| Event | Method | Fires when |
|-------|--------|-----------|
| Primary source changed | `rollaDidChangePrimarySource(_:change:)` | The user's primary data source changes |
| Goals changed | `rollaDidChangeGoals(_:change:)` | The user saves goal changes inside the SDK UI (backend-confirmed) — one call per save |
| Profile updated | `rollaDidUpdateProfile(_:update:)` | The user updates profile data inside the SDK UI — carries only the changed fields |

## Headless Methods

Four methods run **headlessly** — no SDK UI needs to be opened. Each starts the engine automatically on first use. Because there is no UI to prompt from, **your app owns OS permissions**: when one is missing, the methods fail fast with a typed reason instead of prompting.

### warmUpEngine

```swift
func warmUpEngine(completion: ((Result<Void, RollaError>) -> Void)? = nil)
```

Starts and configures the engine ahead of time so the headless calls — and the first `show(from:)` — have zero start-up latency. Optional: the methods below warm the engine themselves if needed; this only moves the one-time cost to a moment you control (a common pattern is right after login). Safe to call repeatedly — a repeat call for the same user is a no-op that preserves the session. See [Engine Lifecycle](08-engine-lifecycle.md#warming-up-the-engine).

### syncHealthData

```swift
func syncHealthData(includeSamples: Bool = false, completion: @escaping (Result<RollaSyncResult, RollaError>) -> Void)
```

Runs a full sync of the user's primary data source (band over BLE, or Apple Health) and resolves to a typed `RollaSyncResult` — the call never throws, and the same result is also delivered to `rollaDidCompleteHealthDataSync(_:result:)`:

| Field | Meaning |
|-------|---------|
| `outcome` | `success`, `skipped` (expectedly did nothing — see `skipReason`), or `failure` (see `error`) |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `band`, `appleHealth`, `healthConnect`, `garmin`, `oura` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device — together they give the sync duration. `startedAt` is `nil` for `skipped` (nothing ran) and on overlapping syncs; `lastSyncAt` is present only on success |
| `skipReason` | `noBandPaired` (no band on the account), `bandNotConnected` (a band is paired but couldn't be reached right now), `alreadyInProgress`, `serverSideSource` (Garmin/Oura sync server-side), `bluetoothPermissionRequired`, `bluetoothUnavailable`, `appleHealthPermissionRequired`, `healthConnectPermissionRequired`, `notInitialized`, `offline` |
| `syncedData` | Per-stream summary of what was uploaded; pass `includeSamples: true` to also receive raw sample arrays |

### getBandBatteryLevel

```swift
func getBandBatteryLevel(completion: @escaping (Result<RollaBatteryResult, RollaError>) -> Void)
```

A **live BLE read** from the paired Rolla band — the band must be reachable. Resolves to a typed `RollaBatteryResult`: a percentage when `status` is `available`, otherwise a documented reason (`noBandPaired`, `bandNotConnected` — a band is paired but couldn't be reached, `notRollaDevice` — reserved for forward compatibility, not currently returned, `bluetoothUnavailable`, `bluetoothPermissionRequired`, `unknownError`). Never a stale value reported as live.

### getPairedBandInfo

```swift
func getPairedBandInfo(completion: @escaping (Result<RollaPairedBandResult, RollaError>) -> Void)
```

Answers "does this account currently have a Rolla band?" with **zero Bluetooth** — no scan, no connect, no BLE permission; works with Bluetooth off. Resolves to a typed `RollaPairedBandResult`:

| Status | Meaning |
|--------|---------|
| `bandPaired` | A band is paired — `band` carries its MAC address (always present) plus the last cached battery/firmware/serial, each possibly nil if the SDK hasn't read the band recently |
| `noBandPaired` | The user's profile confirms no band is paired |
| `unknown` | Could not be determined (offline with no local record) — reported instead of guessing |

The lookup is network-first: the profile is the authoritative pairing record, so a band unpaired remotely from another device is reported correctly. This is a pairing-state query, not a link-state one — live connect/disconnect transitions arrive via `rollaDidConnectBand`/`rollaDidDisconnectBand`.

```swift
rolla.getPairedBandInfo { result in
    if case .success(let info) = result, info.isPaired {
        print("Band: \(info.band!.macAddress)")
    }
}
```

## RollaError

```swift
public enum RollaError: Error {
    case engineFailedToStart
    case initializationFailed(String)
    case flutterError(code: String, message: String)
    case alreadyPresenting
    case invalidPresentationContext
    case underlying(Error)
    case unknown
}
```

Recommended host app actions for each case:

| Error Case | Code | Meaning | Host App Recovery |
|------------|------|---------|-------------------|
| `.engineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `.initializationFailed(String)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `.flutterError(code:message:)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `.alreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `.invalidPresentationContext` | `INVALID_CONTEXT` | View controller not in window hierarchy | Ensure the view controller is visible and in the hierarchy before calling `show(from:)`. |
| `.underlying(Error)` | `UNDERLYING_ERROR` | Wraps a native error | Inspect the wrapped error. Handle based on underlying cause. |
| `.unknown` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## RollaCloseReason

```swift
enum RollaCloseReason {
    case flutterRequested(reason: String?)
    case hostNavigationBack
    case hostModalDismiss
    case programmatic
    case hostStackReplaced
    case unknown
}
```

When each reason is triggered:

| Close Reason | When Triggered |
|-------------|----------------|
| `.flutterRequested(reason:)` | SDK's internal UI initiated the close (e.g., user tapped close/done). Optional `reason` may provide context. |
| `.hostNavigationBack` | User pressed back gesture or navigation back. |
| `.hostModalDismiss` | User dismissed the modal via swipe-down gesture. |
| `.programmatic` | Host app called `dismiss()` programmatically. |
| `.hostStackReplaced` | Host app replaced the navigation stack while SDK was presenting. This can occur if the host app programmatically pops multiple view controllers or pushes a new root while the SDK is on screen. |
| `.unknown` | Close reason could not be determined. |

---

**Previous:** [Live Activities](09-live-activities.md) | **Next:** [Troubleshooting](11-troubleshooting.md) | **Home:** [README](README.md)
