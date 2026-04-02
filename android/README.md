# Rolla SDK – Android Integration Guide

Integration steps for embedding the Rolla SDK in an Android app (Gradle + Maven).

> **See also:** [iOS Integration Guide](../ios/README.md) | [Overview](../README.md)

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Add SDK via Gradle](#1-add-sdk-via-gradle)
   - [1.1 Add Maven Repositories](#11-add-maven-repositories)
   - [1.2 Add SDK Dependency](#12-add-sdk-dependency)
   - [1.3 Sync Project](#13-sync-project)
3. [Configure AndroidManifest.xml](#2-configure-androidmanifestxml)
   - [2.1 Internet Permission](#21-internet-permission)
   - [2.2 Mapbox Token](#22-mapbox-token)
4. [Code Integration](#3-code-integration)
   - [3.1 Import SDK](#31-import-sdk)
   - [3.2 Create Configuration](#32-create-configuration)
   - [3.3 Initialize and Present SDK](#33-initialize-and-present-sdk)
   - [3.4 Implement RollaListener](#34-implement-rollalistener)
   - [3.5 Fragment Support](#35-fragment-support)
5. [Configuration Options](#4-configuration-options)
   - [4.1 Custom Branding (Optional)](#41-custom-branding-optional)
6. [Module Configuration](#5-module-configuration)
7. [Token Management](#6-token-management)
   - [6.1 How It Works](#61-how-it-works)
   - [6.2 Listener Callbacks](#62-listener-callbacks)
   - [6.3 Pushing a New Token](#63-pushing-a-new-token)
   - [6.4 Clearing the Session](#64-clearing-the-session)
8. [Error Handling](#7-error-handling)
9. [Close Reasons](#8-close-reasons)
10. [Programmatic Dismiss](#9-programmatic-dismiss)
11. [Engine Lifecycle](#10-engine-lifecycle)
    - [10.1 Default Behavior](#101-default-behavior)
    - [10.2 Destroying the Engine](#102-destroying-the-engine)
    - [10.3 Recommended Usage](#103-recommended-usage)
12. [Native API Reference](#11-native-api-reference-android)
13. [Troubleshooting](#12-troubleshooting)
14. [Support](#13-support)

---

## Prerequisites

- Android API 24 (Android 7.0) or later
- Android Studio Hedgehog (2023.1) or later
- Gradle 8.0+
- Partner ID and API credentials from Rolla

## 1. Add SDK via Gradle

### 1.1 Add Maven Repositories

Add the required Maven repositories to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()

        // Rolla SDK repository
        maven {
            url = uri("https://rolla-health-fitness.github.io/rolla-sdk-release-android/maven/")
        }

        // Flutter engine artifacts
        maven {
            url = uri("https://storage.googleapis.com/download.flutter.io")
        }

        // Mapbox SDK for maps
        maven {
            url = uri("https://api.mapbox.com/downloads/v2/releases/maven")
        }
    }
}
```

### 1.2 Add SDK Dependency

Add the Rolla SDK dependency to your `app/build.gradle.kts`:

```kotlin
dependencies {
    // Core library desugaring (required by SDK)
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")

    // Rolla SDK
    implementation("com.rolla.sdk:android_release:0.1.5")
}
```

Enable core library desugaring in the android block:

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}
```

### 1.3 Sync Project

Click "Sync Now" in the Gradle notification bar, or run:

```
./gradlew --refresh-dependencies
```

## 2. Configure AndroidManifest.xml

### 2.1 Internet Permission

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### 2.2 Mapbox Token

Add the Mapbox access token to `app/src/main/res/values/strings.xml` for map functionality:

```xml
<string name="mapbox_access_token">your-mapbox-token</string>
```

You will receive the Mapbox token from Rolla along with your partner credentials.

> **Note:** Bluetooth, location, and foreground service permissions are declared by the SDK and merged automatically via manifest merger. You do not need to add them manually.

## 3. Code Integration

### 3.1 Import SDK

```kotlin
import com.rolla.sdk.wrapper.Rolla
import com.rolla.sdk.wrapper.RollaConfiguration
import com.rolla.sdk.wrapper.RollaListener
import com.rolla.sdk.wrapper.RollaCloseReason
import com.rolla.sdk.wrapper.RollaError
```

The SDK needs a user access token (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API after the user has logged in.

- **Typical flow:** User logs in the app → app calls backend → backend returns **access_token** (and optionally **refresh_token**, **expires_in**) → you pass that token into `RollaConfiguration` when opening the SDK.
- **When to fetch:** Before calling `rolla.show(activity)`. If the user is already logged in, use your existing session (e.g. stored token or refresh to get a new access token).
- **What to pass:** At minimum, the **access token** (string). For better behavior, also pass `refreshToken` and `tokenExpiresIn`.
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

Note: You are responsible for authentication, the SDK only consumes the token you provide.

### 3.2 Create Configuration

```kotlin
val configuration = RollaConfiguration(
    token = "your-access-token",
    partnerId = "your-partner-id",
    refreshToken = "your-refresh-token",       // Optional
    tokenExpiresIn = 3600,                     // Optional: token expiry in seconds
    userId = "user-id",                        // Optional: extracted from JWT if not provided
    environment = "production",                // or "rnd" for development
    modules = null,                            // Optional: null enables all modules
    branding = null                            // Optional: custom branding configuration
)
```

> **Note on `environment`:** The SDK supports two environments: `"production"` (live) and `"rnd"` (development/testing). Use `"rnd"` during development and QA to test against a sandbox environment without affecting production data. Switch to `"production"` for release builds. If omitted, the parameter defaults to `"rnd"`.

### 3.3 Initialize and Present SDK

```kotlin
val rolla = Rolla(configuration)
rolla.listener = object : RollaListener {
    override fun onRollaClosed(rolla: Rolla, reason: RollaCloseReason) {
        // Handle SDK close
    }

    override fun onRollaError(rolla: Rolla, error: RollaError) {
        // Handle errors
        Log.e("RollaSDK", "Error: ${error.message}")
    }
}
rolla.show(activity)
```

### 3.4 Implement RollaListener

```kotlin
rolla.listener = object : RollaListener {
    override fun onRollaClosed(rolla: Rolla, reason: RollaCloseReason) {
        // Handle SDK close
        // Clean up any references
        this@YourActivity.rolla = null
    }

    override fun onRollaError(rolla: Rolla, error: RollaError) {
        // Handle errors
        Log.e("RollaSDK", "Error: ${error.message}")
        Toast.makeText(this@YourActivity, error.message, Toast.LENGTH_LONG).show()
    }

    override fun onTokenRefreshed(rolla: Rolla, token: String, refreshToken: String?, expiresIn: Int?) {
        // Called when the SDK internally refreshes the token
        // Store the new token in your app's session for future use
        SessionManager.updateToken(token, refreshToken, expiresIn)
    }

    override fun onTokenExpired(rolla: Rolla) {
        // Called when the token has expired and the SDK cannot refresh it
        // You must fetch a new token from your backend and call updateToken()
        YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn ->
            rolla.updateToken(newToken, newRefreshToken, expiresIn) { result ->
                result.onSuccess { Log.d("RollaSDK", "Token updated") }
                result.onFailure { Log.e("RollaSDK", "Token update failed: $it") }
            }
        }
    }
}
```

All listener methods have default empty implementations, so you only need to override the ones relevant to your use case.

### 3.5 Fragment Support

The SDK can also be launched from a Fragment:

```kotlin
rolla.show(fragment)
```

## 4. Configuration Options

### 4.1 Custom Branding (Optional)

Colors are passed as ARGB integers (e.g. `0xFF6750A4.toInt()`):

```kotlin
val branding = RollaBranding(
    appName = "Your App Name",
    primaryColor = 0xFF1976D2.toInt(),         // Blue
    secondaryColor = 0xFF625B71.toInt(),       // Gray
    accentColor = 0xFF7D5260.toInt(),          // Accent
    brightness = "light",                       // or "dark"
    defaultThemeMode = "system",               // "light", "dark", or "system"
    defaultLocale = "en",                      // Optional
    headerLogoAsset = null,                    // Optional: partner logo asset path (provided by Rolla)
    termsUrl = "https://example.com/terms",    // Optional
    privacyUrl = "https://example.com/privacy" // Optional
)

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    branding = branding
)
```

> **Branding assets:** Image assets (such as partner logos) must be pre-bundled inside the SDK — they cannot be transferred from the host app at runtime. See the [iOS Integration Guide (5.2 Branding Assets)](../ios/README.md) for details on coordinating with Rolla.

## 5. Module Configuration

The SDK is organized into modules. Currently, all modules are always enabled — pass `null` for the `modules` parameter (or omit it entirely).

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    modules = null  // All modules enabled (currently the only supported option)
)
```

Selective module enablement will be available in a future release. See the [iOS Integration Guide modules table](../ios/README.md) for the full modules table with descriptions.

## 6. Token Management

The SDK manages token lifecycle internally, but provides callbacks and methods so your app can stay in sync.

### 6.1 How It Works

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration`.
2. **Internal refresh:** When the token is close to expiring, the SDK attempts to refresh it automatically. If successful, the SDK notifies your app via `onTokenRefreshed`.
3. **Expired token (cannot refresh):** If the SDK cannot refresh the token, it calls `onTokenExpired`. Your app must fetch a new token from your backend and push it to the SDK using `updateToken()`.
4. **Logout / session clear:** Call `clearSession()` when the user logs out to securely remove all SDK-persisted tokens and session data.

### 6.2 Listener Callbacks

```kotlin
override fun onTokenRefreshed(rolla: Rolla, token: String, refreshToken: String?, expiresIn: Int?) {
    // Store the new token in your session management
    SessionManager.updateToken(token, refreshToken, expiresIn)
}

override fun onTokenExpired(rolla: Rolla) {
    // Fetch a new token from your backend
    YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn ->
        rolla.updateToken(newToken, newRefreshToken, expiresIn) { result ->
            // Handle success/failure
        }
    }
}
```

### 6.3 Pushing a New Token

If you refresh tokens outside the SDK (e.g., during a background refresh in your app), you can push the new token to the SDK at any time:

```kotlin
rolla.updateToken(
    token = newAccessToken,
    refreshToken = newRefreshToken,  // Optional
    expiresIn = 3600,                // Optional: seconds until expiry
    callback = { result ->
        result.onSuccess { Log.d("RollaSDK", "Token updated") }
        result.onFailure { Log.e("RollaSDK", "Token update failed: $it") }
    }
)
```

### 6.4 Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data:

```kotlin
rolla.clearSession { result ->
    result.onSuccess { Log.d("RollaSDK", "Session cleared") }
    result.onFailure { Log.e("RollaSDK", "Failed to clear: $it") }
}
```

## 7. Error Handling

The SDK provides detailed error information through RollaError:

```kotlin
sealed class RollaError(val code: String, val message: String) : Exception(message) {
    class EngineFailedToStart
    class InitializationFailed(details: String)
    class FlutterError(errorCode: String, errorMessage: String)
    class AlreadyPresenting
    class InvalidContext
    class Underlying(throwable: Throwable)
    class Unknown(details: String?)
}
```

## 8. Close Reasons

The SDK provides close reasons through RollaCloseReason:

```kotlin
sealed class RollaCloseReason {
    data class FlutterRequested(val reason: String?)
    data object HostNavigationBack
    data object HostModalDismiss
    data object Programmatic
    data object HostStackReplaced
    data object Unknown
}
```

## 9. Programmatic Dismiss

You can dismiss the SDK programmatically:

```kotlin
rolla.dismiss()
```

## 10. Engine Lifecycle

The SDK manages a Flutter engine internally. By default, the engine stays alive for faster re-launch.

### 10.1 Default Behavior

- **First `show(activity)`** — Creates the Flutter engine and presents the SDK UI.
- **`dismiss()`** — Dismisses the SDK UI but keeps the engine alive. The next `show()` will present the SDK instantly in its last state.

### 10.2 Destroying the Engine

If you need to free memory (e.g., on logout), call:

```kotlin
Rolla.destroyEngine()
```

The engine will be recreated automatically on the next `show()` call.

### 10.3 Recommended Usage

```kotlin
// User logs out
fun logout() {
    rolla.clearSession { }
    Rolla.destroyEngine()
}
```

## 11. Native API Reference (Android)

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `Rolla(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var listener: RollaListener?` | Set the listener for callbacks |
| `val isPresenting: Boolean` | Whether the SDK is currently visible |
| `show(activity: Activity)` | Present the SDK from an Activity |
| `show(fragment: Fragment)` | Present the SDK from a Fragment |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token, refreshToken?, expiresIn?, callback?)` | Push fresh credentials to the SDK |
| `clearSession(callback?)` | Clear all persisted session data |
| `companion fun destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaListener Interface

| Method | Description |
|--------|-------------|
| `onRollaClosed(rolla, reason)` | Called when the SDK UI is dismissed |
| `onRollaError(rolla, error)` | Called when an error occurs |
| `onTokenRefreshed(rolla, token, refreshToken?, expiresIn?)` | Called when the SDK refreshes tokens internally |
| `onTokenExpired(rolla)` | Called when the host app must provide new tokens |

All methods have default empty implementations.

## 12. Troubleshooting

### SDK fails to start

- Ensure internet permission is in your manifest
- Check that the access token is valid and not expired
- Verify the partner ID is correct
- Check Logcat for errors with tag `RollaEngineManager` or `RollaSdkPlugin`

### Maps not showing

- Verify the Mapbox token is added to AndroidManifest.xml (or strings.xml)
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

## 13. Support

For issues or questions, contact Rolla support or refer to the SDK documentation.
