# Permissions

Configure the required permissions and Mapbox token for the Rolla SDK to function properly. For the partner-facing justification of each permission for your privacy policy and Play Console **Data safety** form — see the [Permissions Rationale](#permissions-rationale) matrix at the bottom of this page.

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

## Health Connect (Android)

`0.1.10` adds Google Health Connect support. Unlike Bluetooth/location, the Health Connect permissions are **not** declared by the SDK — Google's policy review requires them to be declared by the host app so they appear in the Play Store listing under the host app's identity. You must add the entries below to your `AndroidManifest.xml`.

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

## Permissions Rationale

This section is the partner-facing justification for every permission the SDK requests on Android. The wording is intended to be lifted into a privacy policy or pasted into the **Play Console → App content → Data safety** form. Permissions are grouped by the user-visible capability they gate.

### Location

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `ACCESS_FINE_LOCATION` | Required | During an outdoor activity the SDK records the user's GPS trace to draw the route polyline, compute distance, pace, and elevation, and to attribute the workout to a specific place. Fine (GPS-grade) accuracy is what produces a clean, on-the-road polyline; coarse-only fixes drift across blocks and make pace and split data unusable. |
| `ACCESS_COARSE_LOCATION` | Required | Android pairs fine and coarse location and asks the user to choose between *Precise* and *Approximate* at the runtime prompt. The SDK declares both so the prompt presents the choice; if the user picks Approximate, the route exists but is reduced to neighborhood-level granularity. |
| `ACCESS_BACKGROUND_LOCATION` | Optional, strongly recommended | Outdoor workouts are routinely longer than the screen-on timeout. When the phone locks, Android moves the app to the background and the foreground location service requires this permission to keep streaming GPS fixes. Without it, **the polyline drops out the moment the user locks the phone or switches apps mid-workout**, leaving gaps in the recorded route. The SDK pairs background location with `FOREGROUND_SERVICE_LOCATION` and a persistent notification so the user can see that tracking is still active. |

### Bluetooth

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `BLUETOOTH_SCAN` (Android 12+) / `BLUETOOTH` (≤ Android 11) | Required | The SDK scans for nearby Rolla fitness bands during the pairing flow. The `neverForLocation` flag is set so this scan does *not* count as location access — Google's Data Safety form treats this distinction as material. |
| `BLUETOOTH_CONNECT` (Android 12+) / `BLUETOOTH_ADMIN` (≤ Android 11) | Required | After pairing, the SDK opens a GATT connection to the band to stream heart rate, steps, and activity events while the user wears the device. Without `BLUETOOTH_CONNECT`, the SDK can detect the band but cannot read from it. |
| `BLUETOOTH_ADVERTISE` | Optional | Currently declared for forward compatibility; the SDK does not actively advertise as a peripheral. Future band-companion features may require the host phone to advertise. |

### Health Connect

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `READ_HEART_RATE`, `READ_HEART_RATE_VARIABILITY` | Optional, recommended | Reads heart-rate samples and HRV recorded by other Health Connect-writing apps (Wear OS watches, Fitbit, Garmin Connect, etc.) so the user's Rolla feed combines band-recorded HR with off-band HR from their other devices. |
| `READ_EXERCISE`, `READ_EXERCISE_ROUTES` | Optional, recommended | Reads workout sessions written by other apps and their associated GPS traces. Used by `HealthConnectWorkoutDetailsBridge` to display a unified history of workouts regardless of which app recorded them. |
| `READ_STEPS`, `READ_DISTANCE`, `READ_SPEED` | Optional, recommended | Reads step counts, cumulative distance, and speed samples for daily activity totals and pace charts. These often originate from the device's onboard step counter (via Google Fit / Health Connect aggregation). |
| `READ_ACTIVE_CALORIES_BURNED`, `READ_TOTAL_CALORIES_BURNED` | Optional, recommended | Reads calories-burned samples (active = movement, total = active + basal) for the energy-balance view. |
| `READ_SLEEP` | Optional, recommended | Reads sleep sessions and stages so the user's sleep summary reflects the device they actually slept with (band, watch, ring, etc.). |
| `READ_WEIGHT`, `READ_BLOOD_PRESSURE` | Optional, recommended | Reads body-weight and blood-pressure measurements written by smart scales and BP cuffs that integrate with Health Connect, so trends in the Rolla profile reflect the user's full picture. |
| `READ_HEALTH_DATA_HISTORY` | Optional, recommended | By default Health Connect only exposes data recorded *after* the user grants a given permission. This permission lets the SDK read up to 30 days of history written before the grant, so the first-launch dashboard isn't artificially empty. |

### Activity Recognition

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `ACTIVITY_RECOGNITION` | Required for Health Connect step access on Android 10+ | Google's Health Connect docs require apps that read step data to declare this permission so the user understands that step counting depends on motion sensing. No SDK code currently calls the Activity Recognition Transition API directly. |

### Foreground Service

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `FOREGROUND_SERVICE` | Required | The SDK runs a foreground service while a workout is active so Android does not kill the process when the screen is off. The persistent notification it posts is the user's signal that tracking is in progress and is what the OS uses to justify the elevated process priority. |
| `FOREGROUND_SERVICE_LOCATION` (Android 14+) | Required for GPS-tracked activities | Android 14 split foreground services by purpose. `LocationService` is started with `ServiceInfo.FOREGROUND_SERVICE_TYPE_LOCATION` so the OS knows the service is consuming location and applies the correct policy (e.g. requiring `ACCESS_BACKGROUND_LOCATION`). |
| `FOREGROUND_SERVICE_CONNECTED_DEVICE` (Android 14+) | Required for BLE-only workouts and DFU | `BleForegroundService` and `DfuService` keep the BLE GATT connection alive while a workout or firmware update is in progress. Android 14 requires this dedicated foreground-service type for connected-device work. |
| `WAKE_LOCK` | Required | `LocationService` acquires a `PARTIAL_WAKE_LOCK` for the duration of the workout (cap: 12 hours) so the CPU stays awake to process incoming GPS fixes when the screen is off. Without it, Doze would batch fixes and the polyline would step rather than flow. |

### Notifications

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `POST_NOTIFICATIONS` (Android 13+) | Required for the foreground-service notification | Android 13 made notifications a runtime permission. Without it the foreground service still runs but the persistent notification is suppressed, which removes the user's only visible cue that workout tracking is active and gives the OS grounds to kill the service under memory pressure. |

### Engagement / Reminders

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `SCHEDULE_EXACT_ALARM` (Android 12+) | Optional | The SDK schedules engagement reminders (e.g. "you haven't worn your band in 3 days") via `AndroidScheduleMode.exactAllowWhileIdle`. Exact alarms fire reliably under Doze; inexact alarms can drift by hours, which makes "your morning workout reminder" unusable. |
| `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | Optional, strongly recommended | The SDK opens the system battery-optimization exemption screen so the user can whitelist the app. Without the exemption, OEM battery managers (Xiaomi, Huawei, Samsung "deep sleep") can suspend the foreground service mid-workout, dropping the polyline and disconnecting the band. |

### Network

| Permission | Required / Optional | Rationale |
|------------|---------------------|-----------|
| `INTERNET` | Required | The SDK posts workouts, syncs the user's profile, fetches firmware updates for the band, and exchanges OAuth tokens with Rolla's auth backend. None of these can be deferred — without internet the SDK runs in a read-only "no sync" mode. |

---

**Previous:** [Gradle Setup](02-gradle-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
