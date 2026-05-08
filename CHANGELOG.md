# Changelog

**Tags used in this changelog:**

- `[breaking]` — changes that require immediate attention in order to avoid a failed build.
- `[feature]` — new SDK capability.
- `[improvement]` — existing behavior enhanced, optimized or refactored.
- `[documentation]` — documentation-only change.
- `[fix]` — bug fix.

---

## 0.1.10 — Draft

### Both platforms

- **[feature] Added the `showSettingsButton` boolean config on `RollaConfiguration`.** Renders a Settings button on the Home screen that opens a sheet with Data Sources and Goals shortcuts. Defaults to `true` since most partners need it automatically. See [Android](android/08-api-reference.md) and [iOS](ios/10-api-reference.md) API references.

- **[documentation] Added a permissions rationale matrix to the iOS and Android permissions docs.** Each permission is grouped by capability (Location, Bluetooth, Health Connect / Apple Health, etc.) and carries a required/optional status plus a partner-ready rationale suitable for App Store and Play Console submissions. See [Android permissions rationale](android/03-permissions.md#permissions-rationale) and [iOS permissions rationale](ios/03-permissions-and-entitlements.md#permissions-rationale).

### Android

- **[breaking] `minSdk` raised from 24 to 26.** Required by the bundled Health Connect plugin's manifest.

- **[breaking] Host-app Kotlin floor raised to 2.2.0.** Required by the bundled `health` plugin's transitive `kotlin-stdlib-jdk7:2.2.10`. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.

- **[feature] Google Health Connect support.** Android health data now flows from both the Rolla band and Health Connect. For more information about permissions, please see [Health Connect](android/03-permissions.md#health-connect-android).

- **[improvement] Build JDK floor lowered from 21 to 17.** AAR is now compiled with Java 17 (class file major version 61).

### iOS

- **[improvement] Simulator support added (Debug configuration).** `0.1.10` runs on iPhone simulators under the Debug configuration, in addition to the existing Release-on-device support. Hardware-backed features (Bluetooth, etc.) still only work on physical devices. See [iOS Prerequisites](ios/01-prerequisites.md).

---

## 0.1.9 — First stable release

`0.1.9` is the first stable release of the Rolla SDK. Per-change entries begin at `0.1.10`.
