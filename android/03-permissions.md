# Permissions

Configure the required permissions and Mapbox token for the Rolla SDK to function properly.

## Internet Permission

Add the internet permission to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Mapbox Token

Add the Mapbox access token to `app/src/main/res/values/strings.xml` for map functionality:

```xml
<string name="mapbox_access_token">your-mapbox-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

> **Platform note:** On Android, the Mapbox token is placed in `strings.xml` because the Mapbox SDK reads it as a string resource. On iOS, the token goes in `Info.plist` under the `MBXAccessToken` key — this is the standard Mapbox convention for each platform.

> **Note:** Bluetooth, location, and foreground service permissions are declared by the SDK and merged automatically via manifest merger. You do not need to add them manually.

## Permission Summary

| Permission | Declared by | When Prompted | On Denial |
|------------|------------|---------------|-----------|
| Internet (`INTERNET`) | Host app (`AndroidManifest.xml`) | Never (auto-granted) | SDK cannot reach backend |
| Bluetooth (scan, connect, advertise) | SDK (manifest merger) | Runtime on Android 12+ | Band sync unavailable |
| Location (fine, coarse, background) | SDK (manifest merger) | Runtime when activity tracking starts | Route tracking unavailable |
| Foreground Service | SDK (manifest merger) | Never (normal permission) | Background operations may be interrupted |
| Mapbox Token | Host app (`strings.xml`) | N/A (not a permission) | Maps will not render |

> **Note:** Bluetooth, location, and foreground service permissions are declared by the SDK and merged automatically. The host app only needs to declare `INTERNET` and provide the Mapbox token.

---

**Previous:** [Gradle Setup](02-gradle-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
