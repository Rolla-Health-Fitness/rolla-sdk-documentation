# Permissions

The Rolla SDK uses Bluetooth Low Energy, Location, Motion, Health, and Photos. Because a Flutter host consumes the Dart package (not a prebuilt AAR/pod), **your app must declare these permissions itself** in `ios/Runner/Info.plist` and `android/app/src/main/AndroidManifest.xml` — a fresh `flutter create` scaffold ships with none of them, and the SDK does not inject them for you.

## iOS

Add the following keys to `ios/Runner/Info.plist`. Without them the app **aborts with SIGABRT** the moment the SDK touches the corresponding API — there is no Dart-side error you can catch.

| Key | Required for |
| --- | --- |
| `NSBluetoothAlwaysUsageDescription` | BLE band pairing |
| `NSBluetoothPeripheralUsageDescription` | BLE band pairing |
| `NSLocationWhenInUseUsageDescription` | Outdoor activity tracking |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Background activity tracking |
| `NSMotionUsageDescription` | Activity recognition (phone-only tracking) |
| `NSHealthShareUsageDescription` | Apple Health read |
| `NSHealthUpdateUsageDescription` | Apple Health write (optional — the SDK only reads) |
| `NSPhotoLibraryAddUsageDescription` | Save activity images |
| `UIBackgroundModes` (array: `location`, `bluetooth-central`; add `remote-notification` if you forward Rolla pushes) | Background tracking, band sync |

Write the usage strings in your own wording — they are shown to your users and referenced in App Store Connect privacy questionnaires. For the per-permission rationale, see [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md); the `rolla-sdk-demo-flutter` reference app contains a complete working `Info.plist` with example strings for every key.

## Android

Add the following to `android/app/src/main/AndroidManifest.xml`, inside `<manifest>` above `<application>`. This block mirrors the demo app's manifest:

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

For the per-permission rationale (useful for your Play Console **Data safety** form), the Health Connect `<queries>` block, and the complete list of Health Connect read permissions the SDK can use (including heart-rate variability, exercise routes, and speed), see [Android Permissions](../android/03-permissions.md).

## Mapbox token

The SDK renders maps for activity tracking and needs the Mapbox access token Rolla provides with your partner credentials. As with the permissions above, a Flutter host supplies it at the platform level:

- **iOS** — `MBXAccessToken` in `ios/Runner/Info.plist`; see [iOS Permissions & Entitlements → Mapbox Token](../ios/03-permissions-and-entitlements.md#mapbox-token)
- **Android** — `mapbox_access_token` in `res/values/strings.xml`; see [Android Permissions → Mapbox Token](../android/03-permissions.md#mapbox-token)

## Runtime prompts

Runtime permission prompts (the Bluetooth dialog on iOS, runtime dialogs on Android API 31+) are handled by the **SDK itself** when the user reaches the relevant feature inside `RollaSdkHome`. Your Flutter code does not call any permission API. You may pre-prompt before launching the SDK if your onboarding UX prefers it — see [Permission Gating](07-permissions-gate.md).

---

**Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
