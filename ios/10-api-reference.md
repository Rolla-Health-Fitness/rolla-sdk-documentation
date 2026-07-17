# API Reference

This section provides a comprehensive reference of the Rolla SDK's public API, including the main Rolla class, delegate protocol, and error types. `RollaConfiguration` and its option enums are documented on the [Configuration](05-configuration.md) page.

## Native API Reference

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `init(configuration: RollaConfiguration)` | Initialize with a configuration — see [Configuration](05-configuration.md) for every option |
| `var delegate: RollaDelegate?` | Set the delegate for callbacks |
| `var isPresenting: Bool` | Whether the SDK is currently visible |
| `show(from: UIViewController)` | Present the SDK modally |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token:refreshToken:expiresIn:completion:)` | Push fresh credentials to the SDK. The `completion` handler is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. |
| `clearSession(completion:)` | Clear all persisted session data |
| `warmUpEngine(completion:)` | Start and configure the engine ahead of time, without any UI. See [Headless Methods](#headless-methods) |
| `getBandBatteryLevel(completion:)` | Headless live battery read from the paired Rolla band. See [Headless Methods](#headless-methods) |
| `getPairedBandInfo(completion:)` | Headless paired-band query — zero Bluetooth. See [Headless Methods](#headless-methods) |
| `syncHealthData(includeSamples:completion:)` | Headless sync of the user's primary data source. See [Headless Methods](#headless-methods) |
| `static destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaDelegate Protocol

| Method | Description |
|--------|-------------|
| `rollaDidClose(_:reason:)` | Called when the SDK UI is dismissed |
| `rollaDidFailWithError(_:error:)` | Called when an error occurs |
| `rollaDidRefreshToken(_:token:refreshToken:expiresIn:)` | Called when the SDK refreshes tokens internally |
| `rollaDidRequestTokenRefresh(_:)` | Called when the host app must provide new tokens |
| `rollaDidCompleteHealthDataSync(_:result:)` | Called when a headless `syncHealthData` reaches a terminal outcome, with the same `RollaSyncResult` the completion handler receives |
| `rollaDidStartActivity(_:activity:)` | A live tracking session started — `RollaStartedActivity.origin` distinguishes a fresh start from a crash-recovery resume. See [Host Events](#host-events) |
| `rollaDidCompleteActivity(_:activity:)` | An activity reached a lifecycle phase: `finished` (saved in-SDK), then `uploaded` or `uploadFailed`. Key idempotency on `(activityId, phase)` |
| `rollaDidRemoveActivity(_:activity:)` | An activity's record was removed without a kept result — `reason` is `canceled` (crash-recovery discard; the "activity canceled" case) or `deleted` (user deleted it, backend-confirmed) |
| `rollaDidCompleteUISync(_:result:)` | A sync completed inside the SDK UI (auto-sync on open, return from background, manual refresh) |
| `rollaDidPairBand(_:band:)` | The user paired a band inside the SDK UI |
| `rollaDidUnpairBand(_:band:)` | The user unpaired the band inside the SDK UI (backend-confirmed) |
| `rollaDidConnectBand(_:band:)` | The paired band established a live BLE link. See [Host Events](#host-events) for the link-event caveats |
| `rollaDidDisconnectBand(_:band:)` | The paired band lost its live BLE link (debounced a few seconds) |
| `rollaDidChangePrimarySource(_:change:)` | The user's primary data source changed |
| `rollaDidChangeGoals(_:change:)` | The user saved goal changes inside the SDK UI (backend-confirmed) |
| `rollaDidUpdateProfile(_:update:)` | The user updated profile data inside the SDK UI — carries only the changed fields |

All methods have default empty implementations, so existing integrations compile unchanged — implement only the ones you need.

## Host Events

The event callbacks above (`rollaDidStartActivity` through `rollaDidUpdateProfile`) let your app observe what happens inside the SDK without polling. Delivery semantics:

- **Engine-scoped, engine-lifetime delivery.** Events are armed by any of `show(from:)`, `warmUpEngine`, `getBandBatteryLevel`, `getPairedBandInfo`, or `syncHealthData`, and keep flowing after the SDK UI closes — an upload that completes moments after dismissal still reports. Delivery stops only at `destroyEngine()`. Nothing fires while the engine is cold, and nothing is delivered retroactively.
- **Main thread.** Like all SDK callbacks, events arrive on the main thread.
- **Activity lifecycle.** Every started activity terminates in a `finished` completion or a removal — possibly in a *different app session* if the app dies in between (crash recovery resolves on the next launch, re-firing `rollaDidStartActivity` with origin `crashRecovery`). Two exceptions are cleaned up silently, without an event: a session abandoned mid-tracking for over a day, and an interrupted session neither resumed nor discarded before the user starts their next activity. Dedupe on `activityId`, and treat `(activityId, phase)` as the idempotency key for completions — `uploaded`/`uploadFailed` can re-fire across retries. Manually logged activities enter the lifecycle at `finished` (no started event); pause/resume inside a session fires nothing.
- **Band link events are not a proximity signal.** `rollaDidConnectBand`/`rollaDidDisconnectBand` report genuine BLE link transitions of the user's own band only: connect fires immediately, disconnect only after the BLE supervision timeout plus a ~3-second debounce (a drop with an immediate reconnect reports nothing). They are orthogonal to paired/unpaired — an unpair or logout drops the physical link too, so a disconnect legitimately accompanies those. Use `getPairedBandInfo` for the pairing state.
- **`syncedData` on UI syncs.** On a successful band / Apple Health / Health Connect UI sync, `RollaSyncResult.syncedData` carries the same per-stream summary as the headless result (samples never included). It is `nil` when there is nothing attributable to report — failures, Garmin/Oura content-only refreshes, syncs that recorded nothing, or overlapping syncs — never wrong or double-reported data.

## Headless Methods

Four methods run **headlessly** — no SDK UI needs to be opened. Each starts the engine automatically on first use; `warmUpEngine(completion:)` only moves that one-time cost off the first call (a common pattern is warming up right after login so the first `show(from:)` presents instantly). Because there is no UI to prompt from, **your app owns OS permissions**: when one is missing, the methods fail fast with a typed reason instead of prompting.

### `syncHealthData(includeSamples:completion:)`

Runs a full sync of the user's primary data source (band over BLE, or Apple Health) and resolves to a typed `RollaSyncResult` — the call never throws, and the same result is also delivered to `rollaDidCompleteHealthDataSync(_:result:)`:

| Field | Meaning |
|-------|---------|
| `outcome` | `success`, `skipped` (expectedly did nothing — see `skipReason`), or `failure` (see `error`) |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `band`, `appleHealth`, `healthConnect`, `garmin`, `oura` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device — together they give the sync duration. `startedAt` is `nil` for `skipped` (nothing ran) and on overlapping syncs; `lastSyncAt` is present only on success |
| `skipReason` | `noBandPaired` (no band on the account), `bandNotConnected` (a band is paired but couldn't be reached right now), `alreadyInProgress`, `serverSideSource` (Garmin/Oura sync server-side), `bluetoothPermissionRequired`, `bluetoothUnavailable`, `appleHealthPermissionRequired`, `healthConnectPermissionRequired`, `notInitialized`, `offline` |
| `syncedData` | Per-stream summary of what was uploaded; pass `includeSamples: true` to also receive raw sample arrays |

### `getBandBatteryLevel(completion:)`

A **live BLE read** from the paired Rolla band — the band must be reachable. Resolves to a typed `RollaBatteryResult`: a percentage when `status` is `available`, otherwise a documented reason (`noBandPaired`, `bandNotConnected` — a band is paired but couldn't be reached, `notRollaDevice` — reserved for forward compatibility, not currently returned, `bluetoothUnavailable`, `bluetoothPermissionRequired`, `unknownError`). Never a stale value reported as live.

### `getPairedBandInfo(completion:)`

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

## Error Handling

The SDK provides detailed error information through `RollaError`:

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

## Close Reasons

The SDK provides close reasons through `RollaCloseReason`:

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

## Error Recovery Guide

Recommended host app actions for each `RollaError` case:

| Error Case | Code | Meaning | Host App Recovery |
|------------|------|---------|-------------------|
| `.engineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `.initializationFailed(String)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `.flutterError(code:message:)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `.alreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `.invalidPresentationContext` | `INVALID_CONTEXT` | View controller not in window hierarchy | Ensure the view controller is visible and in the hierarchy before calling `show(from:)`. |
| `.underlying(Error)` | `UNDERLYING_ERROR` | Wraps a native error | Inspect the wrapped error. Handle based on underlying cause. |
| `.unknown` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## Close Reason Reference

When each `RollaCloseReason` is triggered:

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
