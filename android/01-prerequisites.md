# Prerequisites

Before you begin integrating the Rolla SDK into your Android application, ensure you have the following requirements in place.

## Requirements

- Android API 26 (Android 8.0) or later — `minSdk = 26`
- Android Studio Hedgehog (2023.1) or later
- Gradle 8.0+
- **Build JDK 17 or newer.** SDK `0.1.12` is compiled with Java 17 (class file major version 61)
- App-level `sourceCompatibility` / `targetCompatibility` can stay at `VERSION_11` or higher — only the build JDK is constrained.
- **Kotlin 2.2.0 or newer**. The bundled `health` Flutter plugin (used for Health Connect) ships a `kotlin-stdlib-jdk7:2.2.10` transitive dependency whose `kotlin_module` metadata is at version **2.2.0**. Older Kotlin compilers cannot read this metadata and fail with `IllegalArgumentException: source must not be null` inside `FirIncompatibleClassExpressionChecker`.
Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.
- Partner ID and API credentials from Rolla

## SDK Binary Size

The Rolla SDK embeds a Flutter engine, Mapbox native libraries, and several supporting dependencies, which increase your APK download size. The exact amount depends on your build (the debug APK is significantly larger; release builds with R8 minification and ABI splits are smaller) — see the note below to measure it.

Key contributors:
- **Flutter engine** — the cross-platform UI runtime (native `.so` libraries per ABI)
- **Mapbox libraries** — map rendering for activity tracking
- **Rolla SDK core** — the SDK's compiled Dart application and native plugins

> **Note:** Exact size varies by SDK version. Use Android Studio's APK Analyzer (`Build > Analyze APK`) or the [bundletool](https://developer.android.com/tools/bundletool) `get-size` command for a precise measurement. Publishing an App Bundle (`.aab`) with per-ABI splits will yield the smallest download size.

---

**Previous:** [Quick Start](00-quick-start.md) | **Next:** [Gradle Setup](02-gradle-setup.md) | **Home:** [README](README.md)
