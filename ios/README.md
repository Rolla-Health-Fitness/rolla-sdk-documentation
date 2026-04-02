# Rolla SDK – iOS Integration Guide

Integration steps for embedding the Rolla SDK in an iOS app (CocoaPods).

> **See also:** [Live Activities Setup](live-activities.md) for real-time workout tracking on the Lock Screen and Dynamic Island.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [1. Add SDK via CocoaPods](#1-add-sdk-via-cocoapods)
- [2. Configure Info.plist](#2-configure-infoplist)
- [3. Configure Entitlements](#3-configure-entitlements)
- [4. Code Integration](#4-code-integration)
- [5. Configuration Options](#5-configuration-options)
- [6. Module Configuration](#6-module-configuration)
- [7. Apple Health Integration](#7-apple-health-integration)
- [8. Token Management](#8-token-management)
- [9. Engine Lifecycle](#9-engine-lifecycle)
- [10. Live Activities](#10-live-activities-ios-161)
- [11. Native API Reference](#11-native-api-reference)
- [12. Error Handling](#12-error-handling)
- [13. Close Reasons](#13-close-reasons)
- [14. Troubleshooting](#14-troubleshooting)
- [15. Support](#15-support)

---

## Prerequisites

- iOS 14.0 or later
- CocoaPods installed
- Xcode 14.0 or later
- Partner ID and API credentials from Rolla

---

## 1. Add SDK via CocoaPods

### 1.1 Update Podfile

Add the Rolla SDK specs repository and dependency to your Podfile:

```ruby
platform :ios, '14.0'

# Add Rolla SDK specs repository
source 'https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios.git'
source 'https://cdn.cocoapods.org/'

target 'YourApp' do
  use_frameworks!

  # Add Rolla SDK
  pod 'RollaSDK', '~> 0.1.5'
end
```

### 1.2 Set ENABLE_USER_SCRIPT_SANDBOXING to "No"

Disable `ENABLE_USER_SCRIPT_SANDBOXING`:

1. Open your project in Xcode.
2. In the Project Navigator, click the project (top blue icon).
3. Select your target.
4. Open the Build Settings tab.
5. In the search bar, type: `User Script Sandboxing`
6. You'll see "User Script Sandboxing" under "Build Options".
7. Double-click the value and change it to "No".

> **Why?** CocoaPods needs to create temporary files during framework copying, and sandboxing blocks that.

### 1.3 Install Dependencies

Run:

```bash
pod install
```

> **Important:** Always open your project using the `.xcworkspace` file, not `.xcodeproj`.

---

## 2. Configure Info.plist

Add the following permissions to your `Info.plist` file:

### 2.1 Background Modes

```xml
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
    <string>bluetooth-central</string>
</array>
```

### 2.2 Bluetooth Permissions

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Bluetooth access allows the app to connect to your fitness band and keep syncing health data and activity metrics reliably.</string>

<key>NSBluetoothPeripheralUsageDescription</key>
<string>The app uses Bluetooth to connect to your fitness band and sync health data and activity metrics.</string>
```

### 2.3 Location Permissions

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>The app uses your location to track outdoor activities like running and cycling. With 'While Using the App', your route is recorded only when the app is open on screen.</string>

<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>The app uses your location to accurately track outdoor activities like running and cycling even when your phone is locked or the app is in the background.</string>
```

### 2.4 Mapbox Token

The SDK uses Mapbox for map rendering (e.g., activity route maps). Add the Mapbox access token to your `Info.plist`:

```xml
<key>MBXAccessToken</key>
<string>your-mapbox-access-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

### 2.5 Apple Health Permissions

```xml
<key>NSHealthShareUsageDescription</key>
<string>The app reads your Apple Health data — such as heart rate, steps, sleep, and workouts — to provide personalised health insights and keep your metrics in sync.</string>
```

> **Note:** The SDK reads Apple Health data only — it does not write to HealthKit. Only `NSHealthShareUsageDescription` is required.

---

## 3. Configure Entitlements

### 3.1 Bluetooth Central

Add the Bluetooth Central entitlement to your `.entitlements` file:

```xml
<key>com.apple.developer.bluetooth-central</key>
<true/>
```

### 3.2 HealthKit (Required for Apple Health)

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

## 4. Code Integration

### 4.1 Import SDK

```swift
import RollaSDK
```

The SDK needs a **user access token** (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API **after** the user has logged in.

- **Typical flow:** User logs in the app → app calls backend → backend returns `access_token` (and optionally `refresh_token`, `expires_in`) → you pass that token into `RollaConfiguration` when opening the SDK.
- **When to fetch:** Before calling `rolla.show(from:)`. If the user is already logged in, use your existing session (e.g. stored token or refresh to get a new access token).
- **What to pass:** At minimum, the **access token** (string). For better behavior, also pass `refreshToken` and `tokenExpiresIn`.
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

> **Note:** You are responsible for authentication — the SDK only consumes the token you provide.

### 4.2 Create Configuration

```swift
let configuration = RollaConfiguration(
    token: "your-access-token",
    refreshToken: "your-refresh-token",  // Optional
    tokenExpiresIn: TimeInterval(3600),  // Optional: token expiry in seconds (TimeInterval)
    userId: "user-id",  // Optional: extracted from JWT if not provided
    partnerId: "your-partner-id",
    environment: "production",  // or "rnd" for development
    modules: nil,  // Optional: nil enables all modules
    branding: nil  // Optional: custom branding configuration
)
```

> **Note on `environment`:** The SDK supports two environments: `"production"` (live) and `"rnd"` (development/testing). Use `"rnd"` during development and QA to test against a sandbox environment without affecting production data. Switch to `"production"` for release builds. If omitted, the parameter defaults to `"rnd"`.

### 4.3 Initialize and Present SDK

```swift
let rolla = Rolla(configuration: configuration)
rolla.delegate = self
rolla.show(from: self)
```

### 4.4 Implement RollaDelegate

```swift
extension YourViewController: RollaDelegate {
    func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
        // Handle SDK close
        // Clean up any references
    }

    func rolla(_ rolla: Rolla, didFailWithError error: RollaError) {
        // Handle errors
        print("Rolla SDK error: \(error.localizedDescription)")
    }

    func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
        // Called when the SDK internally refreshes the token
        // Store the new token in your app's session for future use
    }

    func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
        // Called when the token has expired and the SDK cannot refresh it
        // You must fetch a new token from your backend and call:
        rolla.updateToken(token: newToken, refreshToken: newRefreshToken, expiresIn: newExpiresIn) { result in
            switch result {
            case .success:
                print("Token updated successfully")
            case .failure(let error):
                print("Failed to update token: \(error)")
            }
        }
    }
}
```

All delegate methods have default empty implementations, so you only need to implement the ones relevant to your use case.

---

## 5. Configuration Options

### 5.1 Custom Branding (Optional)

```swift
let branding = RollaBranding(
    appName: "Your App Name",
    primaryColor: .systemBlue,
    secondaryColor: .systemGray,
    accentColor: .systemGreen,
    brightness: "light",  // or "dark"
    defaultThemeMode: "system",  // "light", "dark", or "system"
    defaultLocale: "en",  // Optional
    headerLogoAsset: nil,  // Optional: partner logo asset path (provided by Rolla)
    termsUrl: "https://example.com/terms",  // Optional
    privacyUrl: "https://example.com/privacy"  // Optional
)

let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    branding: branding
)
```

### 5.2 Branding Assets

Image assets used by the SDK (such as partner logos referenced by `headerLogoAsset`) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. This means any custom logos, icons, or images need to be provided to Rolla in advance so they can be included in your SDK build.

During onboarding, coordinate with Rolla to supply:

- Your partner logo (SVG format preferred) for use in the app header and share cards
- Any other brand assets you want displayed within the SDK

Rolla will bundle these into the SDK and provide the correct asset path to use in your `RollaBranding` configuration.

---

## 6. Module Configuration

The SDK is organized into modules. Currently, all modules are always enabled — pass `nil` for the `modules` parameter (or omit it entirely).

```swift
let configuration = RollaConfiguration(
    token: token,
    partnerId: partnerId,
    modules: nil  // All modules enabled (currently the only supported option)
)
```

Selective module enablement will be available in a future release. Once you see the full feature set, you can tell us which modules you want enabled or disabled and we will implement per-partner module configuration.

See the [full modules table](../README.md#available-modules) for all 23 available modules and their descriptions.

---

## 7. Apple Health Integration

The SDK includes a full Apple Health integration. Partners do **not** need to write any HealthKit code — the SDK handles all reading automatically. However, your app must be configured correctly for HealthKit access.

### 7.1 Required Setup

Make sure you have completed:

1. **Info.plist entries** — `NSHealthShareUsageDescription` (see [Section 2.5](#25-apple-health-permissions) above)
2. **HealthKit capability** — Added via Xcode Signing & Capabilities (see [Section 3.2](#32-healthkit-required-for-apple-health) above)

No additional code is required. When the user opens the SDK and navigates to the integrations section, the SDK will prompt the user for Apple Health permissions automatically.

### 7.2 Supported Health Data Types

The SDK reads the following 14 HealthKit data types:

| Data Type | Read | Write |
|-----------|:----:|:-----:|
| Steps | Yes | No |
| Heart Rate | Yes | No |
| Heart Rate Variability (HRV) | Yes | No |
| Active Energy Burned | Yes | No |
| Sleep Analysis (with stages: awake, light, deep, REM) | Yes | No |
| Workouts | Yes | No |
| Workout Routes | Yes | No |
| Distance Walking/Running | Yes | No |
| Distance Cycling | Yes | No |
| Cycling Cadence (iOS 17+) | Yes | No |
| Cycling Power (iOS 17+) | Yes | No |
| Running Speed (iOS 16+) | Yes | No |
| Weight (maps to HealthKit Body Mass) | Yes | No |
| Blood Pressure (systolic + diastolic) | Yes | No |

> **Note:** Resting Heart Rate is available as a computed metric within the SDK's metrics module, but it is not read directly from HealthKit.

### 7.3 Apple Health Availability

Apple Health is part of the `integrations` module, which is enabled by default. The SDK gracefully handles devices without HealthKit support (e.g., iPads without the Health app) — on those devices, Apple Health features simply won't appear.

### 7.4 No Background Delivery

Apple Health data is read on-demand, not via background delivery in the current SDK implementation. No additional background modes are required beyond what is already configured (location + bluetooth-central).

### 7.5 Android Equivalent

There is currently **no** Android Health Connect integration in the SDK. On Android, health data comes from the Rolla band only. Do not expect feature parity between platforms for health data integrations.

---

## 8. Token Management

The SDK manages token lifecycle internally, but provides callbacks and methods so your app can stay in sync.

### 8.1 How It Works

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration`. On iOS, `tokenExpiresIn` is `TimeInterval?` (seconds as `Double`). On Android, it is `Int?`.
2. **Internal refresh:** When the token is close to expiring, the SDK attempts to refresh it automatically using the `refreshToken`. If successful, the SDK notifies your app via `rollaDidRefreshToken`.
3. **Expired token (cannot refresh):** If the SDK cannot refresh the token (e.g., the refresh token is also expired), it calls `rollaDidRequestTokenRefresh`. Your app must fetch a new token from your backend and push it to the SDK using `updateToken()`.
4. **Logout / session clear:** Call `clearSession()` when the user logs out to securely remove all SDK-persisted tokens and session data.

### 8.2 Delegate Callbacks

```swift
// Called when the SDK refreshes the token internally
func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
    // Store the new token in your session management
    SessionManager.shared.updateToken(token, refreshToken: refreshToken, expiresIn: expiresIn)
}

// Called when the SDK cannot refresh the token — your app must provide a new one
func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
    YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn in
        rolla.updateToken(token: newToken, refreshToken: newRefreshToken, expiresIn: TimeInterval(expiresIn)) { result in
            // Handle success/failure
        }
    }
}
```

### 8.3 Pushing a New Token

If you refresh tokens outside the SDK (e.g., during a background refresh in your app), you can push the new token to the SDK at any time:

```swift
rolla.updateToken(
    token: newAccessToken,
    refreshToken: newRefreshToken,  // Optional
    expiresIn: TimeInterval(3600)   // Optional: seconds until expiry (TimeInterval)
) { result in
    switch result {
    case .success:
        print("SDK token updated")
    case .failure(let error):
        print("Failed: \(error)")
    }
}
```

### 8.4 Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data:

```swift
rolla.clearSession { result in
    switch result {
    case .success:
        print("SDK session cleared")
    case .failure(let error):
        print("Failed to clear session: \(error)")
    }
}
```

---

## 9. Engine Lifecycle

The SDK uses a Flutter engine internally. Understanding its lifecycle helps you manage memory and user experience.

### 9.1 Default Behavior

- **First `show(from:)`** — Creates the Flutter engine and presents the SDK UI.
- **`dismiss()`** — Dismisses the SDK UI but **keeps the engine alive** in the background. The next `show(from:)` call will present the SDK instantly in its last state (no reload).
- This is the recommended behavior for most apps.

### 9.2 Destroying the Engine

If you need to free memory (e.g., on user logout or when the user won't return to the SDK for a while):

```swift
Rolla.destroyEngine()
```

- This fully tears down the Flutter engine and frees its resources.
- The next `show(from:)` call will create a fresh engine automatically (with a brief loading time).
- Call this **after** `dismiss()`, not while the SDK is presenting.

### 9.3 Recommended Usage

```swift
// User taps "Close" inside the SDK → rollaDidClose is called
func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
    // Engine stays alive — fast re-launch next time
}

// User logs out of your app
func logout() {
    rolla.clearSession { _ in }
    Rolla.destroyEngine()
}
```

---

## 10. Live Activities (iOS 16.1+)

The SDK supports iOS Live Activities for real-time workout tracking on the Lock Screen and Dynamic Island (iPhone 14 Pro and later).

**This is a substantial setup process.** See the dedicated guide: **[Live Activities Setup](live-activities.md)**

---

## 11. Native API Reference

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `init(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var delegate: RollaDelegate?` | Set the delegate for callbacks |
| `var isPresenting: Bool` | Whether the SDK is currently visible |
| `show(from: UIViewController)` | Present the SDK modally |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token:refreshToken:expiresIn:completion:)` | Push fresh credentials to the SDK |
| `clearSession(completion:)` | Clear all persisted session data |
| `static destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaDelegate Protocol

| Method | Description |
|--------|-------------|
| `rollaDidClose(_:reason:)` | Called when the SDK UI is dismissed |
| `rolla(_:didFailWithError:)` | Called when an error occurs |
| `rollaDidRefreshToken(_:token:refreshToken:expiresIn:)` | Called when the SDK refreshes tokens internally |
| `rollaDidRequestTokenRefresh(_:)` | Called when the host app must provide new tokens |

All methods have default empty implementations.

---

## 12. Error Handling

The SDK provides detailed error information through `RollaError`:

```swift
public enum RollaError: Error {
    case engineFailedToStart
    case initializationFailed(String)
    case flutterError(code: String, message: String)
    case alreadyPresenting
    case invalidPresentationContext
    case underlying(Error)
    case unknown
}
```

---

## 13. Close Reasons

The SDK provides close reasons through `RollaCloseReason`:

```swift
enum RollaCloseReason {
    case flutterRequested(reason: String?)
    case hostNavigationBack
    case hostModalDismiss
    case programmatic
    case hostStackReplaced
    case unknown
}
```

---

## 14. Troubleshooting

### SDK fails to start

- Ensure all Info.plist permissions are configured
- Verify Bluetooth Central capability is enabled
- Check that the token is valid and not expired
- Verify HealthKit capability is enabled (if using Apple Health)

### Apple Health not showing

- Verify `NSHealthShareUsageDescription` is in Info.plist
- Verify HealthKit capability is added in Xcode (Signing & Capabilities)
- Ensure `"integrations"` module is enabled (or modules is `nil`)
- HealthKit is not available on iPads without the Health app

### Build errors

- Clean build folder: Product > Clean Build Folder (Shift-Command-K)
- Delete Pods folder and Podfile.lock
- Run `pod install` again
- Ensure you're opening `.xcworkspace`, not `.xcodeproj`

---

## 15. Support

For issues or questions, contact Rolla support or refer to the SDK documentation.
