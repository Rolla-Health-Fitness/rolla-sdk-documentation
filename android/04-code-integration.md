# Code Integration

Integrate the Rolla SDK into your Android application with proper configuration, initialization, and listener implementation.

## Import SDK

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

## Create Configuration

```kotlin
val configuration = RollaConfiguration(
    token = "your-access-token",
    partnerId = "your-partner-id",
    refreshToken = "your-refresh-token",       // Optional
    tokenExpiresIn = 1800,                     // Optional: token expiry in seconds
    userId = "user-id",                        // Optional: extracted from JWT if not provided
    environment = "production",                // or "rnd" for development
    disabledModules = emptySet(),              // Optional: modules to hide (default: none disabled)
    branding = null                            // Optional: custom branding configuration
)
```

### Environment Values

| Value | Description |
|-------|-------------|
| `"production"` | Live / release builds |
| `"rnd"` | Development and QA (sandbox environment) |

If omitted, defaults to `"rnd"`. Use `"rnd"` during development and QA to test against a sandbox environment without affecting production data. Switch to `"production"` for release builds.

> **Why `"rnd"`?** The name stands for "Research and Development" — it is the SDK's internal label for the non-production sandbox environment.

## Initialize and Present SDK

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

## Implement RollaListener

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

## Fragment Support

The SDK can also be launched from a Fragment. The parameter expects `androidx.fragment.app.Fragment` (not the legacy `android.app.Fragment`):

```kotlin
rolla.show(fragment)  // fragment: androidx.fragment.app.Fragment
```

## Threading

All public SDK methods dispatch to the main thread internally using `Handler(Looper.getMainLooper())` — you can safely call them from any thread, including background threads or coroutine dispatchers:

| Method | Thread-safe | Notes |
|--------|:-----------:|-------|
| `show(activity)` | Yes | Posts to main handler before presenting |
| `dismiss()` | Yes | Posts to main handler before dismissing |
| `updateToken(...)` | Yes | Posts to main handler; callback fires on main thread |
| `clearSession(...)` | Yes | Posts to main handler; callback fires on main thread |

**Listener callbacks** also arrive on the main thread. The SDK explicitly wraps every listener invocation (`onRollaClosed`, `onRollaError`, `onTokenRefreshed`, `onTokenExpired`) in a `mainHandler.post { }` call. You can safely update your UI directly inside listener methods.

**Kotlin coroutines:** The SDK's public API uses callback-based patterns (not coroutines), but you can call SDK methods from any coroutine dispatcher without needing `withContext(Dispatchers.Main)` — the SDK handles the thread switch internally.

> **Summary:** You do not need to wrap any SDK call or listener handler in `runOnUiThread { }` — the SDK handles this for you.

## Cross-Platform Note: `tokenExpiresIn` Type

On Android, `tokenExpiresIn` is an `Int` (seconds). On iOS, it is a `TimeInterval` (`Double`, seconds).

If you maintain a shared backend or cross-platform token logic, be aware of this difference — passing a floating-point value where an integer is expected (or vice versa) can cause subtle bugs. Both platforms interpret the value as **seconds until expiry**.

The same applies to the `expiresIn` parameter in the `onTokenRefreshed` listener callback (`Int?` on Android, `TimeInterval?` on iOS).

---

**Previous:** [Permissions](03-permissions.md) | **Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
