# Prerequisites

Before you begin integrating the Rolla SDK into your Android application, ensure you have the following requirements in place.

## Requirements

- Android API 24 (Android 7.0) or later
- Android Studio Hedgehog (2023.1) or later
- Gradle 8.0+
- JDK 17 or later to run Gradle (required by Android Gradle Plugin 8.x). The SDK is built with Java 21; the published AAR is consumable on JDK 17+ via core library desugaring.
- App-level `sourceCompatibility` and `targetCompatibility` may be set to `JavaVersion.VERSION_11` or higher.
- Partner ID and API credentials from Rolla

## SDK Binary Size

The Rolla SDK embeds a Flutter engine, Mapbox native libraries, and several supporting dependencies. This adds approximately **20–40 MB** to your APK download size (the debug APK is significantly larger; release builds with R8 minification and ABI splits are smaller).

Key contributors:
- **Flutter engine** — the cross-platform UI runtime (native `.so` libraries per ABI)
- **Mapbox libraries** — map rendering for activity tracking
- **Rolla SDK core** — the SDK's compiled Dart application and native plugins

The SDK also contributes a significant method count. If your app targets API levels below 21 or has not yet enabled multidex, ensure `multiDexEnabled true` is set in your `build.gradle.kts` `defaultConfig` block.

> **Note:** Exact size varies by SDK version. Use Android Studio's APK Analyzer (`Build > Analyze APK`) or the [bundletool](https://developer.android.com/tools/bundletool) `get-size` command for a precise measurement. Publishing an App Bundle (`.aab`) with per-ABI splits will yield the smallest download size.

---

**Next:** [Gradle Setup](02-gradle-setup.md) | **Home:** [README](README.md)
