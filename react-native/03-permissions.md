# Permissions

The Rolla SDK uses Bluetooth Low Energy, Location, Motion, Health, and Photos. The Rolla wrapper does not change which permissions are required — it surfaces them through the same native APIs the iOS and Android guides describe. Configure them at the platform level.

## iOS

Add the following keys to `ios/YourApp/Info.plist`. Without them the app **aborts silently with SIGABRT** the moment the SDK touches the corresponding API — there is no JS-side error you can catch.

| Key | Required for | Triggers crash on |
| --- | --- | --- |
| `NSBluetoothAlwaysUsageDescription` | BLE band pairing | `CBCentralManager` init |
| `NSBluetoothPeripheralUsageDescription` | BLE band pairing | `CBCentralManager` init |
| `NSLocationWhenInUseUsageDescription` | Outdoor activity tracking | `CLLocationManager` request |
| `NSLocationAlwaysAndWhenInUseUsageDescription` | Background activity tracking | `CLLocationManager` request |
| `NSMotionUsageDescription` | Activity recognition | `CMMotionManager` start |
| `NSHealthShareUsageDescription` | Apple Health read | HealthKit authorization |
| `NSPhotoLibraryUsageDescription` | Profile picture | Photo picker present |
| `NSPhotoLibraryAddUsageDescription` | Save activity images | Photo picker present |
| `MBXAccessToken` | Mapbox map rendering | Map tile fetch |
| `UIBackgroundModes` (array: `location`, `bluetooth-central`) | Background band sync | App background transition |

For the exact strings and full rationale (used in App Store Connect privacy questionnaires), see [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md). The `MBXAccessToken` is provided by Rolla along with your partner credentials.

> **Don't pass `PRODUCT_BUNDLE_IDENTIFIER` as a global `xcodebuild` override** — it leaks to Pods sub-frameworks (ZIPFoundation, etc.) and `devicectl install` rejects the app. Set the bundle ID in the app target's `project.pbxproj`.

## Android

**No manifest changes needed.** The native `com.rolla.sdk:android_release` AAR declares all required permissions (`BLUETOOTH_*`, `ACCESS_*_LOCATION`, `ACTIVITY_RECOGNITION`, foreground-service permissions) and AGP's manifest-merger pulls them into your `AndroidManifest.xml` automatically.

You **do** need to handle the Mapbox access token. See [Android Permissions → Mapbox Token](../android/03-permissions.md#mapbox-token).

For Health Connect declarations and the `<queries>` block see [Android Permissions → Health Connect](../android/03-permissions.md).

## Runtime prompts

Runtime permission prompts (`CBCentralManager` Bluetooth dialog on iOS, runtime permission dialogs on Android API 31+) are handled by the SDK itself when the user taps **Open Rolla**. Your RN code does not call any permission API. You can pre-prompt for permissions before opening the modal if your UX prefers it — but this is not required and not recommended unless you have a specific onboarding flow.

---

**Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
