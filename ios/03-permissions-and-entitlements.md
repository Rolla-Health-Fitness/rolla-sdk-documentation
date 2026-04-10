# Permissions & Entitlements

This section covers configuring permissions in `Info.plist` and entitlements in your `.entitlements` file.

## Configure Info.plist

Add the following permissions to your `Info.plist` file:

### Background Modes

```xml
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
    <string>bluetooth-central</string>
</array>
```

### Bluetooth Permissions

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth access allows the app to connect to your fitness band and keep syncing health data and activity metrics reliably.</string>

<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app uses Bluetooth to connect to your fitness band and sync health data and activity metrics.</string>
```

### Location Permissions

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app uses your location to track outdoor activities like running and cycling. With 'While Using the App', your route is recorded only when the app is open on screen.</string>

<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app uses your location to accurately track outdoor activities like running and cycling even when your phone is locked or the app is in the background.</string>
```

### Mapbox Token

The SDK uses Mapbox for map rendering (e.g., activity route maps). Add the Mapbox access token to your `Info.plist`:

```xml
<key>MBXAccessToken</key>
<string>your-mapbox-access-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

### Apple Health Permissions

```xml
<key>NSHealthShareUsageDescription</key>
<string>The app reads your Apple Health data — such as heart rate, steps, sleep, and workouts — to provide personalised health insights and keep your metrics in sync.</string>
```

> **Note:** The SDK reads Apple Health data only — it does not write to HealthKit. Only `NSHealthShareUsageDescription` is required.

## Configure Entitlements

### Bluetooth Central

Add the Bluetooth Central entitlement to your `.entitlements` file:

```xml
<key>com.apple.developer.bluetooth-central</key>
<true/>
```

### HealthKit (Required for Apple Health)

Add the HealthKit capability in Xcode:

1. Select your target in Xcode
2. Go to Signing & Capabilities tab
3. Click **+ Capability**
4. Add **HealthKit**

This adds to the entitlements file:

```xml
<key>com.apple.developer.healthkit</key>
<true/>
<key>com.apple.developer.healthkit.access</key>
<array/>
```

> **Note:** The HealthKit entitlement requires the app to be registered on an Apple Developer account.

---

**Previous:** [CocoaPods Setup](02-cocoapods-setup.md) | **Next:** [Code Integration](04-code-integration.md) | **Home:** [README](README.md)
