# Permissions & Entitlements

This section covers configuring permissions in `Info.plist` and entitlements in your `.entitlements` file. For the partner-facing justification of each permission for your privacy policy and **App Store Connect → App Privacy** form — see the [Permissions Rationale](#permissions-rationale) matrix at the bottom of this page.

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

## Permissions Rationale

This section is the partner-facing justification for every permission the SDK requests on iOS. The wording is intended to be lifted into a privacy policy or pasted into the **App Store Connect → App Privacy** form. Permissions are grouped by the user-visible capability they gate.

### Location

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `NSLocationWhenInUseUsageDescription` | Required for any GPS-tracked activity | During an outdoor activity the SDK records the user's GPS trace via `CLLocationManager` to draw the route polyline, compute distance, pace, and elevation. *While Using the App* authorization is enough to record a workout that stays in the foreground (e.g. on a phone-mounted handlebar with the screen on). |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Optional, strongly recommended | Outdoor workouts are routinely longer than the screen-on timeout. *Always* authorization combined with `UIBackgroundModes: location` lets `CLLocationManager` keep delivering fixes when the app is backgrounded or the phone is locked. Without it, **the polyline drops out as soon as the user locks the phone** — runs longer than the screen-on timeout will record incomplete routes. The SDK explicitly sets `allowsBackgroundLocationUpdates = true` (`CoreLocationManager.swift:70`) and `pausesLocationUpdatesAutomatically = false` so iOS does not opportunistically pause the stream during a workout. |

> **Note on `NSLocationAlwaysUsageDescription`:** This is a legacy iOS-10-and-below key. iOS 11+ uses `NSLocationAlwaysAndWhenInUseUsageDescription` exclusively. Declaring the legacy key alongside it is harmless and is what the white-label app does for safety; new integrations targeting iOS 12+ can omit it.

### Bluetooth

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `NSBluetoothAlwaysUsageDescription` | Required for any band-paired feature | The SDK initializes a `CBCentralManager` (`RollaPermissionsHandler.swift:144-147`) to scan for and connect to Rolla fitness bands. iOS 13+ shows the system Bluetooth prompt the moment `CBCentralManager` is allocated. |
| `NSBluetoothPeripheralUsageDescription` | Optional | Legacy iOS-12-and-below companion key. The SDK does not act as a peripheral (no `CBPeripheralManager` allocations), but Apple's static analyzer will warn if only one of the two Bluetooth keys is present, so we recommend declaring it for compatibility with older deployment targets and to silence the warning. |
| Bluetooth Central Entitlement | Required (build-time) | App Store capability required for the SDK's BLE central role (band scanning and connection). Configured in `.entitlements`, not `Info.plist`. |

### Apple Health

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `NSHealthShareUsageDescription` | Optional, strongly recommended | The SDK reads the user's HealthKit data — heart rate, steps, distance, calories, sleep, workouts — via `HKHealthStore.requestAuthorization(toShare: [], read: readTypes)` (`AppleHealthManager.swift:43`) so the Rolla feed reflects every workout the user does, including ones recorded by Apple Watch, Strava, Nike Run Club, etc. Without it, the user only sees workouts recorded inside Rolla. |
| HealthKit Entitlement | Required if Apple Health is enabled (build-time) | App Store capability that authorizes the HealthKit APIs the SDK uses to read Apple Health data. Configured in `.entitlements`, not `Info.plist`. |

> **Note on `NSHealthUpdateUsageDescription`:** The SDK passes an empty `toShare` array — it does **not** write back to HealthKit. The white-label app declares the write key out of caution, but a partner integration whose product does not write to HealthKit can omit it. If you call any HealthKit write API in your own host code, declare it.

### Background Modes

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `UIBackgroundModes: location` | Required for full-route recording with *Always* location | Pairs with `NSLocationAlwaysAndWhenInUseUsageDescription` to let `CLLocationManager` deliver fixes while the app is backgrounded. |
| `UIBackgroundModes: bluetooth-central` | Required for sustained band connection | Allows the SDK's `CBCentralManager` to maintain its GATT connection to the band when the user locks the phone or switches apps mid-workout. Without this background mode, iOS suspends the connection within ~10 seconds of the app going to the background and the band shows as "disconnected" on the workout screen. |
| `UIBackgroundModes: remote-notification` | Optional | Currently declared by the white-label app for silent-push delivery of post-workout insights from the Rolla backend. A partner integration that does not subscribe to those silent pushes can omit it. |

### Live Activities

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `NSSupportsLiveActivities` | Optional | During an active workout the SDK posts a Live Activity (`LiveWorkoutBridge.swift:180` — `Activity.request(attributes:content:pushType: nil)`) that displays elapsed time, current pace, and heart rate on the lock screen and in the Dynamic Island. |
| `NSSupportsLiveActivitiesFrequentUpdates` | Optional, recommended alongside the above | Lets the SDK push updates more often than the default ~1/15s budget, which is what makes the on-screen pace number flow rather than tick. |

---

**Previous:** [CocoaPods Setup](02-cocoapods-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
