# Code Integration

Integrate the Rolla SDK into your Android application with proper configuration, initialization, and listener implementation.

## 3.1 Import SDK

```kotlin
import com.rolla.sdk.wrapper.Rolla
import com.rolla.sdk.wrapper.RollaConfiguration
import com.rolla.sdk.wrapper.RollaListener
import com.rolla.sdk.wrapper.RollaCloseReason
import com.rolla.sdk.wrapper.RollaError
```

## Authentication & Token Flow

The SDK needs a user access token (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API after the user has logged in.

- **Typical flow:** User logs in the app → app calls backend → backend returns **access_token** (and optionally **refresh_token**, **expires_in**) → you pass that token into `RollaConfiguration` when opening the SDK.
- **When to fetch:** Before calling `rolla.show(activity)`. If the user is already logged in, use your existing session (e.g. stored token or refresh to get a new access token).
- **What to pass:** At minimum, the **access token** (string). For better behavior, also pass `refreshToken` and `tokenExpiresIn`.
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

Note: You are responsible for authentication, the SDK only consumes the token you provide.

## 3.2 Create Configuration

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

## 3.3 Initialize and Present SDK

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

## 3.4 Implement RollaListener

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

## 3.5 Fragment Support

The SDK can also be launched from a Fragment:

```kotlin
rolla.show(fragment)
```

---

**Previous:** [Permissions](03-permissions.md) | **Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
