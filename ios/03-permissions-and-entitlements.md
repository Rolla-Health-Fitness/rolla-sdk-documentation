# Permissions & Entitlements

This section covers configuring permissions in `Info.plist` and entitlements in your `.entitlements` file.

## Configure Info.plist

Add the following permissions to your `Info.plist` file:

### Background Modes

```xml
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
    <string>bluetooth-central</string>
</array>
```

### Bluetooth Permissions

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth access allows the app to connect to your fitness band and keep syncing health data and activity metrics reliably.</string>

<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app uses Bluetooth to connect to your fitness band and sync health data and activity metrics.</string>
```

### Location Permissions

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app uses your location to track outdoor activities like running and cycling. With 'While Using the App', your route is recorded only when the app is open on screen.</string>

<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app uses your location to accurately track outdoor activities like running and cycling even when your phone is locked or the app is in the background.</string>
```

### Mapbox Token

The SDK uses Mapbox for map rendering (e.g., activity route maps). Add the Mapbox access token to your `Info.plist`:

```xml
<key>MBXAccessToken</key>
<string>your-mapbox-access-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

### Apple Health Permissions

```xml
<key>NSHealthShareUsageDescription</key>
<string>The app reads your Apple Health data — such as heart rate, steps, sleep, and workouts — to provide personalised health insights and keep your metrics in sync.</string>
```

> **Note:** The SDK reads Apple Health data only — it does not write to HealthKit. Only `NSHealthShareUsageDescription` is required.

## Configure Entitlements

### Bluetooth Central

Add the Bluetooth Central entitlement to your `.entitlements` file:

```xml
<key>com.apple.developer.bluetooth-central</key>
<true/>
```

### HealthKit (Required for Apple Health)

Add the HealthKit capability in Xcode:

1. Select your target in Xcode
2. Go to Signing & Capabilities tab
3. Click **+ Capability**
4. Add **HealthKit**

This adds to the entitlements file:

```xml
<key>com.apple.developer.healthkit</key>
<true/>
<key>com.apple.developer.healthkit.access</key>
<array/>
```

> **Note:** The HealthKit entitlement requires the app to be registered on an Apple Developer account.

## Permission Summary

| Permission | Rationale |
|------------|-----------|
| Bluetooth Always (`NSBluetoothAlwaysUsageDescription`) | Scans for, pairs with, and reads live metrics from the user's Rolla fitness band via `CBCentralManager`. |
| Bluetooth Peripheral (`NSBluetoothPeripheralUsageDescription`) | Legacy iOS-12-and-below companion key; declared for backwards compatibility, the SDK does not actually act as a peripheral. |
| Location When In Use (`NSLocationWhenInUseUsageDescription`) | Records the GPS polyline, distance, and pace for outdoor activities while the SDK is in the foreground. |
| Location Always (`NSLocationAlwaysAndWhenInUseUsageDescription`) | Continues recording the GPS polyline when the user locks the phone or backgrounds the app mid-workout. |
| HealthKit Read (`NSHealthShareUsageDescription`) | Reads workouts, heart rate, sleep, distance, and calories from Apple Health so the Rolla feed reflects every workout the user does. |
| HealthKit Entitlement | App Store capability that authorizes the HealthKit APIs the SDK uses to read Apple Health data. |
| Bluetooth Central Entitlement | App Store capability required for the SDK's BLE central role (band scanning and connection). |
| Background Mode: `location` | Pairs with *Always* location authorization so `CLLocationManager` keeps delivering fixes when the app is backgrounded. |
| Background Mode: `bluetooth-central` | Keeps the BLE connection to the band alive when the user locks the phone or switches apps mid-workout. |
| Live Activities (`NSSupportsLiveActivities`) | Displays elapsed time, pace, and heart rate for an active workout on the lock screen and in the Dynamic Island. |
| Photo Library Add (`NSPhotoLibraryAddUsageDescription`) | Saves a shareable summary card from a completed workout to the user's Photos library when they tap "Save to Photos". |

> **Note:** The host app must declare all permissions. The SDK triggers system prompts at the appropriate time but does not declare permissions itself.

---

## Permissions Rationale

This section is the partner-facing justification for every permission the SDK requests on iOS. The wording is intended to be lifted into a privacy policy or pasted into the **App Store Connect → App Privacy** form. Permissions are grouped by the user-visible capability they gate.

### Location

#### `NSLocationWhenInUseUsageDescription`

- **Required or optional:** Required for any GPS-tracked activity (running, cycling, walking).
- **Rationale:** During an outdoor activity the SDK records the user's GPS trace via `CLLocationManager` to draw the route polyline, compute distance, pace, and elevation. *While Using the App* authorization is enough to record a workout that stays in the foreground (e.g. on a phone-mounted handlebar with the screen on).

#### `NSLocationAlwaysAndWhenInUseUsageDescription`

- **Required or optional:** Optional but strongly recommended. Without it, **the polyline drops out as soon as the user locks the phone** — runs longer than the screen-on timeout will record incomplete routes.
- **Rationale:** Outdoor workouts are routinely longer than the screen-on timeout. *Always* authorization combined with `UIBackgroundModes: location` lets `CLLocationManager` keep delivering fixes when the app is backgrounded or the phone is locked. The SDK explicitly sets `allowsBackgroundLocationUpdates = true` (`CoreLocationManager.swift:70`) and `pausesLocationUpdatesAutomatically = false` (line 67) so iOS does not opportunistically pause the stream during a workout.

> **Note on `NSLocationAlwaysUsageDescription`:** This is a legacy iOS-10-and-below key. iOS 11+ uses `NSLocationAlwaysAndWhenInUseUsageDescription` exclusively. Declaring the legacy key alongside it is harmless and is what the white-label app does for safety; new integrations targeting iOS 12+ can omit it.

### Bluetooth

#### `NSBluetoothAlwaysUsageDescription`

- **Required or optional:** Required for any band-paired feature.
- **Rationale:** The SDK initializes a `CBCentralManager` (`RollaPermissionsHandler.swift:144-147`) to scan for and connect to Rolla fitness bands. iOS 13+ shows the system Bluetooth prompt the moment `CBCentralManager` is allocated.

#### `NSBluetoothPeripheralUsageDescription`

- **Required or optional:** Optional. Legacy iOS-12-and-below key. The SDK does not act as a peripheral (no `CBPeripheralManager` allocations), but Apple's static analyzer will warn if only one of the two Bluetooth keys is present, so we recommend declaring it for compatibility with older deployment targets and to silence the warning.
- **Rationale:** Historical / lint compatibility only. Not user-visible.

### Apple Health

#### `NSHealthShareUsageDescription` (read)

- **Required or optional:** Optional but strongly recommended.
- **Rationale:** The SDK reads the user's HealthKit data — heart rate, steps, distance, calories, sleep, workouts — via `HKHealthStore.requestAuthorization(toShare: [], read: readTypes)` (`AppleHealthManager.swift:43`) so the Rolla feed reflects every workout the user does, including ones recorded by Apple Watch, Strava, Nike Run Club, etc. Without it, the user only sees workouts recorded inside Rolla.

> **Note on `NSHealthUpdateUsageDescription`:** The SDK passes an empty `toShare` array — it does **not** write back to HealthKit. The white-label app declares the write key out of caution, but a partner integration whose product does not write to HealthKit can omit it. If you call any HealthKit write API in your own host code, declare it.

### Notifications

The SDK does not currently call `UNUserNotificationCenter.requestAuthorization(...)` itself for push or local notifications used by core SDK flows; engagement reminders are scheduled via the local notification plugin and inherit the host app's notification authorization. If the host app uses notifications for its own features, declare the prompt timing in your privacy policy under the host app's notifications usage rather than tying it to the SDK.

### Background Modes

#### `UIBackgroundModes: location`

- **Required or optional:** Required if the host app uses *Always* location for full-route recording.
- **Rationale:** Pairs with `NSLocationAlwaysAndWhenInUseUsageDescription` to let `CLLocationManager` deliver fixes while the app is backgrounded.

#### `UIBackgroundModes: bluetooth-central`

- **Required or optional:** Required for the band to remain connected when the user locks the phone or switches apps mid-workout.
- **Rationale:** Allows the SDK's `CBCentralManager` to maintain its GATT connection to the band in the background. Without this background mode, iOS suspends the connection within ~10 seconds of the app going to the background and the band shows as "disconnected" on the workout screen.

#### `UIBackgroundModes: remote-notification`

- **Required or optional:** Optional. Currently declared by the white-label app for silent-push delivery of post-workout insights from the Rolla backend; a partner integration that does not subscribe to those silent pushes can omit it.
- **Rationale:** Allows the host app to be woken in the background to process silent APNs payloads that fan out workout summaries.

### Live Activities

#### `NSSupportsLiveActivities` and `NSSupportsLiveActivitiesFrequentUpdates`

- **Required or optional:** Optional. Required if the host app wants the live-workout lock-screen / Dynamic Island view.
- **Rationale:** During an active workout the SDK posts a Live Activity (`LiveWorkoutBridge.swift:180` — `Activity.request(attributes:content:pushType: nil)`) that displays elapsed time, current pace, and heart rate on the lock screen and in the Dynamic Island. `NSSupportsLiveActivitiesFrequentUpdates` lets the SDK push updates more often than the default ~1/15s budget, which is what makes the on-screen pace number flow rather than tick.

### Photo Library

#### `NSPhotoLibraryAddUsageDescription`

- **Required or optional:** Optional. Required only if the host app uses the SDK's "save activity image" share action.
- **Rationale:** The SDK can render a shareable summary card from a completed workout and offers a "Save to Photos" action. iOS requires this key the moment the SDK calls `PHPhotoLibrary` to write an image.

> **Note on `NSPhotoLibraryUsageDescription` (read):** The SDK does **not** read from the photo library. Do not declare this key unless your host app uses photo-picker UI for its own features — declaring it surfaces an extra App Store privacy line that you can't justify from SDK behavior alone.

---

**Previous:** [CocoaPods Setup](02-cocoapods-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
