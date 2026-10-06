# Troubleshooting

Flutter-specific symptoms and remedies. For SDK runtime issues inside the native layers — Apple Health, Health Connect, Bluetooth, GPS, background tracking, Gradle and CocoaPods resolution — see [iOS Troubleshooting](../ios/11-troubleshooting.md) and [Android Troubleshooting](../android/09-troubleshooting.md); everything there applies to a Flutter host unchanged.

## Common issues

### App aborts on launch or on the first SDK screen (iOS SIGABRT)

**Symptom:** the app crashes the moment you push `RollaSdkHome`, or as soon as the SDK touches Bluetooth, Location or Motion:

```
* thread #1, ... stop reason = signal SIGABRT
... "This app has crashed because it attempted to access privacy-sensitive
data without a usage description. The app's Info.plist must contain an
NSBluetoothAlwaysUsageDescription key ..."
```

There is no Dart-side exception — the abort happens in the native layer before any Dart error can be caught.

**Cause:** a missing iOS usage string. A Flutter host declares these itself, and a fresh `flutter create` scaffold has none.

**Fix:** add every required key to `ios/Runner/Info.plist`. See [Permissions → iOS](03-permissions.md#ios).

### The SDK's back button does nothing

**Symptom:** you pass `showBackButton: true`, the back button renders in the SDK top bar, but tapping it has no effect — the user is stuck inside the SDK with no way back to your app.

**Cause:** `RollaSdkHome` owns its own `MaterialApp.router`, so its back button cannot pop *your* `Navigator`. In a pure-Flutter host, `showBackButton: true` alone is inert — the SDK needs the `onRequestDismiss` callback to ask your app to dismiss it.

**Fix:** also pass `onRequestDismiss` to `initializeWithToken`:

```dart
await RollaSDK.initializeWithToken(
  // ...
  showBackButton: true,
  onRequestDismiss: () {
    if (mounted) Navigator.of(context).pop();
  },
);
```

See [Code Integration → Host dismissal](04-code-integration.md#host-dismissal--showbackbutton--onrequestdismiss). The same callback handles back from a screen opened with `openScreen`.

### `openScreen` resolves `uiUnavailable`

**Cause:** `RollaSdkHome` was not on screen within 15 seconds of the call. The router lives inside the widget, so a request can only be served once it has mounted and its startup gates have settled.

**Fix:** push the route that renders `RollaSdkHome` right after calling `openScreen`, or keep the widget as your authenticated root. See [API Reference → Host-driven navigation](07-api-reference.md#host-driven-navigation).

### Android build fails: `minSdkVersion cannot be smaller than ... 26`

**Symptom:**

```
Manifest merger failed : uses-sdk:minSdkVersion 21 cannot be smaller than
version 26 declared in library [...health...]
```

**Cause:** the bundled `health` plugin (Health Connect) requires `minSdk 26`; the Flutter default is lower.

**Fix:** set `minSdk = 26` in `android/app/build.gradle.kts`. See [Installation → Android](02-installation.md#3-android--gradle).

### Android build fails during dexing: desugaring required

**Symptom:** the build fails during D8/dexing with a message about Java 8+ APIs or `Default interface methods`, typically pointing at `flutter_local_notifications`:

```
D8: Default interface methods are only supported starting with Android N (--min-api 24): ...
```

**Cause:** a transitive dependency uses Java 8+ APIs that require **core library desugaring**, which a fresh Flutter scaffold does not enable.

**Fix:** enable desugaring and add the `desugar_jdk_libs` dependency in `android/app/build.gradle.kts`. See [Installation → Android](02-installation.md#3-android--gradle).

### Android build fails with a Kotlin metadata or `class file has wrong version` error

**Cause:** the Kotlin plugin version in `android/settings.gradle.kts` is below 2.2.0, or the build runs on a JDK older than 17.

**Fix:** set `org.jetbrains.kotlin.android` to `2.2.0` or newer and build with JDK 17. See [Installation → Android](02-installation.md#3-android--gradle) and [Android Troubleshooting → Build errors](../android/09-troubleshooting.md#build-errors).

### `flutter pub get` fails with a version-solving error

**Symptom:**

```
Because rolla_sdk >=0.1.15 requires Flutter SDK version >=3.35.6,
version solving failed.
```

or a conflict on `mapbox_maps_flutter`, `health` or `device_info_plus`.

**Cause:** your Flutter/Dart toolchain is below the floor, or one of your own dependencies conflicts with the versions `rolla_sdk` pins exactly (`mapbox_maps_flutter 2.22.0`, `health 13.3.1`, `device_info_plus 12.3.0`).

**Fix:** confirm your toolchain meets **Flutter 3.35.6 / Dart 3.9.2** with `flutter --version` (upgrade with `flutter upgrade`). For a plugin conflict, align your own constraint with the pinned version; `flutter pub deps` shows which dependency carries the conflicting constraint.

### Maps stay blank

**Cause:** `MBXAccessToken` (iOS) or `mapbox_access_token` (Android) is missing, or holds a secret (`sk.`) token. The SDK runs without it, but every map stays blank.

**Fix:** add your public token — see [Permissions → Mapbox token](03-permissions.md#mapbox-token) and [Android Troubleshooting → Maps not showing](../android/09-troubleshooting.md#maps-not-showing).

### Routing breaks or the theme looks wrong inside the SDK

**Cause:** `RollaSdkHome` was wrapped in another `MaterialApp`. It builds its own `MaterialApp.router` and owns navigation and theming from that point.

**Fix:** push it onto a route from your existing app (`Navigator.push` / a `GoRoute`) instead of nesting it in a `MaterialApp`. See [Code Integration → Placing `RollaSdkHome`](04-code-integration.md#placing-rollasdkhome).

### Stale native dependencies after bumping `rolla_sdk`

**Symptom:** after upgrading the package, the Android build fails to resolve a transitive native dependency, or runtime behavior doesn't match the release notes.

**Cause:** Gradle caches transitive dependency metadata per coordinate and can reuse stale entries across SDK version bumps.

**Fix:** refresh the local Gradle cache:

```sh
cd android && ./gradlew --refresh-dependencies
```

See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

### Token-related issues

- **`onTokenExpired` keeps firing** — your callback must return a *fresh* pair; a pair older than the one the SDK holds is ignored by design. Obtain new tokens via `/api/login`, directly or through your backend.
- **`onSessionExpired` fires right after initializing** — the pair you passed was already consumed or revoked (for example a refresh token your backend also used). Issue the SDK its own pair. See [Token Management → Your app's responsibilities](06-token-management.md#your-apps-responsibilities).
- **The SDK cannot refresh on its own** — pass all three of `accessToken`, `refreshToken` and `tokenExpiresIn`, and never spend the refresh token in your own backend calls.

### Clean-build reset

When in doubt, regenerate plugin registrants and flush caches:

```sh
flutter clean
flutter pub get
cd ios && pod install && cd ..
cd android && ./gradlew --refresh-dependencies && cd ..
flutter run
```

### Getting debug logs for support tickets

The SDK logs through the platform logging systems:

- **iOS:** unified logging under subsystem `app.rolla.rollaV2` — filter Console.app by it, or `log stream --device --predicate 'subsystem == "app.rolla.rollaV2"' --level info`.
- **Android:** `adb logcat -s RollaSdkPlugin:* Flutter:*`.

Attach the filtered log to your ticket. More symptoms and remedies: [iOS Troubleshooting → debug logs](../ios/11-troubleshooting.md#getting-debug-logs-for-support-tickets) and [Android Troubleshooting → debug logs](../android/09-troubleshooting.md#getting-debug-logs-for-support-tickets).

## Support

Open a support ticket with:

- `rolla_sdk` version (`flutter pub deps | grep rolla_sdk`)
- Flutter/Dart version (`flutter --version`)
- Platform, OS version, device model
- Verbatim error text, not a paraphrase
- The filtered debug log from above

- **Email:** [support@rolla.app](mailto:support@rolla.app)
- **Slack:** If your organization has a dedicated partner Slack channel with Rolla, use it for faster responses on integration questions and live debugging.

---

**Previous:** [API Reference](07-api-reference.md) | **Home:** [README](README.md)
