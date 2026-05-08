# Changelog

All notable changes to the Rolla SDK are documented here. This log starts at `0.1.10`; `0.1.9` is listed as the first stable release without per-change entries.

Format loosely follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); the project follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html). Each entry is tagged with one of: `[breaking]`, `[feature]`, `[improvement]`, `[fix]`.

---

## 0.1.10 — Draft

- **[breaking] Android `minSdk` raised from 24 to 26.** Required by the bundled Health Connect plugin's manifest. See [Why minSdk 26?](android/01-prerequisites.md#why-minsdk-26).

- **[breaking] Host-app Kotlin floor raised to 2.2.0.** Required by the bundled `health` plugin's transitive `kotlin-stdlib-jdk7:2.2.10`. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum. Failure mode and fix in [Troubleshooting → Kotlin compiler crash](android/09-troubleshooting.md).

- **[feature] Google Health Connect support on Android.** Android health data now flows from both the Rolla band and Health Connect. Host-app `AndroidManifest.xml` must declare the permissions, the rationale intent-filter, the `ViewPermissionUsageActivity` alias, and `<queries>` for `com.google.android.apps.healthdata` — see [Health Connect](android/03-permissions.md#health-connect-android).

- **[feature] `showSettingsButton` config on `RollaConfiguration` (both platforms).** Renders a Settings button on the Home screen that opens a sheet with Data Sources and Goals shortcuts. Defaults to `true`. See [Android](android/08-api-reference.md) and [iOS](ios/10-api-reference.md) API references.

- **[improvement] Android build JDK floor lowered from 21 to 17.** AAR is now compiled with Java 17 (class file major version 61), CI-validated. JDK 21 still works.

- **[improvement] Expanded Android troubleshooting** with entries for the `minSdk = 24` manifest-merger failure, the Kotlin metadata crash, the Health Connect "View permissions" no-op, and the missing-`<queries>` detection failure. See [Troubleshooting](android/09-troubleshooting.md).

- **[improvement] iOS simulator support added (Debug configuration).** `0.1.10` runs on iPhone simulators under the Debug configuration, in addition to the existing Release-on-device support. Hardware-backed features (Bluetooth, etc.) still only work on physical devices. See [iOS Prerequisites](ios/01-prerequisites.md).

- **[improvement] Expanded the iOS and Android permissions docs with a rationale matrix grouped by capability:** each permission now carries a one-line rationale in the summary table plus a longer required/optional + rationale entry below, with partner-ready justification text suitable for App Store and Play Console submissions. See [Android permissions rationale](android/03-permissions.md#permissions-rationale) and [iOS permissions rationale](ios/03-permissions-and-entitlements.md#permissions-rationale).

---

## 0.1.9 — First stable release

`0.1.9` is the first stable release of the Rolla SDK. Per-change entries begin at `0.1.10`.
