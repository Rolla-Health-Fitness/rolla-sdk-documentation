# Prerequisites

Before integrating the Rolla SDK into your Flutter app, verify the following.

## Requirements

- **Flutter 3.35.6 or later**, **Dart 3.9.2 or later** — see [Toolchain floor](#toolchain-floor) below for why
- **iOS deployment target 14.0+** — see [iOS Prerequisites](../ios/01-prerequisites.md)
- **Android `minSdk = 26`** — see [Android Prerequisites](../android/01-prerequisites.md)
- **Build JDK 17+, Kotlin 2.2.0+, and core library desugaring enabled** on Android — see [Android floors](#android-floors) below
- **A Mapbox access token** — the SDK renders maps for activity tracking
- **Partner ID and API credentials from Rolla**

The Rolla SDK is the `rolla_sdk` package on [pub.dev](https://pub.dev/packages/rolla_sdk). You consume it as a normal Dart dependency:

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

Because your host app consumes the Dart package directly (not a prebuilt native AAR/pod), **your app — not the SDK — owns the platform permission declarations**. See [Permissions](03-permissions.md).

## Toolchain floor

The SDK targets **Flutter ≥ 3.35.6 / Dart ≥ 3.9.2**. The published package's `environment` constraint is `sdk: ^3.8.1`, `flutter: '>=3.35.6'`; building on an older toolchain fails at `flutter pub get` with a version-solving error. Confirm yours with:

```sh
flutter --version
```

## Android floors

The Android floors are driven by the bundled **`health`** Flutter plugin (Health Connect, `health: 13.3.1`), not by Rolla code directly:

- **`minSdk = 26`** — Health Connect's minimum supported API level.
- **Build JDK 17+** — the SDK and its native dependencies ship Java 17 bytecode (class file major version 61). The build JDK is what's constrained; your app's `sourceCompatibility` / `targetCompatibility` can stay at `VERSION_11`.
- **Kotlin 2.2.0+** — the `health` plugin pulls a `kotlin-stdlib` whose `kotlin_module` metadata is version 2.2.0. Older Kotlin compilers cannot read it and fail with `IllegalArgumentException: source must not be null` inside `FirIncompatibleClassExpressionChecker`.
- **Core library desugaring** — a transitive dependency (`flutter_local_notifications`) uses Java 8+ APIs that need desugaring on older Android versions.

Enable desugaring in `android/app/build.gradle.kts`:

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
    defaultConfig {
        minSdk = 26
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

For the full Gradle setup and the exact rationale behind each floor, see [Android Prerequisites](../android/01-prerequisites.md) and [Android Gradle Setup](../android/02-gradle-setup.md).

## iOS floor

The iOS deployment target is **14.0** (driven by the same bundled native frameworks). For Mapbox/CocoaPods specifics, see [iOS Prerequisites](../ios/01-prerequisites.md) and [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md).

## Mapbox token and partner credentials

The SDK renders maps for activity tracking, so you must supply a **Mapbox access token**. You also need a **Partner ID** and **API credentials** to authenticate SDK requests. Both are issued by Rolla alongside your onboarding.

Where the Mapbox token goes is platform-specific — `MBXAccessToken` in `Info.plist` on iOS, and the Mapbox download/runtime token on Android. See [Permissions](03-permissions.md) for the Flutter-specific summary, and the native guides for exact placement: [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md#mapbox-token) and [Android Permissions → Mapbox Token](../android/03-permissions.md#mapbox-token).

> **Gotcha:** When you bump the SDK to a version that points at a different Mapbox artifact, consumer Gradle caches reuse stale Mapbox metadata per coordinate and produce confusing resolution errors. Fix it on your machine with `./gradlew --refresh-dependencies` (run from `android/`). This is **not** a CI toggle and cannot be fixed on Rolla's side.

## SDK binary size

The SDK embeds a Flutter engine, Mapbox libraries, and the Rolla core — adding **30–50 MB** on iOS (after App Store thinning) and **20–40 MB** on Android (release APK with R8). See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
