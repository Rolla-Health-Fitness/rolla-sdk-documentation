# Troubleshooting

Flutter-specific symptoms and remedies. For underlying native build or runtime issues, see [iOS Troubleshooting](../ios/11-troubleshooting.md) and [Android Troubleshooting](../android/09-troubleshooting.md).

## App aborts on launch / first SDK screen (iOS SIGABRT)

**Symptom:** the app crashes the moment you push `RollaSdkHome`, or as soon as the SDK touches Bluetooth/Location/Motion:

```
* thread #1, ... stop reason = signal SIGABRT
... "This app has crashed because it attempted to access privacy-sensitive
data without a usage description. The app's Info.plist must contain an
NSBluetoothAlwaysUsageDescription key ..."
```

There is no Dart-side exception — the abort happens in the native layer before any Dart error can be caught.

**Cause:** a missing iOS usage string. A Flutter host declares these itself, and a fresh `flutter create` scaffold has none.

**Fix:** add every required key to `ios/Runner/Info.plist`. See [Permissions → iOS](03-permissions.md#ios).

## The SDK's back button does nothing

**Symptom:** you pass `showBackButton: true`, the back button renders in the SDK top bar, but tapping it has no effect — the user is stuck inside the SDK with no way back to your app.

**Cause:** `RollaSdkHome` owns its own `MaterialApp.router`, so its back button cannot pop *your* `Navigator`. In a pure-Flutter host, `showBackButton: true` alone is inert — the SDK needs the `onRequestDismiss` callback to ask your app to dismiss it.

**Fix:** also pass `onRequestDismiss` to `initializeWithToken` (requires `rolla_sdk` **0.1.12+**):

```dart
await RollaSDK.initializeWithToken(
  // ...
  showBackButton: true,
  onRequestDismiss: () {
    if (mounted) Navigator.of(context).pop();
  },
);
```

See [Code Integration → Host dismissal](04-code-integration.md#host-dismissal--showbackbutton--onrequestdismiss).

## Android build fails: `minSdkVersion cannot be smaller than ... 26`

**Symptom:**

```
Manifest merger failed : uses-sdk:minSdkVersion 21 cannot be smaller than
version 26 declared in library [...health...]
```

**Cause:** the bundled `health` plugin (Health Connect) requires `minSdk 26`; the Flutter default is lower.

**Fix:** set `minSdk = 26` in `android/app/build.gradle.kts`. See [Installation → Android](02-installation.md#3-android--gradle).

## Android build fails during dexing: desugaring required

**Symptom:** the build fails during D8/dexing with a message about Java 8+ APIs or `Default interface methods`, typically pointing at `flutter_local_notifications`:

```
D8: Default interface methods are only supported starting with Android N (--min-api 24): ...
```

**Cause:** a transitive dependency uses Java 8+ APIs that require **core library desugaring**, which a fresh Flutter scaffold does not enable.

**Fix:** enable desugaring and add the `desugar_jdk_libs` dependency in `android/app/build.gradle.kts`. See [Installation → Android](02-installation.md#3-android--gradle).

## `flutter pub get` fails with a version-solving error

**Symptom:**

```
Because rolla_sdk >=0.1.12 requires Flutter SDK version >=3.35.6,
version solving failed.
```

**Cause:** your Flutter/Dart toolchain is below the floor, or another direct dependency pins an incompatible version of a shared transitive plugin.

**Fix:** confirm your toolchain meets **Flutter 3.35.6 / Dart 3.9.2** with `flutter --version` (upgrade with `flutter upgrade`). For a plugin conflict, run `flutter pub deps` to find the constraint and align your own pin with the version `rolla_sdk` resolves.

## Routing breaks or theme looks wrong inside the SDK

**Cause:** `RollaSdkHome` was wrapped in another `MaterialApp`. It builds its own `MaterialApp.router` and owns navigation and theming from that point.

**Fix:** push it onto a route from your existing app (`Navigator.push` / a `GoRoute`) instead of nesting it in a `MaterialApp`. See [Code Integration → Placing `RollaSdkHome`](04-code-integration.md#placing-rollasdkhome).

## Stale native dependencies after bumping `rolla_sdk`

**Symptom:** after upgrading the package, the Android build fails to resolve a transitive native dependency, or runtime behavior doesn't match the release notes.

**Cause:** Gradle caches transitive dependency metadata per coordinate and can reuse stale entries across SDK version bumps.

**Fix:** refresh the local Gradle cache:

```sh
cd android && ./gradlew clean --refresh-dependencies
```

See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

## Clean-build reset

When in doubt, regenerate plugin registrants and flush caches:

```sh
flutter clean
flutter pub get
cd ios && pod install && cd ..
cd android && ./gradlew --refresh-dependencies && cd ..
flutter run
```

## Support

Pair the Flutter-side symptoms with the platform guides: [iOS Troubleshooting](../ios/11-troubleshooting.md) | [Android Troubleshooting](../android/09-troubleshooting.md).

For issues or questions, email [support@rolla.app](mailto:support@rolla.app) (or use your partner Slack channel), including:

- `rolla_sdk` version (`flutter pub deps | grep rolla_sdk`)
- Flutter/Dart version (`flutter --version`)
- Platform, OS version, device model
- Verbatim error text (not a paraphrase)

---

**Home:** [README](README.md)
