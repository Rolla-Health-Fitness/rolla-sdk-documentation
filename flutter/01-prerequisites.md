# Prerequisites

Before integrating the Rolla SDK into your Flutter app, verify the following.

## Requirements

| Requirement | Value |
|-------------|-------|
| **Flutter / Dart** | Flutter `3.35.6` or newer, Dart `3.9.2` or newer — the package constrains `flutter: '>=3.35.6'`, so an older toolchain fails at `flutter pub get`. Check yours with `flutter --version` |
| **iOS** | Deployment target `14.0`, CocoaPods. See [iOS Prerequisites](../ios/01-prerequisites.md) |
| **Android** | `minSdk 26`, `compileSdk 36` (the Flutter 3.35 default), Kotlin `2.2.0` or newer, JDK 17 to build, core library desugaring. See [Android Prerequisites](../android/01-prerequisites.md) |
| **Mapbox token** | A Mapbox **public** token (`pk.`), supplied by Rolla with your partner credentials. The SDK renders route maps with it; they stay blank without it. See [Permissions → Mapbox token](03-permissions.md#mapbox-token) |
| **Partner ID and sandbox credentials** | Issued with your Rolla SDK starter package during onboarding (contact [support@rolla.app](mailto:support@rolla.app)) |
| **Devices** | Physical iPhone and Android handsets for Bluetooth and GPS features. Simulators and emulators run everything else |

The native floors come from the SDK's bundled plugins — `health` (Health Connect / HealthKit) sets the `minSdk` and the iOS target, `flutter_local_notifications` needs the desugaring — not from the SDK's own code. The exact build changes are in [Installation](02-installation.md).

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md).

Because your app consumes the Dart package directly (not a prebuilt native AAR or pod), **your app — not the SDK — owns the platform permission declarations**. See [Permissions](03-permissions.md).

## Versioning

The package is released in lockstep with the native SDK, and the package version names the SDK version it contains:

| pub.dev `rolla_sdk` | SDK version | Notes |
|---------------------|-------------|-------|
| **`0.1.15`** (current) | `0.1.15` | Every option the 0.1.15 native wrappers expose, as `initializeWithToken` parameters, plus `openScreen` and the headless calls |
| `0.1.12`, `0.1.11` | pre-release snapshots (June 2026) | Superseded; no longer supported |
| `0.2.0` and later | Equal to the package version | Every official SDK release ships a pub.dev release carrying the same number |

- **Adding the package** with `flutter pub add rolla_sdk` writes `rolla_sdk: ^0.1.15` to your `pubspec.yaml`; your `pubspec.lock` pins the exact version your app was built with. Commit the lockfile.
- **Upgrading is one number:** bump the package, run `cd ios && pod install` and `cd android && ./gradlew --refresh-dependencies` (see [Installation → Verify the build](02-installation.md#4-verify-the-build)), rebuild.
- **The changelog is the SDK changelog.** Every "Both platforms" entry applies to a Flutter host and the iOS / Android sections apply on the respective platform. Entries name the native wrapper surface (`RollaConfiguration`, `RollaDelegate` / `RollaListener`); in Flutter the same options are parameters of `RollaSDK.initializeWithToken` and the callbacks you pass to it — the mapping is in [Configuration](05-configuration.md).
- The native SDK's [iOS](../ios/README.md) and [Android](../android/README.md) guides apply to Flutter hosts one-to-one for everything inside the native projects. This guide links to the exact native section rather than repeating it.

## SDK Binary Size

The package adds the same Flutter plugins, Mapbox libraries, and Rolla core that the native guides describe — roughly **30–50 MB** on iOS after App Store thinning and **20–40 MB** on Android in a release APK with R8. See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Previous:** [Quick Start](00-quick-start.md) | **Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
