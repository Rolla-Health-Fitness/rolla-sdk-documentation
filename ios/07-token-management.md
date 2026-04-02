# Token Management

The SDK manages token lifecycle internally, but provides callbacks and methods so your app can stay in sync with authentication state.

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

**Previous:** [Apple Health Integration](06-apple-health.md) | **Next:** [Engine Lifecycle](08-engine-lifecycle.md) | **Home:** [README](README.md)
