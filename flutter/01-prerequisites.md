# Prerequisites

Before integrating the Rolla SDK into your Flutter app, verify the following.

## Requirements

| Requirement | Value |
|-------------|-------|
| **Flutter / Dart** | Flutter `3.35.6` or newer, Dart `3.9.2` or newer — the package constrains `flutter: '>=3.35.6'`, so an older toolchain fails at `flutter pub get`. Check yours with `flutter --version` |
| **iOS** | Deployment target `14.0`, CocoaPods. See [iOS Prerequisites](../ios/01-prerequisites.md) |
| **Android** | `minSdk 26`, `compileSdk 36` (the Flutter 3.35 default), Kotlin `2.2.0` or newer, JDK 17 to build, core library desugaring. See [Android Prerequisites](../android/01-prerequisites.md) |
| **Mapbox token** | A Mapbox **public** token (`pk.`), supplied by Rolla with your partner credentials. The SDK renders route maps with it; they stay blank without it. See [Permissions → Info.plist](03-permissions.md#infoplist) (iOS) and [Mapbox Token](03-permissions.md#mapbox-token) (Android) |
| **Partner ID and sandbox credentials** | Issued with your Rolla SDK starter package during onboarding (contact [support@rolla.app](mailto:support@rolla.app)) |
| **Devices** | Bluetooth band pairing and motion-sensor tracking need a physical device; simulators and emulators run everything else |

The floors are set by the package's own Android and iOS build files and its bundled plugins (`health` for Health Connect / HealthKit, `mapbox_maps_flutter`, `flutter_local_notifications` for the desugaring). The exact build changes are in [Installation](02-installation.md).

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md).

The platform declarations a fresh `flutter create` scaffold is missing are listed in [Permissions & Entitlements](03-permissions.md).

## Versioning

The package is released in lockstep with the native SDK, and the package version names the SDK version it contains:

| pub.dev `rolla_sdk` | SDK version | Notes |
|---------------------|-------------|-------|
| **`0.1.15+1`** (current) | `0.1.15` | The 0.1.15 configuration options as `initializeWithToken` parameters, plus `openScreen` and the headless calls; see [API Reference → Not Available in Flutter](07-api-reference.md#not-available-in-flutter) for the exceptions |
| `0.1.15` | `0.1.15` | Same code as `0.1.15+1`; its pub.dev page listed a non-existent iOS entitlement, so pin `0.1.15+1` instead. |
| `0.1.12`, `0.1.11` | pre-release snapshots (June 2026) | Superseded; no longer supported |
| `0.2.0` and later | Equal to the package version | Every official SDK release ships a pub.dev release carrying the same number |

- **Pin the exact version:** `flutter pub add rolla_sdk:0.1.15+1` writes `rolla_sdk: 0.1.15+1` to your `pubspec.yaml`. A caret (`^0.1.15`) would let `flutter pub upgrade` pick up any later 0.1.x release, and 0.1.x releases can carry `[breaking]` changes — upgrade deliberately.
- **Upgrading is one number:** bump the package, run `cd ios && pod install` and `cd android && ./gradlew --refresh-dependencies` (see [Installation → Verify the build](02-installation.md#4-verify-the-build)), rebuild.
- **The changelog is the SDK changelog.** Every "Both platforms" entry applies to a Flutter host and the iOS / Android sections apply on the respective platform. Entries name the native wrapper surface (`RollaConfiguration`, `RollaDelegate` / `RollaListener`); in Flutter the same options are parameters of `RollaSDK.initializeWithToken` and the callbacks you pass to it — the mapping is in [Configuration](05-configuration.md).
- The native SDK's [iOS](../ios/README.md) and [Android](../android/README.md) guides apply to Flutter hosts one-to-one for everything inside the native projects. This guide links to the exact native section rather than repeating it.

## SDK Binary Size

The package adds the same Flutter plugins, Mapbox libraries, and Rolla core that the native guides describe — roughly **30–50 MB** on iOS after App Store thinning and **20–40 MB** on Android in a release APK with R8. See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Previous:** [Quick Start](00-quick-start.md) | **Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
