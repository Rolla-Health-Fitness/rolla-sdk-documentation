# Installation

Add the `rolla_sdk` package from pub.dev, then apply the iOS and Android platform changes its bundled native plugins require.

> A Flutter host depends on the **Dart package** — there are no extra Maven repositories or Podfile sources to add. Flutter's tooling resolves the SDK's native dependencies for you; you only apply the platform floors below and the [Permissions](03-permissions.md) in the next step.

## 1. Add the package

```sh
flutter pub add rolla_sdk
```

This pins the current release in `pubspec.yaml`:

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

> Requires **Flutter 3.35.6 / Dart 3.9.2** or newer — see [Prerequisites](01-prerequisites.md).

## 2. iOS — deployment target and pods

Set the deployment target to **14.0** in `ios/Podfile`:

```ruby
platform :ios, '14.0'
```

Then install the pods:

```sh
cd ios && pod install
```

Open the generated `Runner.xcworkspace`, not `Runner.xcodeproj`. If you bump the deployment target after a first build, run `pod install` again.

For deeper native build configuration, see [iOS CocoaPods Setup](../ios/02-cocoapods-setup.md).

## 3. Android — Gradle

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

In `android/settings.gradle.kts`, use Kotlin **2.2.0 or newer** and build with **JDK 17+**:

```kotlin
plugins {
    id("dev.flutter.flutter-plugin-loader") version "1.0.0"
    id("com.android.application") version "8.9.1" apply false
    id("org.jetbrains.kotlin.android") version "2.2.0" apply false
}
```

For the rationale behind each requirement, see [Android Gradle Setup](../android/02-gradle-setup.md) and [Android Prerequisites](../android/01-prerequisites.md).

## 4. Verify the build

Build for each platform to confirm the floors resolve before writing integration code:

```sh
flutter run                       # debug on a connected device
flutter build apk                 # Android release sanity check
flutter build ios --no-codesign   # iOS build sanity check
```

---

**Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
