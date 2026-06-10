# Installation

Add the `rolla_sdk` package from pub.dev, then apply the small iOS and Android platform changes the bundled native plugins require.

> Unlike the React Native wrapper (which consumes a native AAR/pod), a Flutter host depends on the **Dart package**. There are no extra Maven specs repos or a custom Podfile to author — Flutter's tooling resolves the SDK's transitive native dependencies for you. You only need the platform floors below and the permissions in the next step.

## 1. Add the package

```sh
flutter pub add rolla_sdk
```

This pins the current release in `pubspec.yaml`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  rolla_sdk: ^0.1.12
```

Then fetch it:

```sh
flutter pub get
```

> Requires **Flutter 3.35.6 / Dart 3.9.2** or newer. See [Prerequisites](01-prerequisites.md) for the full toolchain floor.

No build flavors are required — the SDK works against the default `debug`/`release` flavors of a `flutter create` scaffold.

## 2. iOS — deployment target & pods

Set the iOS deployment target to **14.0** in `ios/Podfile`:

```ruby
platform :ios, '14.0'
```

Then install the CocoaPods that the SDK's native plugins (Mapbox, health, etc.) pull in:

```sh
cd ios && pod install
```

Open the generated `Runner.xcworkspace`, not `Runner.xcodeproj`. If you bump the deployment target after a first build, run `pod install` again so the pod targets pick it up.

> The `14.0` floor comes from the bundled `health` plugin. A lower target will fail to resolve pods.

For deeper native iOS configuration (signing, build settings, the `ENABLE_USER_SCRIPT_SANDBOXING` toggle for vendored frameworks) see [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md).

## 3. Android — Gradle

The bundled `health` and notification plugins need **minSdk 26**, **JDK 17+**, **Kotlin 2.2.0+**, and **core library desugaring**. Apply these in `android/app/build.gradle.kts`:

```kotlin
android {
    // ...
    compileOptions {
        // Required by the bundled health / notification plugins, which use
        // Java 8+ time APIs that need desugaring on older Android versions.
        isCoreLibraryDesugaringEnabled = true
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    defaultConfig {
        // The Rolla SDK bundles the `health` plugin, which requires API 26+.
        minSdk = 26
    }
}

dependencies {
    // Enables core library desugaring (see compileOptions above).
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

Pin **Kotlin 2.2.0 or newer** in `android/settings.gradle.kts`:

```kotlin
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.9.1" apply false
    id("org.jetbrains.kotlin.android") version "2.2.0" apply false // ≥ 2.2.0
}
```

> The Kotlin 2.2.0 floor is a transitive requirement of the bundled `health` plugin. Older compilers fail to read its `kotlin_module` metadata. Any patch ≥ 2.2.0 (2.2.10, 2.3.x, …) is forward-compatible. Build with **JDK 17+**.

For the rationale behind each requirement (the exact Kotlin metadata error, JDK bytecode version, APK size impact) see [Android Gradle Setup](../android/02-gradle-setup.md) and [Android Prerequisites](../android/01-prerequisites.md).

## 4. Mapbox token

The SDK renders maps and needs a Mapbox token, supplied by you (Rolla provides it with your partner credentials). You add it at the platform level — covered alongside the platform configs:

- iOS — see [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md) (`MBXAccessToken`).
- Android — see [Android Permissions → Mapbox Token](../android/03-permissions.md#mapbox-token).

> **Stale Mapbox metadata after an SDK bump:** consumer Gradle caches can reuse stale Mapbox dependency metadata across SDK versions and fail to resolve. Fix it with `./gradlew --refresh-dependencies` (run from `android/`) — this is a local cache flush, not a CI setting.

## 5. Verify the build

Build for each platform to confirm the native floors resolve before you write any integration code:

```sh
flutter run            # debug on a connected device/simulator
flutter build apk      # Android release sanity check
flutter build ios --no-codesign
```

If you ever delete `.dart_tool`/`Pods`/`android/.gradle` and reinstall, re-run `flutter pub get` and `pod install` to regenerate the plugin registrants.

---

**Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
