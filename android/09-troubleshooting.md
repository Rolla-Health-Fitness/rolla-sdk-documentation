# Troubleshooting

Common issues and solutions for integrating and running the Rolla SDK on Android.

## Common Issues

### SDK fails to start

- Ensure internet permission is in your manifest
- Check that the access token is valid and not expired
- Verify the partner ID is correct
- Check Logcat for errors with tag `RollaEngineManager` or `RollaSdkPlugin`

### Maps not showing

- Verify the Mapbox token is added to `app/src/main/res/values/strings.xml` (see [Permissions](03-permissions.md))
- Ensure the Mapbox Maven repository is in `settings.gradle.kts`

### Bluetooth / GPS not working

- Ensure location services are enabled on the device
- Grant location and Bluetooth permissions when prompted
- On Android 12+, `BLUETOOTH_SCAN` and `BLUETOOTH_CONNECT` are required at runtime

### Background tracking unreliable

- The SDK declares `FOREGROUND_SERVICE`, `FOREGROUND_SERVICE_LOCATION`, and `FOREGROUND_SERVICE_CONNECTED_DEVICE` permissions (merged automatically via manifest merger).
- See the [OEM Battery Optimization](#oem-battery-optimization) section below for device-specific issues.

### OEM Battery Optimization

Some Android manufacturers (Samsung, Xiaomi, Huawei, OnePlus, Oppo) aggressively kill background services to save battery. This can interrupt Bluetooth band sync, GPS tracking, and foreground service notifications even when the user has granted all permissions.

**Request battery optimization exemption:**

Add the permission to your manifest:

```xml
<uses-permission android:name="android.permission.REQUEST_IGNORE_BATTERY_OPTIMIZATIONS" />
```

Then prompt the user:

```kotlin
val intent = Intent(Settings.ACTION_REQUEST_IGNORE_BATTERY_OPTIMIZATIONS).apply {
    data = Uri.parse("package:${packageName}")
}
startActivity(intent)
```

**OEM-specific settings:** Each manufacturer has its own battery management UI. For detailed per-device instructions on how to whitelist your app, see [dontkillmyapp.com](https://dontkillmyapp.com) — a community-maintained guide covering Samsung, Xiaomi, Huawei, OnePlus, and others.

> **Note:** Even with the exemption, some OEM skins require the user to manually whitelist the app in their device's battery settings. Consider guiding users to the relevant settings page if background tracking is critical to your use case.

### Build errors

- Clean project: Build > Clean Project
- Invalidate caches: File > Invalidate Caches / Restart
- Clear Gradle cache: `./gradlew --refresh-dependencies`
- Ensure all three Maven repositories are configured in `settings.gradle.kts`

### `class file has wrong version 61.0, should be ...`

Starting with `0.1.10`, the SDK is compiled with Java 17 (class file major version 61) and the published AAR is validated in CI to never exceed major version 61. If your build JDK is older than 17, you'll see a `wrong version` error. Fix: point `JAVA_HOME` (or Android Studio → Settings → Build Tools → Gradle → Gradle JDK) to **JDK 17 or newer**.

> If you were previously on SDK `0.1.9` you may have set the build JDK to 21 to satisfy that release's Java 21 bytecode. JDK 21 still works with `0.1.10`, but JDK 17 is now the documented floor.

### Kotlin compiler crash: `IllegalArgumentException: source must not be null`

**Symptom:** the app fails to compile with a stack trace containing:

```text
IllegalArgumentException: source must not be null
    at org.jetbrains.kotlin.fir.analysis.checkers.FirIncompatibleClassExpressionChecker...
```

…or your build complains about reading `kotlin_module` metadata at version 2.2.0.

**Cause:** the SDK bundles the `health` Flutter plugin, which transitively pins `org.jetbrains.kotlin:kotlin-stdlib-jdk7:2.2.10`. That stdlib's `kotlin_module` files carry metadata version **2.2.0**. Kotlin's metadata compatibility rule is one-directional: a compiler at version X.Y can read metadata up to X.Y, never higher. Older compilers crash inside the K2 checker when they encounter the newer metadata.

**Fix:** raise the Kotlin version in your host app to **2.2.0 or newer**. Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum. In a typical Flutter Add-to-App or Compose project this lives in `gradle/libs.versions.toml`:

```toml
[versions]
kotlin = "2.2.0"   # any patch ≥ 2.2.0 works (2.2.10, 2.3.x, …)
```

Or, in legacy `build.gradle` setups:

```groovy
ext.kotlin_version = '2.2.0'
```

After bumping, run `./gradlew --refresh-dependencies` so the new Kotlin runtime is resolved.

### Manifest merger fails on `minSdk = 24`

**Symptom:** after upgrading to `0.1.10`, the Gradle build fails with a manifest merger error pointing at the bundled `health` plugin and complaining about `minSdkVersion`.

**Cause:** Health Connect requires `minSdk = 26`. The bundled `health` plugin's manifest declares this floor and the merger refuses to inject Health Connect components into a host app that targets an older API.

**Fix:** raise `minSdk` to 26 in `app/build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        minSdk = 26
    }
}
```

If you previously supported API 24, this drops Android 7.0 / 7.1 (Nougat) — verify your install base before shipping.

### Health Connect: nothing happens when the user taps "View permissions"

**Symptom:** the user opens the Health Connect app, taps your app's entry, and taps "View permissions" — but nothing happens, or Android shows "App not installed".

**Cause:** your host activity is missing the rationale intent-filter, or the `ViewPermissionUsageActivity` alias is missing/misconfigured.

**Fix:** verify your `AndroidManifest.xml` contains both the rationale intent-filter on the SDK host activity and the `ViewPermissionUsageActivity` alias. See [Permissions → Health Connect](03-permissions.md#health-connect-android).

### Health Connect: "Health Connect is not available on this device"

**Symptom:** the SDK reports that Health Connect is unavailable even on a device that has the Health Connect app installed.

**Cause:** the host app is missing the package-visibility entry for `com.google.android.apps.healthdata`. Starting on Android 11 (API 30), an app cannot detect another app's presence unless it declares it in `<queries>`.

**Fix:** add the `<queries>` block to your `AndroidManifest.xml`:

```xml
<queries>
    <package android:name="com.google.android.apps.healthdata" />
    <intent>
        <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
    </intent>
</queries>
```

### "Could not find com.rolla.sdk:android_release"

- Verify the Rolla SDK Maven repository URL is correct
- Check your network connection
- Run: `./gradlew --refresh-dependencies`

### Calling `show()` while the SDK is already presenting

If you call `show(activity)` while the SDK UI is already on screen, the SDK fires an `AlreadyPresenting` error through your listener:

```kotlin
override fun onRollaError(rolla: Rolla, error: RollaError) {
    if (error is RollaError.AlreadyPresenting) {
        // SDK is already showing — no action needed
    }
}
```

Guard your UI so the button or trigger that calls `show()` is disabled while the SDK is visible.

### Configuration changes (rotation, dark mode)

The SDK's Flutter activity handles configuration changes (screen rotation, dark mode toggle, language changes) internally. You do not need to add `android:configChanges` to your manifest — the SDK's activity declaration already includes the necessary flags via manifest merger.

If your **host activity** (the one that calls `show()`) is destroyed and recreated during a configuration change, keep in mind that your `Rolla` instance and listener will be lost. Store the `Rolla` reference in a `ViewModel` or re-attach the listener in `onCreate()`.

### Token-related issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| SDK opens then immediately closes or shows an error | Expired access token | Initialize with a current token pair — see [Token Management](06-token-management.md) |
| Works at first, then `Access denied (HTTP 401)` on SDK screens after ~30 minutes; re-entering the SDK temporarily fixes it | The SDK holds an already-consumed refresh token and `onTokenExpired` is not answered | See [Token Management — Symptoms](06-token-management.md#symptoms-of-incorrect-token-wiring) |
| `onTokenExpired` fires immediately at launch | The access token was already expired and the refresh token invalid at initialization | Initialize with the newest persisted pair — see [Token Management — Symptoms](06-token-management.md#symptoms-of-incorrect-token-wiring) |
| SDK works in `"rnd"` but fails in `"production"` | Token was issued for the wrong environment | Verify your backend issues tokens against the correct Rolla environment |
| "Unauthorized" or 401-style errors | Wrong partner ID or mismatched credentials | Double-check the `partnerId` passed to `RollaConfiguration` |

### Flutter engine crash recovery

If the Flutter engine crashes (rare, but possible under extreme memory pressure or after prolonged backgrounding), the SDK UI will close and `onRollaClosed` will fire. To recover:

1. Call `Rolla.destroyEngine()` to fully tear down the crashed engine.
2. Create a new `Rolla` instance with a fresh configuration.
3. Call `show(activity)` to restart.

```kotlin
override fun onRollaClosed(rolla: Rolla, reason: RollaCloseReason) {
    Rolla.destroyEngine()
    // Re-create and show when ready
}
```

See [Engine Lifecycle](07-engine-lifecycle.md) for more on `destroyEngine()`.

### FlutterJNI `nativeSetViewportMetrics` crash after upgrading the SDK

**Symptom:** after changing the Rolla SDK version, Flutter version, or Maven repository, the host app builds and launches, but crashes when the SDK initializes its Flutter engine. Logcat may show:

```text
Failed to register native method io.flutter.embedding.engine.FlutterJNI.nativeSetViewportMetrics(...)
[ERROR:flutter/shell/platform/android/platform_view_android_jni_impl.cc] Failed to RegisterNatives with FlutterJNI
[FATAL:flutter/shell/platform/android/library_loader.cc] Check failed: result.
Fatal signal 6 (SIGABRT)
```

**Cause:** the installed APK or local Gradle cache can contain a stale Flutter engine combination. In practice this means `libflutter.so` and `flutter_embedding_release` were resolved from different Flutter engine revisions, or the device is still running an APK installed before the dependency refresh.

**Fix:** do a full dependency refresh and reinstall the app:

```bash
adb shell am force-stop <your.package>
adb uninstall <your.package>

./gradlew --stop

rm -rf .gradle .kotlin app/build build
rm -rf ~/.gradle/caches/modules-2/files-2.1/com.rolla.sdk
rm -rf ~/.gradle/caches/modules-2/files-2.1/io.flutter
rm -rf ~/.gradle/caches/modules-2/files-2.1/com.github.Yalantis
rm -rf ~/.gradle/caches/*/transforms

./gradlew clean :app:assembleDebug --refresh-dependencies --no-build-cache --rerun-tasks
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

For release builds, replace `assembleDebug` and the APK path with your app's release variant. After reinstalling, confirm Logcat shows:

```text
RollaEngineManager: Flutter engine initialized
RollaEngineManager: SDK configured successfully
```

Run this cleanup after Rolla SDK upgrades if you see FlutterJNI registration errors, especially when testing multiple SDK builds on the same physical device.

### Getting debug logs for support tickets

The SDK logs to Logcat with the tags `RollaEngineManager` and `RollaSdkPlugin`. To capture logs for a support ticket:

```bash
adb logcat -s RollaEngineManager:* RollaSdkPlugin:* Flutter:*
```

Reproduce the issue while the log is running, then copy the output and attach it to your ticket. Including the `Flutter` tag captures any underlying Flutter engine errors.

In Android Studio, you can filter Logcat with:
```
tag:RollaEngineManager | tag:RollaSdkPlugin | tag:Flutter
```

### Gradle / Maven resolution issues

**"Could not resolve" or dependency conflict:**

```bash
# Force Gradle to re-download all dependencies
./gradlew --refresh-dependencies
```

**Repository order matters.** Ensure all three Maven repositories are in `settings.gradle.kts` (Rolla SDK, Flutter engine, Mapbox). See [Gradle Setup](02-gradle-setup.md).

**Version conflict with a transitive dependency:**
If the SDK pulls in a library version that conflicts with your app, use Gradle's `resolutionStrategy` to force a specific version:

```kotlin
configurations.all {
    resolutionStrategy {
        force("com.example:library:1.2.3")
    }
}
```

**Offline builds fail:** The SDK is hosted on a remote Maven repository and cannot be resolved offline. Ensure network access during the first build or when updating versions.

### Stale transitive dependencies after bumping the SDK version

**Symptom:** after bumping `com.rolla.sdk:android_release` to a new version, the app crashes with one of:

- `java.lang.ClassCastException: java.lang.Long cannot be cast to java.lang.Double` inside `com.mapbox.maps.mapbox_maps.pigeons.*`
- `java.lang.NoSuchMethodError: No direct method <init>(...) in class Lcom/mapbox/maps/plugin/annotation/AnnotationConfig;`
- Logcat shows a Mapbox native SDK version that doesn't match the SDK release notes (e.g. `[maps-core]: Using Mapbox Core Maps SDK v11.18.0` when the release expects `11.22.0`).
- Logcat shows `Failed to register native method io.flutter.embedding.engine.FlutterJNI.nativeSetViewportMetrics(...)`.

**Cause:** the SDK bundles Flutter plugin and engine artifacts that rely on Gradle metadata. After an SDK upgrade, Gradle's local cache can reuse old resolved metadata, mixing old native libraries with newer Kotlin or Flutter Java classes.

**Fix:** force Gradle to re-resolve dependency metadata:

```bash
./gradlew clean --refresh-dependencies
```

Then rebuild and reinstall the app. The new build will pull the correct transitive versions declared by the bundled plugin AARs.

Do this after every SDK version bump. If the issue persists, clear the affected local cache entries:

```bash
rm -rf ~/.gradle/caches/modules-2/files-2.1/com.rolla.sdk
rm -rf ~/.gradle/caches/modules-2/files-2.1/io.flutter
rm -rf ~/.gradle/caches/modules-2/files-2.1/com.mapbox.maps.mapbox_maps
rm -rf ~/.gradle/caches/modules-2/metadata-2.*
./gradlew clean --refresh-dependencies
```

## Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
