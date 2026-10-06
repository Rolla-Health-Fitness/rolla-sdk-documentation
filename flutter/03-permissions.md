# Permissions & Entitlements

The Rolla SDK uses Bluetooth Low Energy, Core Location, Core Motion, HealthKit / Health Connect, Photos, and Mapbox. The package does not change which permissions are required — they are declared in your `ios/` and `android/` projects exactly as in a native app. This page lists what a fresh `flutter create` scaffold is missing and links to the native pages for the rationale and the exact wording.

## iOS

### Info.plist

iOS calls `abort()` (SIGABRT) the moment the SDK touches Bluetooth, Location, Motion, HealthKit or Photos without a corresponding usage-description string in `Info.plist` — there is no Dart exception to catch. A fresh Flutter scaffold has none of the keys below.

Also required: `MBXAccessToken` (your Mapbox public token) and a `UIBackgroundModes` array with `location` and `bluetooth-central`, so outdoor tracking continues and the band stays connected when the app is backgrounded.

Add the following inside the top-level `<dict>` of `ios/Runner/Info.plist`. Customize the strings to match your app's wording — they are shown to the user in the OS permission dialogs and referenced in your App Store Connect privacy answers:

```xml
<key>MBXAccessToken</key>
<string>YOUR_MAPBOX_PUBLIC_TOKEN</string>

<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth access lets the app connect to your fitness band and keep syncing data, even when the app isn't open.</string>

<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app uses Bluetooth to connect to your fitness band and sync health data.</string>

<key>NSLocationWhenInUseUsageDescription</key>
<string>The app uses your location to track outdoor activities like running and cycling. With 'While Using the App', your route is recorded only when the app is open.</string>

<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app uses your location to accurately track outdoor activities like running and cycling, even when your phone is locked or the app is in the background.</string>

<key>NSMotionUsageDescription</key>
<string>Used to count steps and measure cadence during workouts when no fitness band is connected.</string>

<key>NSHealthShareUsageDescription</key>
<string>The app reads your Apple Health data, such as heart rate, steps, sleep and workouts, to provide personalised health insights and keep your metrics in sync.</string>

<key>NSPhotoLibraryAddUsageDescription</key>
<string>Save your activity images to Photos.</string>

<key>UIBackgroundModes</key>
<array>
  <string>location</string>
  <string>bluetooth-central</string>
</array>
```

| Key | Required for | Missing it means |
|-----|--------------|------------------|
| `NSBluetoothAlwaysUsageDescription`, `NSBluetoothPeripheralUsageDescription` | Band pairing and sync | Crash at `CBCentralManager` init |
| `NSLocationWhenInUseUsageDescription`, `NSLocationAlwaysAndWhenInUseUsageDescription` | Outdoor and background activity tracking | Crash at `CLLocationManager` request |
| `NSMotionUsageDescription` | Smartphone-only workouts (pedometer) | Crash at `CMPedometer` start |
| `NSHealthShareUsageDescription` | Apple Health read — the SDK never writes to HealthKit, so `NSHealthUpdateUsageDescription` is not needed | Crash at HealthKit authorization |
| `NSPhotoLibraryAddUsageDescription` | Saving activity share images | Crash at the photo save |
| `MBXAccessToken` | Route maps | Blank maps |
| `UIBackgroundModes` (`location`, `bluetooth-central`) | Band connection and GPS while backgrounded | Tracking stops when the app leaves the foreground |

You will receive the Mapbox token from Rolla along with your partner credentials. Use a **public** token (`pk.` prefix) — secret tokens are not valid client-side. Add `remote-notification` to `UIBackgroundModes` only if your app forwards push notifications.

For the rationale behind every key, in wording you can lift into your privacy policy and the App Store Connect privacy form, see [iOS Permissions & Entitlements → Permissions Rationale](../ios/03-permissions-and-entitlements.md#permissions-rationale).

### Entitlements

Two capabilities live in `ios/Runner/Runner.entitlements`, not in `Info.plist` — a fresh Flutter scaffold has neither:

- **HealthKit** — required for Apple Health. Add the capability in Xcode (Runner target → Signing & Capabilities → + Capability → HealthKit); it writes `com.apple.developer.healthkit` and the empty `com.apple.developer.healthkit.access` array. The App ID must have HealthKit enabled on your Apple Developer account.
- **Bluetooth Central** — `com.apple.developer.bluetooth-central`, the capability for the SDK's Bluetooth central role.

Exact keys and steps: [iOS Permissions & Entitlements → Configure Entitlements](../ios/03-permissions-and-entitlements.md#configure-entitlements). Apple Health needs no code on your side — the SDK reads the 14 HealthKit types listed in [iOS Apple Health Integration](../ios/06-apple-health.md) and prompts the user from its own UI.

### Notification Delegate

The SDK posts its notifications through `flutter_local_notifications`, which needs your `AppDelegate` to be the notification-center delegate so taps reach the SDK and its notifications show while the app is in the foreground. Add one line to `ios/Runner/AppDelegate.swift`, after the plugin registration:

```swift
GeneratedPluginRegistrant.register(with: self)
UNUserNotificationCenter.current().delegate = self as UNUserNotificationCenterDelegate
```

If a push library already owns the delegate, keep it — the plugin forwards through the delegate the app already has. Where a tap leads: [API Reference → Notification Taps](07-api-reference.md#notification-taps).

### Live Activities (Optional, iOS 16.1+)

The SDK drives a Lock Screen / Dynamic Island Live Activity during workouts. Everything it needs is native and lives in your `ios/` project exactly as in a Swift app: a Widget Extension target named `liveworkout`, three Swift files (the shared `LiveWorkoutAttributes` data contract compiled into both targets, the widget bundle, and the SwiftUI UI you own and can restyle), the Push Notifications capability on both targets, and two keys in the main app's `Info.plist`:

```xml
<key>NSSupportsLiveActivities</key>
<true/>
<key>NSSupportsLiveActivitiesFrequentUpdates</key>
<true/>
```

No Dart is involved. Follow the step-by-step procedure and copy the complete Swift sources from [iOS Live Activities](../ios/09-live-activities.md). Apps whose deployment target is below 16.1 still build — the SDK skips Live Activities on older devices.

## Android

### Mapbox Token

Route maps need the Mapbox public token in `android/app/src/main/res/values/strings.xml` — the Android counterpart of `MBXAccessToken`:

```xml
<string name="mapbox_access_token">YOUR_MAPBOX_PUBLIC_TOKEN</string>
```

Without it the SDK runs, but every map stays blank — see [Android Permissions → Mapbox Token](../android/03-permissions.md#mapbox-token).

### AndroidManifest.xml

The package's own Android manifest declares the Bluetooth (including `BLUETOOTH_ADVERTISE`), location, activity-recognition, foreground-service, notification and boot permissions the SDK needs, and the manifest merger pulls them into your app — you do not declare those. Three things are still yours to declare, because Google reviews the **merged** manifest under your app's identity:

**Internet.** Flutter's debug manifest adds `INTERNET` for hot reload; release builds need it declared explicitly.

**Health Connect.** The package's manifest declares only part of the Health Connect read set. Declare the full set, the permissions-rationale intent-filter on your `MainActivity`, the `ViewPermissionUsageActivity` alias for Android 14, and the `<queries>` block that lets the SDK detect the Health Connect app:

```xml
<manifest …>
    <uses-permission android:name="android.permission.INTERNET" />

    <!-- Health Connect permissions -->
    <uses-permission android:name="android.permission.health.READ_HEART_RATE" />
    <uses-permission android:name="android.permission.health.READ_HEART_RATE_VARIABILITY" />
    <uses-permission android:name="android.permission.health.READ_STEPS" />
    <uses-permission android:name="android.permission.health.READ_ACTIVE_CALORIES_BURNED" />
    <uses-permission android:name="android.permission.health.READ_SLEEP" />
    <uses-permission android:name="android.permission.health.READ_WEIGHT" />
    <uses-permission android:name="android.permission.health.READ_BLOOD_PRESSURE" />
    <uses-permission android:name="android.permission.health.READ_EXERCISE" />
    <uses-permission android:name="android.permission.health.READ_EXERCISE_ROUTES" />
    <uses-permission android:name="android.permission.health.READ_SPEED" />
    <uses-permission android:name="android.permission.health.READ_DISTANCE" />
    <uses-permission android:name="android.permission.health.READ_TOTAL_CALORIES_BURNED" />
    <uses-permission android:name="android.permission.health.READ_HEALTH_DATA_HISTORY" />

    <application …>
        <activity android:name=".MainActivity" …>
            <!-- your existing intent-filters (LAUNCHER, deep links, etc.) -->

            <!-- Health Connect "View permissions" rationale entry point; the SDK handles the intent. -->
            <intent-filter>
                <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
            </intent-filter>
        </activity>

        <!-- Android 14 Health Connect compliance -->
        <activity-alias
            android:name=".ViewPermissionUsageActivity"
            android:exported="true"
            android:permission="android.permission.START_VIEW_PERMISSION_USAGE"
            android:targetActivity=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.VIEW_PERMISSION_USAGE" />
                <category android:name="android.intent.category.HEALTH_PERMISSIONS" />
            </intent-filter>
        </activity-alias>
    </application>

    <!-- Package visibility: lets the SDK detect the Health Connect app on Android 11+. -->
    <queries>
        <package android:name="com.google.android.apps.healthdata" />
        <intent>
            <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
        </intent>
    </queries>
</manifest>
```

Every entry is explained in [Android Permissions → Health Connect](../android/03-permissions.md#health-connect-android), including the privacy-policy and Data Safety updates a Health Connect build needs. The Flutter demo's `android/app/src/main/AndroidManifest.xml` is the same set applied to a Flutter scaffold.

**Optional permissions the SDK leaves to you.** `SCHEDULE_EXACT_ALARM` for on-time reminders and `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` for the OEM battery-manager exemption — see the [Permissions Rationale](../android/03-permissions.md#permissions-rationale) and [OEM Battery Optimization](../android/09-troubleshooting.md#oem-battery-optimization).

**Your Play listing.** Declare the permissions you ship in the Data Safety form and your privacy policy; the rationale matrix on the Android permissions page is written to be lifted into both. Listing the SDK-merged permissions explicitly in your own manifest, as the demo does, keeps that declaration reviewable in one place.

### Launch Mode

Keep Flutter's default `android:launchMode="singleTop"` on `MainActivity`. The SDK runs inside your activity, so a tap on one of its notifications re-enters the same activity and the SDK routes it — see [API Reference → Notification Taps](07-api-reference.md#notification-taps).

### Notification Channels

The SDK creates its notification channels itself, with brand-neutral names that read naturally under your app's name in system settings — nothing to declare. See [Android Permissions → Notification Channels](../android/03-permissions.md#notification-channels).

## Runtime Prompts

Runtime permission prompts — the Bluetooth, location, motion, HealthKit and Health Connect dialogs — are driven by the SDK from its own UI when the user reaches the feature that needs them. Your Dart code calls no permission API. One prompt comes earlier: `initializeWithToken` requests the notification permission (Android 13+ `POST_NOTIFICATIONS`; iOS alert, badge and sound) the first time it runs, so call it at a point where that prompt makes sense, or request the permission yourself beforehand. The exception is the headless calls: because there is no SDK UI to prompt from, a missing permission makes them return a typed reason (`bluetoothPermissionRequired`, `healthConnectPermissionRequired`, …) instead of prompting — request the permission first, for example with [`permission_handler`](https://pub.dev/packages/permission_handler). See [API Reference → Headless Calls](07-api-reference.md#headless-calls).

---

**Previous:** [Installation](02-installation.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
