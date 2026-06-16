# Changelog

**Tags used in this changelog:**

- `[breaking]` — changes that require immediate attention in order to avoid a failed build.
- `[feature]` — new SDK capability.
- `[improvement]` — existing behavior enhanced, optimized or refactored.
- `[documentation]` — documentation-only change.
- `[fix]` — bug fix.

---

## 0.1.11

### Both platforms

- **[breaking] Replaced the `modules` parameter on `RollaConfiguration` with `disabledModules`.** The previous `modules` (enable-list) parameter — and the previously undocumented `enabledModules` parameter — have been removed. Module configuration is now opt-*out*: pass a set of `RollaDisabledModule` values to hide a module's entire UI, or omit the parameter to keep everything enabled. `weight` and `bloodPressure` are the first two modules supported for disabling. Any integration that passed `modules`/`enabledModules` must switch to `disabledModules`. See [Android](android/05-branding-and-modules.md#module-configuration) and [iOS](ios/05-branding-and-modules.md#module-configuration) branding & modules guides and the [Android](android/08-api-reference.md#rolladisabledmodule) / [iOS](ios/10-api-reference.md#rolladisabledmodule) API references.

- **[feature] Added the `removeRollaBandReferences` flag to `RollaConfiguration`, default `true`.** When `true` the SDK UI uses generic "fitness device" wording; set it to `false` to show Rolla Band-specific references. See [Android](android/05-branding-and-modules.md#rolla-band-references) and [iOS](ios/05-branding-and-modules.md#rolla-band-references) branding & modules guides.

- **[feature] Smartphone-only workout tracking.** Workouts can now be started and tracked without a paired wearable, using the phone's pedometer and motion sensors. This adds a new permission requirement on each platform — see the Android and iOS notes below.

### Android

- **[breaking] `ACTIVITY_RECOGNITION` is now declared by the SDK manifest.** The bundled SDK manifest declares `android.permission.ACTIVITY_RECOGNITION` (API 29+) to read the phone's step counter for smartphone-only workouts. You no longer add it yourself — but ensure your host app does not strip it via `tools:node="remove"`, and that your Play Console listing covers the activity-recognition rationale. See [Smartphone-Only Workouts](android/03-permissions.md#smartphone-only-workouts-activity_recognition) and the updated [Permissions Rationale](android/03-permissions.md#permissions-rationale).

### iOS

- **[breaking] `NSMotionUsageDescription` now required in the host app's `Info.plist`.** Smartphone-only workouts use `CMPedometer`, and iOS hard-terminates any app that starts it without an `NSMotionUsageDescription` string declared. Add the key with a user-facing rationale or smartphone-only workouts will crash the app on first start. See [Motion & Fitness](ios/03-permissions-and-entitlements.md#motion--fitness-required-for-smartphone-only-workouts).

- **[improvement] Live Workout widget honors phone-only mode.** `LiveWorkoutAttributes.ContentState` gained an `isPhoneOnly` flag (defaulted to `false`) so the Dynamic Island and Lock Screen views can hide band-specific elements when a workout is tracked from the phone alone. If you copied an older data contract into your widget, add the field to match the current SDK. See [Live Activities](ios/09-live-activities.md#step-3-widget-extension-files).

---

## 0.1.10

### Both platforms

- **[feature] Added the `showSettingsButton` boolean config on `RollaConfiguration`.** Renders a Settings button on the Home screen that opens a sheet with Data Sources and Goals shortcuts. Defaults to `true` since most partners need it automatically. See [Android](android/08-api-reference.md) and [iOS](ios/10-api-reference.md) API references.

- **[improvement] Improved the GPS tracking to be more accurate on iOS and Android.** The location pipeline has been refactored on both platforms to hold steady when you're standing still, filter out GPS zig-zags, and recover cleanly when you start moving again. Additionally, the in-app map views got some general UX improvements and polishing.

- **[documentation] Added a permissions rationale matrix to the iOS and Android permissions docs.** Each permission is grouped by capability (Location, Bluetooth, Health Connect / Apple Health, etc.) and carries a required/optional status plus a partner-ready rationale suitable for App Store and Play Console submissions. See [Android permissions rationale](android/03-permissions.md#permissions-rationale) and [iOS permissions rationale](ios/03-permissions-and-entitlements.md#permissions-rationale).

### Android

- **[breaking] `minSdk` raised from 24 to 26.** Required by the bundled Health Connect plugin's manifest.

- **[breaking] Host-app Kotlin floor raised to 2.2.0.** Required by the bundled `health` plugin's transitive `kotlin-stdlib-jdk7:2.2.10`. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.

- **[feature] Google Health Connect support.** Android health data can now be synced and tracked from Health Connect. For more information about permissions, please see [Health Connect](android/03-permissions.md#health-connect-android).

- **[improvement] Build JDK floor lowered from 21 to 17.** AAR is now compiled with Java 17 (class file major version 61).

### iOS

- **[improvement] Simulator support added (Debug configuration).** `0.1.10` runs on iPhone simulators under the Debug configuration, in addition to the existing Release-on-device support. Hardware-backed features (Bluetooth, etc.) still only work on physical devices. See [iOS Prerequisites](ios/01-prerequisites.md).

---

## 0.1.9 — First stable release

`0.1.9` is the first stable release of the Rolla SDK. Per-change entries begin at `0.1.10`.
