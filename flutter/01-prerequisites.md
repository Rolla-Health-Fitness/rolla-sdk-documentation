# Prerequisites

Before integrating the Rolla SDK into your Flutter app, verify the following.

## Requirements

The native floors come from the SDK's bundled plugins (chiefly `health` for Health Connect / HealthKit):

- **Flutter 3.35.6+ / Dart 3.9.2+** — the published package constrains `flutter: '>=3.35.6'`; older toolchains fail at `flutter pub get`. Check yours with `flutter --version`.
- **iOS deployment target 14.0+** — minimum for the bundled native frameworks.
- **Android `minSdk 26`** — Health Connect's minimum supported API level.
- **Build JDK 17+** — the SDK's native dependencies ship Java 17 bytecode; your app's `sourceCompatibility` / `targetCompatibility` can stay at `VERSION_11`.
- **Kotlin 2.2.0+** — required to read the bundled plugin's Kotlin metadata.
- **Core library desugaring** — a transitive dependency uses Java 8+ APIs that need desugaring on older Android versions.
- **A Mapbox access token** — the SDK renders maps for activity tracking; Rolla provides the token alongside your partner credentials. See [Permissions → Mapbox token](03-permissions.md#mapbox-token) for where it goes.
- **Partner ID and sandbox credentials from Rolla** — issued with your SDK starter package during onboarding (contact [support@rolla.app](mailto:support@rolla.app))

The exact build changes are in [Installation](02-installation.md). For the underlying details, see [Android Gradle Setup](../android/02-gradle-setup.md) and [iOS Prerequisites](../ios/01-prerequisites.md).

The SDK is the [`rolla_sdk`](https://pub.dev/packages/rolla_sdk) package on pub.dev, consumed as a normal Dart dependency:

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

Because your app consumes the Dart package directly (not a prebuilt native AAR/pod), **your app — not the SDK — owns the platform permission declarations**. See [Permissions](03-permissions.md).

## SDK binary size

The SDK adds roughly **30–50 MB** on iOS (after App Store thinning) and **20–40 MB** on Android (release APK with R8). See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
