# Permissions

Configure the required permissions and Mapbox token for the Rolla SDK to function properly.

## 2.1 Internet Permission

Add the internet permission to your `AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## 2.2 Mapbox Token

Add the Mapbox access token to `app/src/main/res/values/strings.xml` for map functionality:

```xml
<string name="mapbox_access_token">your-mapbox-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

> **Note:** Bluetooth, location, and foreground service permissions are declared by the SDK and merged automatically via manifest merger. You do not need to add them manually.

---

**Previous:** [Gradle Setup](02-gradle-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
