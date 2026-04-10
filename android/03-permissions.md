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

---

**Previous:** [Gradle Setup](02-gradle-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
