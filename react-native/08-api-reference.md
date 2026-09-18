# Public API Reference

The complete public API of `@rolla-health/react-native-sdk`: the `Rolla` class, host-driven navigation and notification taps, all 17 events, the headless methods, the TypeScript types, and the error and close-reason values. `RollaConfiguration` and its option unions are documented on the [Configuration](05-configuration.md) page.

**On this page:** [Rolla Class](#rolla-class) · [Host-Driven Navigation](#host-driven-navigation) · [Notification Taps](#notification-taps) · [Events](#events) · [Headless Methods](#headless-methods) · [Types](#types) · [Errors](#errors) · [RollaCloseReason](#rollaclosereason)

## Rolla Class

All methods are static and return promises. Every entry point takes the configuration it runs under, exactly as a native host builds a `Rolla(configuration)` per call; the SDK engine itself is process-wide and shared by all of them.

```ts
import { Rolla } from '@rolla-health/react-native-sdk';

const close = await Rolla.show(configuration);
```

### Presentation

| Method | Description |
|--------|-------------|
| `show(config, options?)` | Present the SDK UI. Resolves with a [`RollaCloseEvent`](#rollacloseevent) when it is dismissed; rejects with `{ code, message }` when it could not be presented — see [Errors](#errors). `options.transition` selects the open/close animation — see [Transition](05-configuration.md#transition) |
| `openScreen(config, screen, options?)` | Open the SDK UI directly on a specific screen — see [Host-Driven Navigation](#host-driven-navigation) |
| `dismiss()` | Dismiss the SDK UI; the engine stays alive — see [Engine Lifecycle](07-engine-lifecycle.md) |
| `isPresenting()` | Resolves `true` from `show()` — or an `openScreen()` that presents — until the SDK UI closes |
| `getInitialNotificationTarget()` | The Rolla notification tap that launched or resumed the app, or `null`; clears on read — see [Notification Taps](#notification-taps) |
| `notificationTarget(payload)` | Resolve a notification payload your own notification handling received; `null` when it is not Rolla's — see [Notification Taps](#notification-taps) |

### Session & Tokens

| Method | Description |
|--------|-------------|
| `updateToken(token, refreshToken?, expiresIn?)` | Push fresh credentials to the SDK — the answer to `onTokenExpired`, or a proactive push. Needs a running engine (`NO_ACTIVE_SESSION` otherwise); an older pair than the SDK holds is ignored and still resolves. See [Token Management](06-token-management.md) |
| `clearSession(config?)` | Purge all persisted session data (tokens, auth metadata) — call on logout. With `config`, warms the engine first when none is running; without it, a cold engine rejects with `NO_ACTIVE_SESSION`. Call `destroyEngine()` after it resolves |

### Headless & Engine

> **Headless** means callable without the SDK UI ever being opened — no `show()` needed, no screen presented. The SDK runs its engine invisibly in the background and hands your app a typed result.

| Method | Description |
|--------|-------------|
| `warmUpEngine(config)` | Start and configure the engine ahead of time, without any UI — see [Headless Methods](#warmupengine) |
| `syncHealthData(config, options?)` | Headless sync of the user's primary data source — see [Headless Methods](#synchealthdata) |
| `getBandBatteryLevel(config)` | Headless live battery read from the paired Rolla band — see [Headless Methods](#getbandbatterylevel) |
| `getPairedBandInfo(config)` | Headless paired-band query, zero Bluetooth — see [Headless Methods](#getpairedbandinfo) |
| `destroyEngine()` | Fully tear down the Flutter engine and free its memory — see [Engine Lifecycle](07-engine-lifecycle.md) |
| `getNativeSdkVersion()` | The native SDK version this package links (`'0.1.15'` for package `0.1.16`) — see [Prerequisites → Versioning](01-prerequisites.md#versioning) |

### Events

| Method | Description |
|--------|-------------|
| `addListener(event, listener)` | Subscribe to any event in the [Events](#events) table. Returns a `RollaSubscription` whose `remove()` you call in your effect cleanup |
| `removeAllListeners()` | Drop every subscription made through `addListener` — a hard reset, useful between integration tests |

## Host-Driven Navigation

### openScreen

```ts
Rolla.openScreen(
  config: RollaConfiguration,
  screen: RollaScreen,
  options?: { transition?: 'default' | 'fade' },
): Promise<RollaOpenScreenStatus>
```

Opens the SDK UI directly on a specific screen. If the SDK UI is hidden, the call presents it, animating in with `transition`. If the SDK UI is already visible, it just switches to the requested screen. The opened screen becomes the **root of the SDK UI**, so back returns the user straight to your app — never to an SDK Home screen the user did not visit. Each subsequent call replaces the root with the new screen. Close events arrive through `onClose`, exactly like a `show()`.

When the SDK UI is not showing, what happens next depends on the engine:

- **Warm engine**: the SDK stays hidden while it navigates, and is presented only if the request resolves as `'opened'`. Every other status leaves the SDK hidden and tells you why it was not presented. The engine is warm after a prior `show()`, a `warmUpEngine()`, or any headless call.
- **Cold engine**: it must present before it can navigate, so the SDK opens behind its loader and resolves the request while starting up. A failure such as `'screenDisabled'` therefore leaves the SDK on the Home screen, and the status tells you why the requested screen could not be presented.

To avoid the cold start entirely, call `warmUpEngine()` before the first `openScreen()` — typically right after login.

```ts
const status = await Rolla.openScreen(configuration, 'insights', { transition: 'fade' });
if (status !== 'opened') {
  console.log('Not opened:', status);
}
```

### RollaScreen

The screens your app can open directly — a deliberate whitelist:

| Value | Opens |
|-------|-------|
| `'activityHistory'` | The activity history list |
| `'goals'` | The goals editor |
| `'home'` | The SDK Home screen — restores Home as the root |
| `'insights'` | The insights feed — requires the insights module to be enabled (see [RollaDisabledModule](05-configuration.md#rolladisabledmodule)) |
| `'resume'` | No navigation at all: the SDK exactly as the user left it — the last opened screen while the engine stays alive, or Home on a fresh engine. Always resolves as `'opened'` |

### RollaOpenScreenStatus

Every outcome is a typed status — the call never fails silently:

| Status | Meaning |
|--------|---------|
| `'opened'` | The SDK UI is on the requested screen |
| `'screenDisabled'` | The screen's module is in `disabledModules` (e.g. `'insights'` with the insights module disabled) — nothing was opened |
| `'blockedByGate'` | A mandatory startup step (onboarding, consent, permissions, data-source connection) must be completed first. If the engine is cold, the SDK opens on that step; if it is warm and the SDK is hidden, it stays hidden |
| `'uiUnavailable'` | The SDK UI could not be shown — no foreground activity or view controller to present from, or the SDK never became ready to navigate |
| `'superseded'` | A newer `openScreen()` request replaced this one while waiting for the UI — only the latest request is honored |
| `'notInitialized'` / `'unknownError'` | Internal problems; neither is an expected runtime condition |

Presentation failures additionally fire `onError` exactly as a failed `show()` would — the status is additive, not a replacement for the event. An unknown `screen` or `transition` value rejects the promise with `INVALID_CONFIG`.

## Notification Taps

The SDK posts its own notifications (a workout in progress, reminders, a permission warning). Every one of them carries a payload that names its destination, and the wrapper resolves it to a [`RollaNotificationTarget`](#rollanotificationtarget): `{ kind: 'screen', screen }` to route with `openScreen()`, or `{ kind: 'appSettings' }` to send the user to the OS app-settings page.

Taps reach JavaScript in two ways, so handle both:

- **While the app is running**, a tap arrives as the `onNotificationTap` event.
- **When a tap launches the app** — or resumes it before your listeners are attached — call `Rolla.getInitialNotificationTarget()` once your app is up and route the result. It resolves `null` for a plain launch and clears on read, so an old tap never replays.

```ts
Rolla.getInitialNotificationTarget(): Promise<RollaNotificationTarget | null>
Rolla.notificationTarget(payload: Record<string, unknown>): Promise<RollaNotificationTarget | null>
```

`notificationTarget()` is for apps whose own notification handling — a push library, for example — already owns the tap: hand it the notification's user-info dictionary on iOS, or `{ payload }` with the intent's `payload` string extra on Android. `null` means the notification is not Rolla's.

These are the notifications the SDK posts and where a tap leads (English copy shown; the SDK localizes the text):

| Notification | When the SDK posts it | Tap resolves to |
|--------------|-----------------------|-----------------|
| **Background tracking disabled** | The SDK UI leaves the foreground mid-workout — the app goes to the background, or your own screen covers it — and *Always* location is missing | `{ kind: 'appSettings' }` — the fix is a permission, so the OS app-settings page is the destination |
| **Stay on track** (inactivity reminder) | Two calendar days after the SDK was last opened, at 10:00 — every open of the SDK re-arms it | `{ kind: 'screen', screen: 'insights' }`, or `'home'` when the insights module was disabled at the time the reminder was scheduled |
| **Battery low** (band battery warning) | At most once a day, when a battery reading before 18:00 shows the band at 20% or heading there before midnight — right away if it is already there, otherwise at 18:00 | `{ kind: 'screen', screen: 'home' }` |
| **Workout in progress** / **Location Tracking** (Android only — the ongoing workout notifications) | For the whole of a Bluetooth or GPS workout | `{ kind: 'screen', screen: 'resume' }` — the live workout, exactly as the user left it |

All of them resolve the same way, so your code never needs to tell them apart — pass whatever screen you receive straight to `openScreen()`, or handle the tap however suits your app best. The destination is our recommendation, not an obligation.

### Routing a tap

Do not present the SDK from the tap handler itself: the inactivity reminder fires days after the last open, so its tap usually cold-launches the app — before a user session exists. Keep the screen aside and route it once the user is signed in and the screen that owns your SDK calls is up. The pattern below is what the [React Native demo](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-react-native) does in `App.tsx`:

```tsx
import { useEffect, useState } from 'react';
import { Linking } from 'react-native';
import { Rolla, RollaNotificationTarget, RollaScreen } from '@rolla-health/react-native-sdk';

// A tapped Rolla screen waiting for a signed-in session to open it.
const [pendingScreen, setPendingScreen] = useState<RollaScreen | null>(null);

useEffect(() => {
  const route = (target: RollaNotificationTarget | null) => {
    if (target === null) return;               // a plain launch, or a notification of your own
    if (target.kind === 'appSettings') {
      Linking.openSettings();                  // the background-location permission warning
      return;
    }
    setPendingScreen(target.screen);           // your signed-in screen opens it with openScreen()
  };

  Rolla.getInitialNotificationTarget().then(route);   // the tap that launched the app, if any
  const sub = Rolla.addListener('onNotificationTap', route); // taps while the app is running
  return () => sub.remove();
}, []);

// In the screen that owns your session, once it is on screen:
useEffect(() => {
  if (pendingScreen === null || !session) return;
  setPendingScreen(null);
  Rolla.openScreen(configuration, pendingScreen).then((status) => {
    if (status !== 'opened') console.log('Not opened:', status);
  });
}, [pendingScreen, session]);
```

### iOS setup

The SDK never claims your app's `UNUserNotificationCenter` delegate — your app owns it and receives every tap through it, including the SDK's. The wrapper ships `RollaBridgeNotifications` for that delegate: `handle(response:)` identifies Rolla's notifications and delivers the destination to JavaScript (as `onNotificationTap`, or queued for `getInitialNotificationTarget()` when no listener is attached yet); `isRollaNotification(_:)` lets Rolla's notifications show as banners while your app is frontmost. Assign the delegate inside `application(_:didFinishLaunchingWithOptions:)` and not later — when a tap cold-launches your app, iOS delivers `didReceive` right after launch, and only to a delegate that is already in place:

```swift
// ios/YourApp/AppDelegate.swift
import UserNotifications
import RollaWrapper

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]? = nil) -> Bool {
    UNUserNotificationCenter.current().delegate = self
    // … the React Native factory setup your template already has …
    return true
  }
}

extension AppDelegate: UNUserNotificationCenterDelegate {

  // Foreground arrivals: while your app is frontmost iOS shows nothing unless you say so here.
  // Show Rolla's as banners — and keep deciding for your own notifications exactly as you do today.
  func userNotificationCenter(_ center: UNUserNotificationCenter,
                              willPresent notification: UNNotification,
                              withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    let isRolla = RollaBridgeNotifications.isRollaNotification(notification)
    completionHandler(isRolla ? [.banner, .list, .sound] : [])
  }

  // Taps: returns false for notifications that are not Rolla's — handle those yourself.
  func userNotificationCenter(_ center: UNUserNotificationCenter,
                              didReceive response: UNNotificationResponse,
                              withCompletionHandler completionHandler: @escaping () -> Void) {
    defer { completionHandler() }
    _ = RollaBridgeNotifications.handle(response: response)
  }
}
```

If a push library already owns the delegate, hand it the notification's user-info dictionary from JavaScript instead: `Rolla.notificationTarget(userInfo)`. The native details — payload shape and the notification table — are in [iOS notificationTarget](../ios/10-api-reference.md#notificationtarget).

### Android setup

Nothing to add. Every notification the SDK posts opens your launcher activity when tapped; the wrapper reads the payload off the delivering intent — in `onNewIntent` for the running `MainActivity`, or from the launch intent when the tap started the app — and delivers it to JavaScript. The React Native template's `singleTask` launch mode is the right one for a React Native host: a tap reaches the running `MainActivity` through `onNewIntent`, arrives as `onNotificationTap`, and your `openScreen()` presents the SDK on the tapped screen — see [Permissions & Entitlements → Launch mode](03-permissions.md#launch-mode). The native details are in [Android notificationTarget](../android/08-api-reference.md#notificationtarget).

### RollaNotificationTarget

Where a recognized tap should lead:

| Target | Meaning |
|--------|---------|
| `{ kind: 'appSettings' }` | Take the user to the OS app-settings page (`Linking.openSettings()`) — carried by the background-location warning, see the table above |
| `{ kind: 'screen', screen }` | Open the [`RollaScreen`](#rollascreen) it carries via `openScreen()` — the table above lists which notification leads where |

## Events

All 17 events are subscribed through `Rolla.addListener(event, listener)`; the payload type of each is `RollaEventMap[event]`. They fall in three groups:

- **Presentation & token events** — the SDK needs your app to react: dismissal, errors, token exchange.
- **[Host events](#host-events)** — the SDK tells your app what happened inside it: syncs, activities, band, profile. Purely observational.
- **`onNotificationTap`** — the user tapped one of the SDK's notifications while the app was running — see [Notification Taps](#notification-taps).

### Presentation & Token Events

| Event | Payload | Fires when |
|-------|---------|-----------|
| `onClose` | [`RollaCloseEvent`](#rollacloseevent) — `{ reason, detail? }` | The SDK UI was dismissed — see [RollaCloseReason](#rollaclosereason). Also resolves the pending `show()` |
| `onError` | [`RollaErrorEvent`](#rollaerrorevent) — `{ code, message, presentationFailed }` | An error occurred — see [Errors](#errors). `presentationFailed` is `true` when the error ended a pending `show()` (no `onClose` follows and that `show()` rejects), `false` for errors raised while the SDK UI is running |
| `onTokenRefreshed` | `{ token, refreshToken?, expiresIn? }` | The SDK refreshed tokens internally — store them for future use; refresh tokens are single-use |
| `onTokenExpired` | `{}` | The SDK could not refresh the token — obtain fresh tokens from the Rolla auth API and call `updateToken()` (see [Token Management](06-token-management.md)) |

### Host Events

Twelve events push SDK happenings to your app, so you never have to poll. Two rules apply to all of them:

- **Engine-scoped, engine-lifetime delivery.** Events are armed by any of `show()`, `openScreen()`, `warmUpEngine()`, or any headless call, and keep flowing after the SDK UI closes — an upload that completes moments after dismissal still reports. Delivery stops only at `destroyEngine()`. Nothing fires while the engine is cold, and nothing is delivered retroactively — they reach whichever listeners are attached at that moment.
- **Payload conventions.** Dates are ISO-8601 strings; sample timestamps inside `syncedData.samples` are epoch milliseconds; fields the SDK did not report are omitted rather than `null`.

#### Sync Events

| Event | Payload | Fires when |
|-------|---------|-----------|
| `onSyncHealthDataCompleted` | [`RollaSyncResult`](#rollasyncresult) | A headless [`syncHealthData()`](#synchealthdata) reaches a terminal outcome — with the same result the promise resolves with |
| `onUiSyncCompleted` | [`RollaSyncResult`](#rollasyncresult) | A sync completes inside the SDK UI (auto-sync on open, return from background, manual refresh) |

**`syncedData` on UI syncs.** On a successful band / Apple Health / Health Connect UI sync, `syncedData` carries the same per-stream summary as the headless result (samples never included). It is omitted when there is nothing attributable to report — failures, Garmin/Oura content-only refreshes, syncs that recorded nothing, or overlapping syncs — never wrong or double-reported data.

#### Activity Events

| Event | Payload | Fires when |
|-------|---------|-----------|
| `onActivityStarted` | `{ activityId, type?, startTime?, origin, catalogId? }` | A live tracking session starts — `origin` is `'fresh'` or `'crashRecovery'` |
| `onActivityCompleted` | [`RollaCompletedActivity`](#activity-payloads) | An activity reaches a lifecycle phase: `'finished'` (saved in-SDK), then `'uploaded'` or `'uploadFailed'` |
| `onActivityRemoved` | `{ activityId, reason }` | An activity's record is removed without a kept result — `reason` is `'canceled'` (crash-recovery discard) or `'deleted'` (user deleted it, backend-confirmed) |

**Lifecycle guarantees.** Every started activity terminates in a `'finished'` completion or a removal — possibly in a *different app session* if the app dies in between (crash recovery resolves on the next launch, re-firing `onActivityStarted` with origin `'crashRecovery'`). Two exceptions are cleaned up silently, without an event: a session abandoned mid-tracking for over a day, and an interrupted session neither resumed nor discarded before the user starts their next activity. Dedupe on `activityId`, and treat `(activityId, phase)` as the idempotency key for completions — `'uploaded'` / `'uploadFailed'` can re-fire across retries. Manually logged activities enter the lifecycle at `'finished'` (no started event); pause/resume inside a session fires nothing.

#### Band Events

| Event | Payload | Fires when |
|-------|---------|-----------|
| `onBandPaired` | [`RollaBandInfo`](#rollabandinfo) | The user pairs a band inside the SDK UI |
| `onBandUnpaired` | [`RollaBandInfo`](#rollabandinfo) | The user unpairs the band inside the SDK UI (backend-confirmed) |
| `onBandConnected` | [`RollaBandInfo`](#rollabandinfo) | The paired band establishes a live BLE link |
| `onBandDisconnected` | [`RollaBandInfo`](#rollabandinfo) | The paired band loses its live BLE link (debounced a few seconds) |

**Link events are not a proximity signal.** `onBandConnected` / `onBandDisconnected` report genuine BLE link transitions of the user's own band only: connect fires immediately, disconnect only after the BLE supervision timeout plus a ~3-second debounce (a drop with an immediate reconnect reports nothing). They are orthogonal to paired/unpaired — an unpair or logout drops the physical link too, so a disconnect legitimately accompanies those. Use [`getPairedBandInfo()`](#getpairedbandinfo) for the pairing state.

#### Profile & Settings Events

| Event | Payload | Fires when |
|-------|---------|-----------|
| `onPrimarySourceChanged` | `{ previousSource, currentSource }` — [`RollaSyncSource`](#rollasyncresult) values | The user's primary data source changes |
| `onGoalsChanged` | `{ changedGoals, enabledGoals }` — arrays of `{ id, name, enabled }` | The user saves goal changes inside the SDK UI (backend-confirmed) — one event per save |
| `onProfileUpdated` | `{ changedFields }` — only the changed fields, keyed by the SDK's field names | The user updates profile data inside the SDK UI |

## Headless Methods

Four methods run **headlessly** — no SDK UI needs to be opened. Each starts the engine automatically on first use. Because there is no UI to prompt from, **your app owns OS permissions**: when one is missing, the methods resolve with a typed reason instead of prompting. They reject only on a transport failure — the engine not starting, or an SDK `RollaError` (the rejection `code` is the native error code).

### warmUpEngine

```ts
Rolla.warmUpEngine(config: RollaConfiguration): Promise<void>
```

Starts and configures the engine ahead of time so the headless calls — and the first `show()` — have zero start-up latency. Optional: the methods below warm the engine themselves if needed; this only moves the one-time cost to a moment you control (a common pattern is right after login). Safe to call repeatedly — a repeat call for the same user is a no-op that preserves the session. See [Engine Lifecycle → Warming Up the Engine](07-engine-lifecycle.md#warming-up-the-engine).

### syncHealthData

```ts
Rolla.syncHealthData(config: RollaConfiguration, options?: { includeSamples?: boolean }): Promise<RollaSyncResult>
```

Runs a full sync of the user's primary data source (band over BLE, Apple Health, or Health Connect) and resolves with a typed [`RollaSyncResult`](#rollasyncresult) — the same result is also delivered to `onSyncHealthDataCompleted` listeners:

| Field | Meaning |
|-------|---------|
| `outcome` | `'success'`, `'partial'`, `'skipped'` (expectedly did nothing — see `skipReason`), or `'failure'` (see `error`) |
| `hasNewData` | Whether anything new was uploaded (success only) |
| `source` | `'band'`, `'appleHealth'`, `'healthConnect'`, `'garmin'`, `'oura'` |
| `startedAt` / `lastSyncAt` | When the sync started / completed on the device — together they give the sync duration. `startedAt` is omitted for `'skipped'` (nothing ran) and on overlapping syncs; `lastSyncAt` is present only on success |
| `skipReason` | `'noBandPaired'` (no band on the account), `'bandNotConnected'` (a band is paired but couldn't be reached right now), `'alreadyInProgress'`, `'serverSideSource'` (Garmin/Oura sync server-side), `'bluetoothPermissionRequired'`, `'bluetoothUnavailable'`, `'appleHealthPermissionRequired'`, `'healthConnectPermissionRequired'`, `'notInitialized'`, `'offline'` |
| `syncedData` | Per-stream summary of what was uploaded; pass `{ includeSamples: true }` to also receive raw sample arrays |

```ts
const result = await Rolla.syncHealthData(configuration);
switch (result.outcome) {
  case 'success':
  case 'partial':
    console.log('Synced', result.source, result.syncedData?.syncedDates);
    break;
  case 'skipped':
    console.log('Skipped because', result.skipReason);
    break;
  case 'failure':
    console.warn(result.error);
    break;
}
```

### getBandBatteryLevel

```ts
Rolla.getBandBatteryLevel(config: RollaConfiguration): Promise<RollaBatteryResult>
```

A **live BLE read** from the paired Rolla band — the band must be reachable. Resolves with `{ status, level? }`: a percentage in `level` when `status` is `'available'`, otherwise a documented reason (`'noBandPaired'`, `'bandNotConnected'` — a band is paired but couldn't be reached, `'notRollaDevice'` — reserved for forward compatibility, not currently returned, `'bluetoothUnavailable'`, `'bluetoothPermissionRequired'`, `'unknownError'`). Never a stale value reported as live.

### getPairedBandInfo

```ts
Rolla.getPairedBandInfo(config: RollaConfiguration): Promise<RollaPairedBandResult>
```

Answers "does this account currently have a Rolla band?" with **zero Bluetooth** — no scan, no connect, no BLE permission; works with Bluetooth off. Resolves with `{ status, band? }`:

| Status | Meaning |
|--------|---------|
| `'bandPaired'` | A band is paired — `band` carries its MAC address (always present) plus the last cached battery/firmware/serial, each possibly omitted if the SDK hasn't read the band recently |
| `'noBandPaired'` | The user's profile confirms no band is paired |
| `'unknown'` | Could not be determined (offline with no local record) — reported instead of guessing |

The lookup is network-first: the profile is the authoritative pairing record, so a band unpaired remotely from another device is reported correctly. This is a pairing-state query, not a link-state one — live connect/disconnect transitions arrive via `onBandConnected` / `onBandDisconnected`.

```ts
const paired = await Rolla.getPairedBandInfo(configuration);
if (paired.status === 'bandPaired') {
  console.log('Band:', paired.band?.macAddress);
}
```

## Types

Every type below is exported from the package. The unions mirror the native SDK enums by their raw values. `RollaConfiguration`, `RollaBranding` and their option unions are on the [Configuration](05-configuration.md) page.

### RollaCloseEvent

```ts
interface RollaCloseEvent {
  reason: RollaCloseReasonKind;  // see RollaCloseReason below
  detail?: string;               // present when reason is 'flutterRequested'
}
```

### RollaErrorEvent

```ts
interface RollaErrorEvent {
  code: string;                  // a native RollaError code, e.g. 'ENGINE_FAILED', 'INIT_FAILED', 'FLUTTER_ERROR'
  message: string;
  presentationFailed: boolean;   // true when the error ended a pending show()
}
```

### RollaSyncResult

```ts
type RollaSyncOutcome = 'success' | 'partial' | 'skipped' | 'failure' | 'unknown';
type RollaSyncSource = 'band' | 'appleHealth' | 'healthConnect' | 'garmin' | 'oura' | 'unknown';

interface RollaSyncResult {
  outcome: RollaSyncOutcome;
  hasNewData: boolean;
  source: RollaSyncSource;
  startedAt?: string;            // ISO-8601
  lastSyncAt?: string;           // ISO-8601
  skipReason?: RollaSyncSkipReason;
  error?: string;
  syncedData?: RollaSyncedHealthData;
}

interface RollaSyncedHealthData {
  source: RollaSyncSource;
  syncedDates: string[];         // 'YYYY-MM-DD'
  batteryLevel?: number;
  heartRate?: RollaSyncedStreamSummary;   // { count, from?, to?, total?, blocks?, minutes? }
  hrv?: RollaSyncedStreamSummary;
  steps?: RollaSyncedStreamSummary;
  sleep?: RollaSyncedStreamSummary;
  weight?: RollaSyncedStreamSummary;
  bloodPressure?: RollaSyncedStreamSummary;
  workouts?: RollaSyncedStreamSummary;
  samples?: RollaSyncedSamples;  // only with includeSamples; timestamps are epoch milliseconds
}
```

### Activity payloads

```ts
interface RollaStartedActivity {
  activityId: string;
  type?: string;
  startTime?: string;            // ISO-8601
  origin: 'fresh' | 'crashRecovery' | 'unknown';
  catalogId?: string;
}

interface RollaCompletedActivity {
  activityId: string;
  phase: 'finished' | 'uploaded' | 'uploadFailed' | 'unknown';
  source: 'rolla' | 'manual' | 'unknown';
  catalogId?: string;
  type?: string;
  environment?: string;
  category?: string;
  totalDurationS?: number;
  totalDistanceM?: number;
  totalCalories?: number;
  startTime?: string;            // ISO-8601
  endTime?: string;              // ISO-8601
}

interface RollaRemovedActivity {
  activityId: string;
  reason: 'canceled' | 'deleted' | 'unknown';
}
```

### RollaBandInfo

```ts
interface RollaBandInfo {
  macAddress: string;
  name?: string;
  rssi?: number;
  deviceType?: string;
  batteryPercent?: number;
  firmwareVersion?: string;
  serialNumber?: string;
}
```

### RollaSubscription

```ts
interface RollaSubscription {
  remove(): void;
}
```

## Errors

Method promises reject with an `Error` carrying a `code`; `onError` carries the same code for errors raised by the native SDK. The wrapper's own codes come first, then the native `RollaError` codes pass through verbatim:

| `code` | Raised by | Meaning | Host App Recovery |
|--------|-----------|---------|-------------------|
| `INVALID_CONFIG` | `show`, `openScreen`, headless calls | A configuration value the SDK does not know — a misspelled module, data source, language or transition, an unparsable color, a missing `token` or `partnerId` | Fix the value; TypeScript catches most of these at compile time |
| `ALREADY_PRESENTING` | `show` | `show()` called while another `show()` is pending or the SDK UI is on screen | Check `isPresenting()` before calling; `dismiss()` first if needed |
| `NO_PRESENTER` (iOS) / `NO_ACTIVITY` (Android) | `show`, `openScreen` | No view controller or foreground activity to present from | Call from a mounted, settled screen — see [Code Integration](04-code-integration.md#present-the-sdk) |
| `NO_ACTIVE_SESSION` | `updateToken`, `clearSession` | The engine is cold, so there is no session to update or clear | `updateToken`: pass the newest pair in your next configuration. `clearSession`: pass the configuration so the wrapper warms the engine first |
| `UPDATE_TOKEN_FAILED` / `CLEAR_SESSION_FAILED` | `updateToken`, `clearSession` | The native SDK reported a failure | Inspect `message`; retry |
| `SHOW_FAILED` | `show` | The native SDK failed to launch without a more specific code | Inspect `message`; retry, then `destroyEngine()` and re-initialize |
| `ENGINE_FAILED` | native | Flutter engine failed to start | Retry after a delay. If persistent, `destroyEngine()` and re-initialize. Check device memory |
| `INIT_FAILED` | native | SDK init failed — `message` explains why | Common causes: invalid credentials, network failure, expired token. Verify the configuration and retry |
| `FLUTTER_ERROR` | native | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with the code |
| `UNKNOWN` | native | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent |

The native codes and their meaning are the same as on the native platforms — see [iOS RollaError](../ios/10-api-reference.md#rollaerror) and [Android RollaError](../android/08-api-reference.md#rollaerror).

## RollaCloseReason

```ts
type RollaCloseReasonKind =
  | 'flutterRequested'
  | 'hostNavigationBack'
  | 'hostModalDismiss'
  | 'programmatic'
  | 'hostStackReplaced'
  | 'unknown';
```

When each reason is triggered:

| Close Reason | When Triggered |
|-------------|----------------|
| `'flutterRequested'` | The SDK's internal UI initiated the close (e.g. the user tapped close/done). The optional `detail` may provide context |
| `'hostNavigationBack'` | The user pressed the back gesture or navigation back (Android) |
| `'hostModalDismiss'` | The user dismissed the SDK UI via the swipe-down gesture, or the view controller it was presented on was dismissed (iOS) |
| `'programmatic'` | Your app called `dismiss()` |
| `'hostStackReplaced'` | Your app replaced the navigation stack while the SDK was presenting |
| `'unknown'` | The close reason could not be determined |

---

**Previous:** [Engine Lifecycle](07-engine-lifecycle.md) | **Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
