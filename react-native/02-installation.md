# Installation

Install the wrapper from npm, then apply the iOS Podfile and Android Gradle changes required by the underlying native SDK.

## 1. npm install

```sh
npm install @rolla-health/react-native-sdk --legacy-peer-deps
```

After installing, pin React to exact `19.1.0` (see [Prerequisites → React version pin](01-prerequisites.md#react-version-pin)):

```sh
npm pkg set 'dependencies.react=19.1.0'
```

### Why `--legacy-peer-deps`?

The RN 0.80 template ships `react-test-renderer@18.x` in `devDependencies`, which declares a `peerDependencies.react@^18.2.0`. Your project also depends on `react@19.1.0` (required by `react-native@0.80.3`). npm 7+ treats this conflict as fatal:

```
npm error Could not resolve dependency:
npm error peer react@"^18.2.0" from react-test-renderer@18.2.0
```

`--legacy-peer-deps` restores npm 6 behaviour: warn but install. This is safe because `react-test-renderer` only runs under Jest — at runtime your app only ever loads `react@19.1.0`.

If you don't run snapshot tests, you can also drop `react-test-renderer` from `devDependencies` and skip the flag entirely.

## 2. iOS — Podfile

`RollaSDK` is distributed via a public CocoaPods specs repository on GitHub. Edit `ios/Podfile`:

```ruby
require Pod::Executable.execute_command('node', ['-p',
  'require.resolve(
    "react-native/scripts/react_native_pods.rb",
    {paths: [process.argv[1]]},
  )', __dir__]).strip

platform :ios, '15.1'

# IMPORTANT: order matters — Rolla source must come BEFORE the CocoaPods CDN.
source 'https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios.git'
source 'https://cdn.cocoapods.org/'

prepare_react_native_project!

# Required: RollaSDK vendors xcframeworks (Flutter, Mapbox, etc.).
# Static linkage keeps your other RN pods working.
use_frameworks! :linkage => :static

# Required: Flipper does not support framework linkage.
ENV['NO_FLIPPER'] = '1'

target 'YourApp' do
  config = use_native_modules!

  # Required: NordicDFU.xcframework (vendored by RollaSDK) was pre-built
  # expecting ZIPFoundation as a *dynamic* framework. Under our global
  # static linkage, ZIPFoundation would otherwise be statically linked
  # into the app binary and not embedded in Frameworks/, causing
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
        # .swiftinterface imports ZIPFoundation; if ZIPFoundation is
        # rebuilt at 15.1 the swiftinterface fails to compile with
        # "compiling for iOS 14.0, but module 'ZIPFoundation' has a
        # minimum deployment target of iOS 15.1". Pin ZIPFoundation
        # to iOS 14.0; the host app deployment target stays at 15.1.
        if target.name == 'ZIPFoundation'
          c.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '14.0'
        end
      end
    end
  end
end
```

Then:

```sh
cd ios && pod install
```

> Use the system `pod` (CocoaPods `1.16.x`). The RN template's vendored Bundler 1.17.x has Ruby 2.6 compatibility issues with current CocoaPods — calling `bundle exec pod install` will likely fail with an ActiveSupport error.

Open the `.xcworkspace`, not the `.xcodeproj`.

**Bundle ID note:** do **not** pass `PRODUCT_BUNDLE_IDENTIFIER=...` as a global `xcodebuild` override — CocoaPods will then assign your app's bundle ID to its sub-frameworks (e.g. ZIPFoundation) and `devicectl install` rejects the app with `parent bundle has the same identifier as sub-bundle`. Set the bundle ID directly in the app target's `project.pbxproj`.

For the underlying iOS configuration (signing, build settings, etc.) see [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md).

## 3. iOS — Info.plist

The fresh RN template ships only an empty `NSLocationWhenInUseUsageDescription`. The Rolla SDK uses Bluetooth, Location, Motion, Health, and Photos — you must add usage strings for each or the app will abort silently with **SIGABRT** at `Rolla.show()`. See [Permissions → iOS](03-permissions.md#ios) for the exact keys.

## 4. iOS — `.xcode.env`

Xcode's script-phase shell does not source your interactive PATH. `command -v node` returns empty in script phases, which breaks Hermes' `replace-config` step. Hard-code an absolute path in `ios/.xcode.env`:

```sh
export NODE_BINARY=/opt/homebrew/bin/node
```

Adjust if your Node lives elsewhere (`which node` in your shell tells you).

## 5. Android — `settings.gradle`

The native `com.rolla.sdk:android_release` artifact and its Flutter / Mapbox transitive dependencies live in three separate public Maven repositories. Register them in `android/settings.gradle`:

```groovy
pluginManagement { includeBuild("../node_modules/@react-native/gradle-plugin") }
plugins { id("com.facebook.react.settings") }
extensions.configure(com.facebook.react.ReactSettingsExtension){ ex -> ex.autolinkLibrariesFromCommand() }

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

rootProject.name = 'YourApp'
include ':app'
includeBuild('../node_modules/@react-native/gradle-plugin')
```

For the rationale behind each Maven repo see [Android Gradle Setup](../android/02-gradle-setup.md).

## 6. Android — `build.gradle`

In `android/build.gradle`, pin AGP `8.9.1`, Kotlin `2.2.0`, `compileSdk 36`, `minSdk 26`:

```groovy
buildscript {
    ext {
        buildToolsVersion = "35.0.0"
        minSdkVersion = 26
        compileSdkVersion = 36
        targetSdkVersion = 36
        ndkVersion = "27.1.12297006"
        kotlinVersion = "2.2.0"
    }
    repositories { google(); mavenCentral() }
    dependencies {
        classpath("com.android.tools.build:gradle:8.9.1")
        classpath("com.facebook.react:react-native-gradle-plugin")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin")
    }
}

apply plugin: "com.facebook.react.rootproject"
```

In `android/app/build.gradle`, enable core-library desugaring (the native SDK uses `java.time`):

```groovy
android {
    // ...
    compileOptions {
        coreLibraryDesugaringEnabled true
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
}

dependencies {
    implementation("com.facebook.react:react-android")
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.4'
    // ...
}
```

Ensure `ANDROID_HOME` is set or write `sdk.dir=$HOME/Library/Android/sdk` (adjust for your OS) into `android/local.properties`.

## 7. Order of operations

Always:

1. `npm install --legacy-peer-deps` (or `yarn install`)
2. `cd ios && pod install`
3. Build

If you delete `node_modules` and reinstall, you may need to wipe `ios/Pods` and `android/.gradle` / `android/build` to clear stale codegen artifacts.

---

**Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
