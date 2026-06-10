# Troubleshooting

Flutter-specific symptoms and remedies. For underlying native build / SDK runtime issues, see [iOS Troubleshooting](../ios/11-troubleshooting.md) and [Android Troubleshooting](../android/09-troubleshooting.md).

## App aborts on launch / first SDK screen (iOS SIGABRT)

**Symptom:** the app crashes the moment you push `RollaSdkHome` (or as soon as the SDK touches Bluetooth/Location/Motion). Running on a device shows only:

```
* thread #1, ... stop reason = signal SIGABRT
... Thread 1: "This app has crashed because it attempted to access privacy-sensitive
data without a usage description. The app's Info.plist must contain an
NSBluetoothAlwaysUsageDescription key ..."
```

No Dart-side exception, no `flutter` log — the abort happens in the native layer before any Dart error can be caught.

**Cause:** a missing iOS usage string. Unlike the native pod consumer, a Flutter host ships the SDK's native code itself, so a fresh `flutter create` scaffold has **no** usage strings. The first privacy-sensitive API call (`CBCentralManager`, `CLLocationManager`, `CMMotionManager`, HealthKit, photo picker) then triggers `abort()`.

**Fix:** add every required key to `ios/Runner/Info.plist`. See [Permissions → iOS](03-permissions.md#ios) for the full list and [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md) for the exact strings and rationale.

## Android `flutter build` fails with manifest-merger `minSdkVersion 21 cannot be smaller than ... 26`

**Symptom:**

```
Manifest merger failed : uses-sdk:minSdkVersion 21 cannot be smaller than
version 26 declared in library [...health...]
Suggestion: use a compatible library with a minSdk of at most 21,
or increase this project's minSdk version to at least 26 ...
```

**Cause:** the bundled `health` plugin (Health Connect) requires `minSdk 26`. Your host app's default is lower.

**Fix:** set `minSdk = 26` in `android/app/build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        minSdk = 26
    }
}
```

See [Installation → Android](02-installation.md#3-android--gradle) and [Android Prerequisites](../android/01-prerequisites.md). For the native-side merger detail, see [Android Troubleshooting → Manifest merger](../android/09-troubleshooting.md#manifest-merger-fails-on-minsdk--24).

## Android build fails with `Default interface methods are only supported starting with Android N (--min-api 24)` / `flutter_local_notifications` desugaring error

**Symptom:** the build fails during D8/dexing with a message about Java 8+ APIs (e.g. `java.time`) or `Default interface methods`, pointing at `flutter_local_notifications`:

```
D8: Default interface methods are only supported starting with Android N (--min-api 24): ...
... use --core-library-desugaring ...
```

**Cause:** a transitive dependency (`flutter_local_notifications`) uses Java 8+ APIs that require **core library desugaring**, which a fresh Flutter scaffold does not enable.

**Fix:** enable desugaring in `android/app/build.gradle.kts`:

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

See [Installation → Android](02-installation.md#3-android--gradle).

## The SDK's back button does nothing

**Symptom:** you pass `showBackButton: true` to `RollaSdkHome`, the back button renders in the SDK top bar, but tapping it has no effect — the user is stuck inside the SDK with no way back to your app.

**Cause:** `RollaSdkHome` owns its own `MaterialApp.router`, so its back button cannot pop *your* `Navigator`. In a pure-Flutter host there is no native side listening on the `rolla_sdk/init` method channel to receive the dismiss, so `showBackButton: true` alone is inert.

**Fix:** also pass `onRequestDismiss` (added in **0.1.12** — bump `rolla_sdk` to `^0.1.12` if you don't have it) to pop your route:

```dart
RollaSdkHome(
  userId: widget.userId,
  showBackButton: true,
  onRequestDismiss: () {
    if (mounted) Navigator.of(context).pop();
  },
)
```

See [Code Integration → Host dismissal](04-code-integration.md#host-dismissal--showbackbutton--onrequestdismiss). (Native add-to-app hosts that present Flutter as a `FlutterViewController`/`FlutterActivity` receive the dismiss over the method channel instead and don't need this callback.)

## Maps don't update / wrong Mapbox version after bumping the SDK

**Symptom:** after bumping `rolla_sdk`, maps fail to render or Logcat reports a Mapbox native version that doesn't match the SDK release notes.

**Cause:** consumer Gradle caches reuse stale Mapbox dependency metadata across SDK versions.

**Fix:** flush the local Gradle cache — **not** a CI toggle:

```sh
cd android && ./gradlew --refresh-dependencies
```

If it persists, add a clean:

```sh
cd android && ./gradlew clean --refresh-dependencies
```

This must run on your machine; Rolla cannot fix it from the package side. See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

## Maps blank with no error

**Cause:** the Mapbox token is missing or not wired in at the platform level.

**Fix:** add the token Rolla provides with your partner credentials. See [Installation → Mapbox token](02-installation.md#4-mapbox-token), which links to the iOS and Android token locations.

## `flutter pub get` fails with a version-solving error

**Symptom:**

```
Because rolla_sdk >=0.1.12 requires SDK version >=3.9.2 and your app requires
SDK version <... , version solving failed.
```

or a conflict on a transitive plugin (`health`, `flutter_local_notifications`, Mapbox).

**Cause:** your Flutter/Dart toolchain is below the floor, or another direct dependency pins an incompatible version of a shared transitive plugin.

**Fix:**

1. Confirm your toolchain meets the floor (**Flutter 3.35.6 / Dart 3.9.2**):

   ```sh
   flutter --version
   ```

   Upgrade with `flutter upgrade` if below.
2. If a plugin conflicts, run `flutter pub deps` to find the constraint, then loosen your own pin or align it with the version `rolla_sdk` resolves. See [Prerequisites → Toolchain floor](01-prerequisites.md#toolchain-floor).

## `Navigator` / routing breaks, theme looks wrong inside the SDK

**Cause:** `RollaSdkHome` was wrapped in another `MaterialApp`. It builds its own `MaterialApp.router` and owns navigation and theming from that point.

**Fix:** push it onto a route from your existing app (`Navigator.push` / a `GoRoute`) instead of nesting it in a `MaterialApp`. See [Code Integration](04-code-integration.md#render-rollasdkhome).

## Clean-build reset

When in doubt, regenerate plugin registrants and flush caches:

```sh
flutter clean
flutter pub get
cd ios && pod install && cd ..
cd android && ./gradlew --refresh-dependencies && cd ..
flutter run
```

## Still stuck

Pair the Flutter-side symptoms with the platform troubleshooting guides:

- [iOS Troubleshooting](../ios/11-troubleshooting.md) — build errors, Apple Health, code-signing
- [Android Troubleshooting](../android/09-troubleshooting.md) — Gradle errors, Mapbox cache, Health Connect, manifest merger

Open a support ticket with:

- `rolla_sdk` version (`flutter pub deps | grep rolla_sdk`)
- Flutter/Dart version (`flutter --version`)
- Platform, OS version, device model
- Verbatim error text (not a paraphrase)

---

**Home:** [README](README.md)
