# Permissions

The Rolla SDK uses Bluetooth Low Energy, Location, Motion, Health, and Photos. Unlike a native add-to-app host, a **pure-Flutter host consumes the Dart package** — so **your app must declare these permissions itself** in both `ios/Runner/Info.plist` and `android/app/src/main/AndroidManifest.xml`. The `flutter create` scaffold ships with **none** of them.

> **You DO add these.** The fresh Flutter scaffold has no Bluetooth/Health/Location strings. On device this was proven: iOS **SIGABRT-crashed** on first BLE access because `NSBluetoothAlwaysUsageDescription` was missing, and Android features failed at runtime until the permissions were in the manifest. The SDK does **not** inject them for you.

## iOS

Add the following keys to `ios/Runner/Info.plist`. Without them the app **aborts silently with SIGABRT** the moment the SDK touches the corresponding API — there is no Dart-side error you can catch.

| Key | Required for | Triggers crash on |
| --- | --- | --- |
| `NSBluetoothAlwaysUsageDescription` | BLE band pairing | `CBCentralManager` init |
| `NSBluetoothPeripheralUsageDescription` | BLE band pairing | `CBCentralManager` init |
| `NSLocationWhenInUseUsageDescription` | Outdoor activity tracking | `CLLocationManager` request |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Background activity tracking | `CLLocationManager` request |
| `NSMotionUsageDescription` | Activity recognition (phone-only) | `CMMotionManager` start |
| `NSHealthShareUsageDescription` | Apple Health read | HealthKit authorization |
| `NSHealthUpdateUsageDescription` | Apple Health write | HealthKit authorization |
| `NSPhotoLibraryAddUsageDescription` | Save activity images | Photo picker present |
| `UIBackgroundModes` (array: `location`, `bluetooth-central`, `remote-notification`) | Background band sync | App background transition |

Example block (from the demo `ios/Runner/Info.plist` — replace the strings with your own wording):

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth access lets the app connect to your fitness band and keep syncing health and activity data.</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app uses Bluetooth to connect to your fitness band and sync health data.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app uses your location to track outdoor activities like running and cycling.</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app tracks outdoor activities accurately even when the screen is off or the app is in the background.</string>
<key>NSMotionUsageDescription</key>
<string>The app uses motion sensors to count steps and measure cadence when no band is connected.</string>
<key>NSHealthShareUsageDescription</key>
<string>The app reads Apple Health data such as heart rate, steps, sleep, and workouts for personalised insights.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>The app writes workout and activity data to Apple Health so your records stay consistent.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Save your activity images to Photos.</string>
<key>UIBackgroundModes</key>
<array>
  <string>location</string>
  <string>bluetooth-central</string>
  <string>remote-notification</string>
</array>
```

For the exact strings and full rationale (used in App Store Connect privacy questionnaires), and for where to supply the **Mapbox access token**, see [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md). The Mapbox token is provided by Rolla along with your partner credentials.

## Android

Add the following permissions to `android/app/src/main/AndroidManifest.xml`, inside the `<manifest>` element (above `<application>`). Because you consume the Dart package — not the native AAR — **AGP's manifest-merger will not pull these in for you**; you declare them.

```xml
<uses-permission android:name="android.permission.INTERNET"/>

<!-- Bluetooth (band pairing) -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.BLUETOOTH" android:maxSdkVersion="30"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" android:maxSdkVersion="30"/>

<!-- Location (outdoor activity tracking) -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>

<!-- Motion / step counting when no band is connected -->
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION"/>

<!-- Background workout tracking + notifications -->
<uses-permission android:name="android.permission.WAKE_LOCK"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>

<!-- Health Connect (optional secondary data source) -->
<uses-permission android:name="android.permission.health.READ_HEART_RATE"/>
<uses-permission android:name="android.permission.health.READ_STEPS"/>
<uses-permission android:name="android.permission.health.READ_ACTIVE_CALORIES_BURNED"/>
<uses-permission android:name="android.permission.health.READ_SLEEP"/>
<uses-permission android:name="android.permission.health.READ_WEIGHT"/>
<uses-permission android:name="android.permission.health.READ_BLOOD_PRESSURE"/>
<uses-permission android:name="android.permission.health.READ_EXERCISE"/>
<uses-permission android:name="android.permission.health.READ_DISTANCE"/>
<uses-permission android:name="android.permission.health.READ_TOTAL_CALORIES_BURNED"/>
```

This block is taken from the demo `android/app/src/main/AndroidManifest.xml`. For the per-permission rationale, the Health Connect `<queries>` block, and where to supply the **Mapbox access token**, see [Android Permissions](../android/03-permissions.md).

> **Mapbox metadata caches across SDK bumps.** If maps fail to render after upgrading `rolla_sdk`, your consumer Gradle cache is reusing stale Mapbox metadata. Fix it with `./gradlew --refresh-dependencies` from `android/` — this is not a CI toggle.

## Runtime prompts

Runtime permission prompts (the Bluetooth dialog on iOS, the runtime permission dialogs on Android API 31+) are handled by the **SDK itself** when the user reaches the relevant feature inside `RollaSdkHome`. Your Flutter code does not call any permission API. You may pre-prompt before launching the SDK if your onboarding UX prefers it, but this is not required.

---

**Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
