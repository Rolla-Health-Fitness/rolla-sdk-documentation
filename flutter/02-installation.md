# Installation

Add the `rolla_sdk` package from pub.dev, then apply the iOS and Android changes its bundled plugins require. Everything below is a one-time change to your project.

> A Flutter host depends on the **Dart package** — there are no extra Maven repositories or Podfile sources to add. Flutter's tooling resolves the SDK's native dependencies for you; you only apply the platform floors below and the [Permissions](03-permissions.md) in the next step.

## 1. Add the package

```sh
flutter pub add rolla_sdk
```

This writes the current release to `pubspec.yaml`:

```yaml
dependencies:
  rolla_sdk: ^0.1.15
```

No authentication is required — the package is published to the public pub.dev registry. Requires **Flutter 3.35.6 / Dart 3.9.2** or newer, see [Prerequisites](01-prerequisites.md).

> `rolla_sdk` pins three plugins to exact versions on purpose — `mapbox_maps_flutter 2.22.0`, `health 13.3.1`, `device_info_plus 12.3.0` — because the SDK's native code is built against them. If your app depends on one of these directly, align your constraint with the pinned version or `flutter pub get` fails to solve.

## 2. iOS — deployment target and pods

Set the deployment target to **14.0** in `ios/Podfile`:

```ruby
platform :ios, '14.0'
```

Then install the pods:

```sh
flutter pub get
cd ios && pod install
```

Open the generated `Runner.xcworkspace`, not `Runner.xcodeproj`. If you bump the deployment target after a first build, run `pod install` again.

For the underlying native build settings, see [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md).

## 3. Android — Gradle

In `android/settings.gradle.kts`, use Kotlin **2.2.0 or newer** — a fresh Flutter scaffold ships an older one — and build with **JDK 17**:

```kotlin
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.9.1" apply false
    id("org.jetbrains.kotlin.android") version "2.2.0" apply false
}
```

In `android/app/build.gradle.kts`, set `minSdk = 26` and enable core library desugaring:

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }

    defaultConfig {
        minSdk = 26
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

`compileSdk` and `targetSdk` stay on Flutter's defaults (`flutter.compileSdkVersion`, API 36 with Flutter 3.35). Your app's `sourceCompatibility` / `targetCompatibility` can stay as the scaffold set them; the JDK 17 requirement is for the build itself.

For the rationale behind each floor, see [Android Gradle Setup](../android/02-gradle-setup.md) and [Android Prerequisites](../android/01-prerequisites.md).

**ProGuard / R8:** the package's Android half bundles consumer rules, so minified release builds need no manual configuration. Verify your release build with `minifyEnabled true` through the full flow once — see [Android Gradle Setup → ProGuard / R8](../android/02-gradle-setup.md#proguard--r8).

## 4. Verify the build

Build for each platform to confirm the floors resolve before writing integration code:

```sh
flutter run                       # debug on a connected device
flutter build apk                 # Android release sanity check
flutter build ios --no-codesign   # iOS build sanity check
```

> **Whenever you bump `rolla_sdk`, run:**
>
> ```sh
> cd android && ./gradlew --refresh-dependencies
> ```
>
> Gradle caches transitive metadata (notably for Mapbox) per coordinate, and stale metadata produces confusing resolution errors. This cannot be fixed on Rolla's side; the refresh must happen on your machine. See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

---

**Previous:** [Prerequisites](01-prerequisites.md) | **Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
