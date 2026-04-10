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
- For reliable background operation, consider requesting the user to exempt your app from battery optimization. Use `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` to prompt the user.
- On some OEM devices (Samsung, Xiaomi, Huawei), the user may need to manually disable battery optimization for your app in device settings.

### Build errors

- Clean project: Build > Clean Project
- Invalidate caches: File > Invalidate Caches / Restart
- Clear Gradle cache: `./gradlew --refresh-dependencies`
- Ensure all three Maven repositories are configured in `settings.gradle.kts`

### "Could not find com.rolla.sdk:android_release"

- Verify the Rolla SDK Maven repository URL is correct
- Check your network connection
- Run: `./gradlew --refresh-dependencies`

## Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
