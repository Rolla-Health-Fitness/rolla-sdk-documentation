# Prerequisites

Before integrating the Rolla SDK into your React Native app, verify the following.

## Requirements

| Requirement | Value |
|-------------|-------|
| **React Native** | `0.80.3` with the **New Architecture** enabled (`newArchEnabled=true`). The wrapper is a codegen-backed TurboModule and runs Bridgeless; the legacy architecture is not supported. See [React Native version floor](#react-native-version-floor) |
| **React** | Exactly the version your React Native release pins — `19.1.0` for RN 0.80.3, no caret. See [React version pin](#react-version-pin) |
| **JavaScript engine** | Validated with Hermes, the React Native default |
| **iOS** | Deployment target `15.1`; CocoaPods with `use_frameworks! :linkage => :static`. See [iOS Prerequisites](../ios/01-prerequisites.md) |
| **Android** | `minSdk 26`, `compileSdk` / `targetSdk` `36`, Kotlin `2.2.0`, JDK 17 to build, core-library desugaring. See [Android Prerequisites](../android/01-prerequisites.md) |
| **Partner ID** | Provided by Rolla during onboarding (contact [support@rolla.app](mailto:support@rolla.app)) |
| **Devices** | Physical iPhone and Android handsets — Bluetooth and GPS features are not available in simulators or emulators |

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md).

## React Native version floor

The wrapper is validated end-to-end on React Native **0.80.3** on physical iOS and Android devices, and `0.80` is the floor.

The reason is the Android toolchain: the native Android SDK ships with Kotlin 2.2 metadata, and React Native's bundled `react-native-gradle-plugin` must itself be compiled against Kotlin ≥ 2.1 — where `org.jetbrains.kotlin.gradle.dsl.KotlinTopLevelExtension` became an interface (it was a class in Kotlin ≤ 2.0). RN 0.80 was the first release whose gradle plugin moved to Kotlin 2.1.20. Older React Native versions fail under any Kotlin 2.1+ pin with:

```
Found interface org.jetbrains.kotlin.gradle.dsl.KotlinTopLevelExtension, but class was expected
```

Overriding the Kotlin or Gradle plugin versions at the root `build.gradle` does not help — the React Native gradle plugin re-imposes its own downstream. Upgrade to React Native 0.80.3 or later.

## React version pin

`react-native@0.80.3` ships a pre-built renderer that hard-checks the running React version at launch. Carets are not safe:

```jsonc
{
  "dependencies": {
    "react": "19.1.0",                          // exact — NOT "^19.1.0"
    "react-native": "0.80.3",
    "@rolla-health/react-native-sdk": "0.1.16"  // exact — see Versioning
  }
}
```

A caret (`^19.1.0`) resolves to whatever is current on npm (`19.2.x` at the time of writing), which the renderer rejects at launch with `Incompatible React versions: react-native-renderer: 19.1.0`. This is a React Native constraint, not a Rolla one.

## Versioning

The package is released in lockstep with the native SDK, and the package version names the native version it links — with one documented exception:

| `@rolla-health/react-native-sdk` | Native SDK it links | Notes |
|----------------------------------|---------------------|-------|
| **`0.1.16`** (current) | **`0.1.15`** | A React Native-only release that completes the JavaScript surface of the 0.1.15 SDK (every host event, the headless methods, `openScreen`, notification taps). There is no native 0.1.16 |
| `0.1.15` | `0.1.15` | First lockstep release; presentation and token API only |
| `0.2.0` and later | Equal to the package version | Every official SDK release ships an npm release carrying the same number |

- **Pin the exact version** (`"0.1.16"`, not `"^0.1.16"`). The wrapper's podspec and `build.gradle` pin iOS pod `RollaSDK` and Android `com.rolla.sdk:android_release` exactly in turn — you do not declare these yourself. If your Podfile or Gradle files pin a conflicting version, the build fails fast; that is intentional.
- **Upgrading is one number:** bump the package, run `cd ios && pod install` and `cd android && ./gradlew --refresh-dependencies` (see [Installation → Verify the integration](02-installation.md#7-verify-the-integration)), rebuild.
- **`Rolla.getNativeSdkVersion()`** resolves with the linked native version — `'0.1.15'` for package `0.1.16`.
- The native SDK's [iOS](../ios/README.md) and [Android](../android/README.md) guides apply to React Native hosts one-to-one for everything inside the native projects. This guide links to the exact native section rather than repeating it.

## SDK Binary Size

The wrapper adds the same Flutter engine, Mapbox libraries, and Rolla core that the native guides describe — roughly **30–50 MB** on iOS after App Store thinning and **20–40 MB** on Android in a release APK with R8. See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Previous:** [Quick Start](00-quick-start.md) | **Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
