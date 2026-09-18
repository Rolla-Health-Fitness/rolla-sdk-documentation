# Installation

Install the wrapper from npm, then apply the iOS Podfile and Android Gradle changes the underlying native SDK requires. Everything below is a one-time change to your `ios/` and `android/` projects.

## 1. Install the Package

```sh
npm install @rolla-health/react-native-sdk@0.1.16 --save-exact
# or
yarn add -E @rolla-health/react-native-sdk@0.1.16
```

Both flags write the exact version — a caret here is the one thing the [Versioning](01-prerequisites.md#versioning) rules forbid. No authentication is required — the package is published to the public npm registry. Your `package.json` should end up with an exact pin for the wrapper and for React (see [Prerequisites → React version pin](01-prerequisites.md#react-version-pin)):

```jsonc
{
  "dependencies": {
    "react": "19.1.0",
    "react-native": "0.80.3",
    "@rolla-health/react-native-sdk": "0.1.16"
  }
}
```

> **`--legacy-peer-deps` on a fresh template.** The React Native 0.80 template ships `react-test-renderer@18.x` in `devDependencies`, whose peer dependency `react@^18.2.0` conflicts with the `react@19.1.0` your app requires. npm 7+ treats that as fatal (`ERESOLVE … peer react@"^18.2.0" from react-test-renderer@18.2.0`). Install with `npm install … --legacy-peer-deps`, or drop `react-test-renderer` from `devDependencies` if you do not run snapshot tests. At runtime your app only ever loads `react@19.1.0`.

## 2. iOS — Podfile

`RollaSDK` is distributed through a public CocoaPods specs repository on GitHub. You must add it as a Podfile `source`, **and** enable framework linkage because the pod vendors pre-built `.xcframework` bundles (Flutter engine, Mapbox, and others).

Edit `ios/Podfile`:

```ruby
platform :ios, '15.1'

# IMPORTANT: order matters — the Rolla source must come BEFORE the CocoaPods CDN.
source 'https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios.git'
source 'https://cdn.cocoapods.org/'

# Required: RollaSDK vendors xcframeworks (Flutter, Mapbox, etc.).
# Static linkage keeps your other React Native pods working.
use_frameworks! :linkage => :static

# Required: Flipper does not support framework linkage.
ENV['NO_FLIPPER'] = '1'

target 'YourApp' do
  config = use_native_modules!

  # Required: NordicDFU.xcframework (vendored by RollaSDK) was pre-built
  # expecting ZIPFoundation as a *dynamic* framework. Under global static
  # linkage, ZIPFoundation would otherwise be statically linked into the app
  # binary and not embedded in Frameworks/, causing
  # `dyld: Library not loaded: @rpath/ZIPFoundation.framework/ZIPFoundation`
  # at app launch.
  pre_install do |installer|
    installer.pod_targets.each do |pod|
      if pod.name == 'ZIPFoundation'
        def pod.build_type
          Pod::BuildType.dynamic_framework
        end
      end
    end
  end

  use_react_native!(
    :path => config[:reactNativePath],
    :app_path => "#{Pod::Config.instance.installation_root}/.."
  )

  post_install do |installer|
    react_native_post_install(installer, config[:reactNativePath], :mac_catalyst_enabled => false)

    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |c|
        # Xcode 15+ ships User Script Sandboxing on by default. CocoaPods'
        # resource-copy scripts for vendored xcframeworks need it off.
        c.build_settings['ENABLE_USER_SCRIPT_SANDBOXING'] = 'NO'

        # NordicDFU.xcframework was pre-built targeting iOS 14.0. Its
        # .swiftinterface imports ZIPFoundation; if ZIPFoundation is rebuilt
        # at 15.1 the swiftinterface fails to compile with "compiling for
        # iOS 14.0, but module 'ZIPFoundation' has a minimum deployment
        # target of iOS 15.1". Pin ZIPFoundation to iOS 14.0; the host app
        # deployment target stays at 15.1.
        if target.name == 'ZIPFoundation'
          c.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'
        end
      end
    end
  end
end
```

Keep the `require` / `prepare_react_native_project!` lines your template already has above `platform`. Then:

```sh
cd ios && pod install
```

Open the `.xcworkspace`, not the `.xcodeproj`. For the underlying iOS build settings see [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md) — the `ENABLE_USER_SCRIPT_SANDBOXING` requirement documented there is what the `post_install` hook above applies for you.

### Transitive pod dependencies

`RollaSDK` vendors or depends on `ZIPFoundation` (`~> 0.9`), `TOCropViewController`, `MapboxMaps` / `MapboxCommon` / `MapboxCoreMaps` / `Turf`, and `NordicDFU`. If your app declares any of these directly, align the versions.

## 3. iOS — Project Notes

- **Bundle identifier:** do **not** pass `PRODUCT_BUNDLE_IDENTIFIER=…` as a global `xcodebuild` override. CocoaPods then assigns your app's bundle ID to its sub-frameworks (`ZIPFoundation`, for example) and `devicectl install` rejects the app with `parent bundle has the same identifier as sub-bundle`. Set the bundle ID in the app target's `project.pbxproj` instead.
- **`ios/.xcode.env.local`:** Xcode's script-phase shell does not source your interactive `PATH`. If the Hermes `replace-config` step fails with `: command not found`, put an absolute node path in the gitignored `ios/.xcode.env.local` (`export NODE_BINARY=/opt/homebrew/bin/node`, or your equivalent from `which node`). Do not edit the versioned `ios/.xcode.env`.
- **Permissions:** the fresh template ships only an empty `NSLocationWhenInUseUsageDescription`. Every other key the SDK needs is added by hand — see [Permissions & Entitlements → iOS](03-permissions.md#ios).

## 4. Android — `settings.gradle`

The native `com.rolla.sdk:android_release` artifact and its Flutter and Mapbox transitive dependencies live in three separate public Maven repositories. Register them in `android/settings.gradle` — libraries cannot declare repositories on your behalf under the strict resolution mode React Native templates use:

```groovy
dependencyResolutionManagement {
  // PREFER_SETTINGS — not FAIL_ON_PROJECT_REPOS. The React Native root
  // plugin (`com.facebook.react.rootproject`) registers its own repo at
  // the project level; FAIL_ON_PROJECT_REPOS rejects that and breaks the
  // build with "repository 'maven' was added by plugin 'com.facebook.react'".
  repositoriesMode.set(RepositoriesMode.PREFER_SETTINGS)
  repositories {
    google()
    mavenCentral()

    // Rolla SDK
    maven { url 'https://rolla-health-fitness.github.io/rolla-sdk-release-android/maven/' }

    // Flutter engine artifacts (bundled inside the Rolla SDK)
    maven { url 'https://storage.googleapis.com/download.flutter.io' }

    // Mapbox SDK (public, no token required)
    maven { url 'https://api.mapbox.com/downloads/v2/releases/maven' }
  }
}
```

Keep the `pluginManagement`, `plugins`, `rootProject.name`, `include ':app'` and `includeBuild` lines your template already has. For the rationale behind each repository see [Android Gradle Setup → Add Maven Repositories](../android/02-gradle-setup.md#add-maven-repositories).

## 5. Android — `build.gradle`

In `android/build.gradle`, set the Kotlin version the SDK is compiled with and the SDK levels it requires. Leave the Android Gradle Plugin version to React Native — its template already picks a compatible one:

```groovy
buildscript {
  ext {
    minSdkVersion = 26
    compileSdkVersion = 36
    targetSdkVersion = 36
    kotlinVersion = "2.2.0"
  }
}
```

See [Android Gradle Setup → Kotlin Version](../android/02-gradle-setup.md#kotlin-version) and [Build JDK](../android/02-gradle-setup.md#build-jdk) for why these floors exist.

## 6. Android — `app/build.gradle`

Enable core-library desugaring — the native SDK uses `java.time`:

```groovy
android {
  compileOptions {
    coreLibraryDesugaringEnabled true
    sourceCompatibility JavaVersion.VERSION_17
    targetCompatibility JavaVersion.VERSION_17
  }
}

dependencies {
  coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.4'
}
```

Make sure `ANDROID_HOME` is set, or `android/local.properties` defines `sdk.dir`.

**ProGuard / R8:** the AAR bundles consumer rules, so minified release builds need no manual configuration. Verify your release build with `minifyEnabled true` through the full flow once — see [Android Gradle Setup → ProGuard / R8](../android/02-gradle-setup.md#proguard--r8).

## 7. Verify the Integration

Autolinking registers the `RollaWrapper` TurboModule — nothing to add to `getPackages()` or the iOS factory. The wrapper assumes the React Native 0.80 scaffold: a Swift `AppDelegate.swift` built on `RCTReactNativeFactory` and a Kotlin `MainApplication.kt` extending `DefaultReactNativeHost`, with the New Architecture on in both `ios/YourApp/Info.plist` (`RCTNewArchEnabled`) and `android/gradle.properties` (`newArchEnabled=true`) — the template defaults.

Build on a physical device and call `Rolla.getNativeSdkVersion()` once on app load:

```ts
const version = await Rolla.getNativeSdkVersion(); // '0.1.15' for package 0.1.16
```

Resolving proves the TurboModule is wired up. If you instead get `Invariant Violation: TurboModuleRegistry.getEnforcing('RollaWrapper') could not be found`, autolinking did not pick up the wrapper — see [Troubleshooting](09-troubleshooting.md#invariant-violation-turbomoduleregistrygetenforcingrollawrapper-could-not-be-found).

> **Whenever you bump `@rolla-health/react-native-sdk` — every bump points at a different native artifact — run:**
>
> ```sh
> cd android && ./gradlew --refresh-dependencies
> ```
>
> Gradle caches transitive metadata (notably for Mapbox) per coordinate, and stale metadata produces confusing resolution errors. This cannot be fixed on Rolla's side; the refresh must happen on your machine. See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

## Order of Operations

1. `npm install` (or `yarn install`)
2. `cd ios && pod install`
3. Build

If you delete `node_modules` and reinstall, also wipe `ios/Pods`, `ios/build`, `android/.gradle` and `android/build` to clear stale codegen artifacts.

---

**Previous:** [Prerequisites](01-prerequisites.md) | **Next:** [Permissions & Entitlements](03-permissions.md) | **Home:** [README](README.md)
