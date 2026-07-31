# Code Integration

Integrate the Rolla SDK into your Android application with proper configuration, initialization, and listener implementation.

## Import SDK

```kotlin
import com.rolla.sdk.wrapper.Rolla
import com.rolla.sdk.wrapper.RollaListener
import com.rolla.sdk.wrapper.config.RollaConfiguration
import com.rolla.sdk.wrapper.features.session.RollaCloseReason
import com.rolla.sdk.wrapper.features.session.RollaError
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
    environment = "production"                 // or "rnd" for development
)
```

These are the identity and auth essentials. `RollaConfiguration` also takes `branding`, `language`, `disabledModules`, `disabledDataSources`, `userId`, and `showSettingsButton` — see [Configuration](05-configuration.md) for the full reference.

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

The four above are the presentation and token callbacks — the minimum for a production integration. `RollaListener` has twelve more optional methods that push SDK events to your app (activity lifecycle, sync results, band pairing and live link state, primary source, goals, profile) — see the [RollaListener reference](08-api-reference.md#rollalistener-interface) and its [Host Events](08-api-reference.md#host-events) delivery semantics.

## Fragment Support

The SDK can also be launched from a Fragment. The parameter expects `androidx.fragment.app.Fragment` (not the legacy `android.app.Fragment`):

```kotlin
rolla.show(fragment)  // fragment: androidx.fragment.app.Fragment
```

## Threading

The SDK is thread-safe at its public surface, so you never have to think about threads when calling it:

- **Every method on a `Rolla` instance** — presentation, token, and headless methods alike — dispatches to the main thread internally via `Handler(Looper.getMainLooper())`. Call them from any thread, including background threads or coroutine dispatchers.
- **Every callback** — all `RollaListener` methods and every result callback — arrives on the main thread. You can update your UI directly inside them.
- **The one exception is the companion `Rolla.destroyEngine()`**: it runs synchronously on the calling thread, with no internal dispatch — call it from the main thread.

**Kotlin coroutines:** The SDK's public API is callback-based (not coroutines), but you can call SDK methods from any coroutine dispatcher without `withContext(Dispatchers.Main)` — the SDK handles the thread switch internally.

> **Summary:** You never need `runOnUiThread { }` around an SDK call or listener handler.

## Cross-Platform Note: `tokenExpiresIn` Type

Both platforms mean the same thing — **seconds until expiry** — but the declared type follows each platform's idiom: `Int?` on Android, `TimeInterval?` (a `Double` number of seconds) on iOS. The same applies to the `expiresIn` parameter in the `onTokenRefreshed` callback. There is no unit difference: if your backend returns `expires_in` in seconds, pass it straight through on both platforms.

---

**Previous:** [Permissions](03-permissions.md) | **Next:** [Configuration](05-configuration.md) | **Home:** [README](README.md)
