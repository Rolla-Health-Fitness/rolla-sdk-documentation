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

## Health Connect (Android)

`0.1.10` adds Google Health Connect support. Unlike Bluetooth/location, the Health Connect permissions are **not** declared by the SDK — Google's policy review requires them to be declared by the host app so they appear in the Play Store listing under the host app's identity. You must add the entries below to your `AndroidManifest.xml`.

> **Why the SDK doesn't merge these for you:** Google scans declared `android.permission.health.*` entries against the host app's privacy policy and Data Safety form during Play review. If the SDK injected them via manifest merger, every partner would inherit Rolla's permission set without their privacy policy reflecting it — Play would reject the upload.

### Required permissions

Add inside the `<manifest>` element, alongside your other `<uses-permission>` entries:

```xml
<!-- Health Connect permissions -->
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE_VARIABILITY" />
<uses-permission android:name="android.permission.health.READ_STEPS" />
<uses-permission android:name="android.permission.health.READ_ACTIVE_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_SLEEP" />
<uses-permission android:name="android.permission.health.READ_WEIGHT" />
<uses-permission android:name="android.permission.health.READ_BLOOD_PRESSURE" />
<uses-permission android:name="android.permission.health.READ_EXERCISE" />
<uses-permission android:name="android.permission.health.READ_EXERCISE_ROUTES" />
<uses-permission android:name="android.permission.health.READ_SPEED" />
<uses-permission android:name="android.permission.health.READ_DISTANCE" />
<uses-permission android:name="android.permission.health.READ_TOTAL_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_HEALTH_DATA_HISTORY" />
```

### Permissions rationale intent-filter

Health Connect requires every app that requests health permissions to expose a "view permissions rationale" entry point so the user can read why each permission is requested. Add this intent-filter to **the activity that hosts the Rolla SDK** (the same `Activity` from which you call `rolla.show(activity)`):

```xml
<activity android:name=".MainActivity" ...>
    <!-- your existing intent-filters (LAUNCHER, deep links, etc.) -->

    <intent-filter>
        <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
    </intent-filter>
</activity>
```

When the user taps "View permissions" inside the Health Connect app, Android launches your activity with this action. The Rolla SDK detects the action and routes the user to the appropriate rationale screen automatically — you do not need to handle the intent in your code.

### View Permission Usage activity-alias

For Android 14+ (API 34) Health Connect compliance, declare a `ViewPermissionUsageActivity` alias that targets your SDK host activity. This is the entry point Android 14 uses when a user inspects which apps are reading their health data:

```xml
<activity-alias
    android:name=".ViewPermissionUsageActivity"
    android:exported="true"
    android:permission="android.permission.START_VIEW_PERMISSION_USAGE"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW_PERMISSION_USAGE" />
        <category android:name="android.intent.category.HEALTH_PERMISSIONS" />
    </intent-filter>
</activity-alias>
```

Replace `.MainActivity` with the activity that hosts the SDK. The alias must point at an activity that is `exported="true"` and that the system can launch directly — your SDK host activity satisfies both.

### Package visibility queries

Android 11+ (API 30) requires the host app to declare any package it intends to interact with via `<queries>`. The Health Connect client lives in `com.google.android.apps.healthdata`; without this entry, the SDK cannot detect whether Health Connect is installed on the device.

Add a `<queries>` block (or extend your existing one) at the `<manifest>` root:

```xml
<queries>
    <package android:name="com.google.android.apps.healthdata" />
    <intent>
        <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
    </intent>
</queries>
```

### `minSdk` requirement

Health Connect requires `minSdk = 26`. Set this in your `app/build.gradle.kts`:

```kotlin
android {
    defaultConfig {
        minSdk = 26
    }
}
```

If your project was previously on `minSdk = 24`, the manifest merger will fail with a clear error pointing at the `health` plugin's manifest until you bump the floor.

### Privacy policy & Data Safety

Before shipping a build with Health Connect enabled, update:

- Your **privacy policy** to disclose that the app reads the listed health data types from Health Connect.
- The **Data Safety** section of your Play Console listing to declare each data type as collected and what it is used for.

Google's policy review compares the manifest entries above against these two surfaces. A mismatch is the most common reason for a Play rejection on a Health Connect-enabled build.

### Demo

For a complete working `AndroidManifest.xml`, see the demo app's [Health Connect changes commit](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-android/commit/0f8a4b2a4060c07ec249d672009b5135d648eca0).

## Permission Summary

| Permission | Declared by | When Prompted | On Denial |
|------------|------------|---------------|-----------|
| Internet (`INTERNET`) | Host app (`AndroidManifest.xml`) | Never (auto-granted) | SDK cannot reach backend |
| Bluetooth (scan, connect, advertise) | SDK (manifest merger) | Runtime on Android 12+ | Band sync unavailable |
| Location (fine, coarse, background) | SDK (manifest merger) | Runtime when activity tracking starts | Route tracking unavailable |
| Foreground Service | SDK (manifest merger) | Never (normal permission) | Background operations may be interrupted |
| Activity Recognition (`ACTIVITY_RECOGNITION`) | Host app | Runtime when SDK opens Health Connect flow | Step / activity data unavailable |
| Health Connect (`android.permission.health.*`) | Host app | Granted per-type inside the Health Connect app | That data type is not synced |
| Mapbox Token | Host app (`strings.xml`) | N/A (not a permission) | Maps will not render |

> **Note:** Bluetooth, location, and foreground service permissions are declared by the SDK and merged automatically. The host app must declare `INTERNET`, the Health Connect entries listed above, and the Mapbox token.

---

**Previous:** [Gradle Setup](02-gradle-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
