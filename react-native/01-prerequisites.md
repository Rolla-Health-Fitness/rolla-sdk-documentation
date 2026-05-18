# Prerequisites

Before integrating the Rolla SDK into your React Native app, verify the following.

## Requirements

- **React Native 0.80.3 or later** — see [RN version floor](#react-native-version-floor) below for why
- **React 19.1.0 exact** — see [React version pin](#react-version-pin) below for why
- **New Architecture enabled** (`newArchEnabled=true`). The Rolla wrapper ships a codegen-backed TurboModule and does not run under the old bridge
- **iOS deployment target 15.1+** — see [iOS Prerequisites](../ios/01-prerequisites.md)
- **Android `minSdk = 26`, `compileSdk = 36`** — see [Android Prerequisites](../android/01-prerequisites.md)
- **AGP 8.9.1+, Kotlin 2.2.0+** — see [Android Prerequisites](../android/01-prerequisites.md)
- **Node 18+**, **npm 9+** (or **Yarn 4.x via corepack**)
- **Partner ID and API credentials from Rolla**

## React Native version floor

The Rolla wrapper requires **React Native ≥ 0.80.3**.

**iOS only:** the Rolla wrapper has been verified to build and run on RN `0.77.3` with new arch + TurboModule interop. The Swift / ObjC++ TurboModule does not depend on RN internals, so older RN versions on iOS will likely work.

**Android requires RN ≥ 0.80.3.** RN 0.80 was the first release whose bundled `react-native-gradle-plugin` allows Android Gradle Plugin `8.9.x`. The native `com.rolla.sdk:android_release` AAR transitively pulls AndroidX dependencies (`androidx.activity:1.12.x`, `androidx.core:1.18.x`, `androidx.navigationevent:1.0.x`) that require **AGP 8.9.1 + compileSdk 36**. On RN 0.77–0.79 the build fails at `:app:checkDebugAarMetadata` with:

```
Dependency 'androidx.activity:activity:1.12.4' requires Android Gradle plugin 8.9.1 or higher.
This build currently uses Android Gradle plugin 8.7.2.
```

Overriding AGP at the root `build.gradle` does not help — the RN gradle plugin re-imposes its bundled AGP version downstream. The partner must bump to RN ≥ 0.80.3.

## React version pin

`react-native@0.80.3` ships a pre-built renderer that hard-checks the running React version. Carets are not safe:

```jsonc
{
  "dependencies": {
    "react": "19.1.0"   // exact — NOT "^19.1.0"
  }
}
```

A caret (`^19.1.0`) resolves to whatever is current on npm (`19.2.x` at time of writing), which the renderer rejects at launch with `Incompatible React versions: react-native-renderer: 19.1.0`.

## What you don't need

- **No CocoaPods version pin beyond what RN itself requires.** System `pod 1.16.x` works.
- **No Mapbox account on Android.** The Maven repo is public.
- **No `bundle install` for iOS.** Use the system `pod`, not `bundle exec pod` — the RN template's vendored Bundler 1.17.x has Ruby 2.6 compatibility issues with newer CocoaPods.

## Native SDK pin

The Rolla wrapper's `0.1.x` series pins the native iOS pod and Android Maven artifact at **`0.1.10`**. You do not declare these versions yourself — the Rolla wrapper's podspec and `android/build.gradle` declare them. If you try to override the native version your build will fail fast. That is intentional.

When you bump `@rolla-health/react-native-sdk` to a version that points at a different native artifact, run:

```sh
cd android && ./gradlew --refresh-dependencies
```

Gradle caches transitive metadata (notably for Mapbox) per coordinate, and stale metadata produces confusing resolution errors. This cannot be fixed on Rolla's side; the refresh must happen on your machine.

## SDK binary size

The same Flutter engine, Mapbox libraries, and Rolla core that the native iOS and Android guides describe — adding **30–50 MB** on iOS (after App Store thinning) and **20–40 MB** on Android (release APK with R8). See [iOS Prerequisites → SDK Binary Size](../ios/01-prerequisites.md#sdk-binary-size) and [Android Prerequisites → SDK Binary Size](../android/01-prerequisites.md#sdk-binary-size).

---

**Next:** [Installation](02-installation.md) | **Home:** [README](README.md)
