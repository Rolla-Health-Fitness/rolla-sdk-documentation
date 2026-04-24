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
| SDK opens then immediately closes or shows an error | Expired access token | Fetch a fresh token from your backend before calling `show()` |
| `onTokenExpired` fires immediately | Token was already expired at launch | Ensure `tokenExpiresIn` reflects the *remaining* lifetime, not the original TTL |
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

**Cause:** the SDK bundles Flutter plugin AARs (Mapbox, Bluetooth, etc.) under the Flutter-assigned coordinate `<plugin>:1.0`. Because the `:1.0` version string never changes across SDK releases, Gradle's local module cache can reuse old resolved metadata for the plugin — pulling in older Mapbox native libraries that are ABI-incompatible with the new plugin Kotlin code.

**Fix:** force Gradle to re-resolve dependency metadata:

```bash
./gradlew clean --refresh-dependencies
```

Then rebuild and reinstall the app. The new build will pull the correct transitive versions declared by the bundled plugin AARs.

Do this after every SDK version bump. If the issue persists, clear the local cache for the Mapbox plugin:

```bash
rm -rf ~/.gradle/caches/modules-2/files-2.1/com.mapbox.maps.mapbox_maps
rm -rf ~/.gradle/caches/modules-2/metadata-2.*
./gradlew clean --refresh-dependencies
```

## Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
