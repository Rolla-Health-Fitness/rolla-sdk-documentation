# Token Management

Learn how the SDK manages token lifecycle and how your app can stay in sync with token state through callbacks and API methods.

## How It Works

The SDK manages token lifecycle internally, but provides callbacks and methods so your app can stay in sync.

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration`.
2. **Internal refresh:** When the token is close to expiring, the SDK attempts to refresh it automatically. If successful, the SDK notifies your app via `onTokenRefreshed`.
3. **Expired token (cannot refresh):** If the SDK cannot refresh the token, it calls `onTokenExpired`. Your app must fetch a new token from your backend and push it to the SDK using `updateToken()`.
4. **Logout / session clear:** Call `clearSession()` when the user logs out to securely remove all SDK-persisted tokens and session data.

## Listener Callbacks

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

## Pushing a New Token

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

## Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data:

```kotlin
rolla.clearSession { result ->
    result.onSuccess { Log.d("RollaSDK", "Session cleared") }
    result.onFailure { Log.e("RollaSDK", "Failed to clear: $it") }
}
```

---

**Previous:** [Branding & Modules](05-branding-and-modules.md) | **Next:** [Engine Lifecycle](07-engine-lifecycle.md) | **Home:** [README](README.md)
