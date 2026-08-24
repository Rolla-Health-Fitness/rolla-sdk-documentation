# Token Management

The SDK manages the token lifecycle by itself. However, the refresh tokens issued by the Rolla auth API are single-use, so your app still carries a small set of obligations that will keep the session healthy beyond the first access-token expiry.

## How It Works

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration` — always the newest pair your app has stored (see [Your App's Responsibilities](#your-apps-responsibilities)).
2. **Internal refresh:** The SDK refreshes the access token automatically — proactively, shortly before the token expires (based on `tokenExpiresIn`), and reactively, when a request receives HTTP 401. After every successful internal refresh while the SDK UI is presented, the SDK hands the **new token pair** to your app via `onTokenRefreshed`.
3. **Expired session (SDK cannot refresh):** If the internal refresh fails (typically because the refresh token was consumed outside the SDK, or has expired), the SDK calls `onTokenExpired` and holds the failing request for up to 10 seconds while you answer. Obtain a fresh token pair from the Rolla auth API — by re-authenticating via [`/api/login`](../sdk-auth-api/02-authentication.md#log-in), directly or through your backend — and push it with `updateToken()`: a push inside that 10s window retries the request with the new tokens, and the user sees no error. If the window passes unanswered — or answered only with a pair older than the one the SDK already holds, which is ignored (see [responsibility 4](#your-apps-responsibilities)) — screens that request backend data show an error state and recover on their next load after the tokens arrive.

   > **Avoiding this state:** pass the refresh token to the SDK, but never spend it yourself — let the SDK do all refreshing.
4. **Logout / session clear:** Call `clearSession()` when the user logs out. It removes all SDK-persisted tokens and session data from secure storage.

## Token Facts

| Fact | Value |
|------|-------|
| Access token lifetime | 30 minutes (`expires_in: 1800` in auth responses) |
| Refresh token lifetime | 30 days (`refresh_expires_in: 2592000`) |
| Refresh token reuse | **Single-use.** Every successful [`/api/refresh_token`](../sdk-auth-api/02-authentication.md#refresh-token) call returns a *new* refresh token and permanently invalidates the one that was just used — regardless of whether the SDK or your own code made the call. |

## Your App's Responsibilities

All of the following should be implemented.

1. **Pass all three token fields at initialization.** `token` alone is enough to open the SDK, but without `refreshToken` the SDK cannot refresh at all on its own and every expiry escalates to `onTokenExpired`; without `tokenExpiresIn` the SDK cannot refresh proactively and only recovers after the first 401.
2. **Implement `onTokenRefreshed` and persist the delivered pair.** The SDK's internal refresh rotates the tokens, so the pair your app stored earlier is stale from that moment on. Overwrite it with the delivered pair, which is now the only pair that can refresh.
3. **Implement `onTokenExpired` and answer it with `updateToken()`.** This is the recovery path when the SDK cannot help itself. Obtain a fresh pair from the Rolla auth API ([`/api/login`](../sdk-auth-api/02-authentication.md#log-in)), directly or through your backend, and push it with `updateToken()`. The session then recovers in place, without the user leaving the SDK.
4. **Always initialize with the newest pair you have.** The SDK compares the tokens you pass against the pair it already holds (through the tokens' own JWT claims) and **ignores anything older** — so a re-sent original login pair can no longer overwrite a newer pair the SDK obtained by rotation. Initializing with the newest persisted pair still matters in the other direction: when your app re-authenticates, the fresh pair outranks the SDK's stored one and is what re-arms the session.
   - Note: if you pass `refreshToken = null` on a re-initialization, the SDK keeps the refresh token it already holds; an access token older than the stored one is ignored entirely.
5. **Keep the SDK's refresh token exclusive to the SDK.** If your backend uses the same refresh token for its own session refresh, whichever side refreshes first invalidates the token for the other. Issue your own session credentials separately, or route all refreshes through a single owner.
6. **Call `clearSession()` on logout** so the next user cannot inherit tokens or data from the previous user.

## Listener Callbacks

Both callbacks are part of `RollaListener` (attach it via `rolla.listener = ...` before showing the SDK):

```kotlin
override fun onTokenRefreshed(rolla: Rolla, token: String, refreshToken: String?, expiresIn: Int?) {
    // The SDK rotated the tokens — persist this pair; it replaces every
    // previously stored pair, including the original login response.
    SessionManager.updateToken(token, refreshToken, expiresIn)
}

override fun onTokenExpired(rolla: Rolla) {
    // The SDK could not refresh on its own. Obtain fresh tokens from the
    // Rolla auth API (/api/login), directly or through your backend, and
    // hand them over — the session recovers in place.
    YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn ->
        rolla.updateToken(newToken, newRefreshToken, expiresIn) { result ->
            // Handle success/failure
        }
    }
}
```

## Pushing a New Token

If you refresh tokens outside the SDK (e.g. during a background refresh in your app), push the new pair to the SDK at any time — you don't need to wait for a callback. Two caveats: `updateToken()` needs a running engine (any prior `show()`, `openScreen`, `warmUpEngine()`, or headless call — before that, simply initialize with the newest pair instead), and a success result means the push was delivered, not that it was applied — a pair older than the one the SDK already holds is ignored by design (see [responsibility 4](#your-apps-responsibilities)) and still resolves as success:

```kotlin
rolla.updateToken(
    token = newAccessToken,
    refreshToken = newRefreshToken,  // Optional: null keeps the SDK's stored refresh token (if it exists)
    expiresIn = 1800,                // Optional: seconds until expiry
    callback = { result ->
        result.onSuccess { Log.d("RollaSDK", "Token updated") }
        result.onFailure { Log.e("RollaSDK", "Token update failed: $it") }
    }
)
```

## Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data. Like `updateToken()`, it needs a running engine — on a cold engine the call fails and the stored tokens remain, so if the user logs out before the engine has started in this app session, call `warmUpEngine()` first and clear from its completion:

```kotlin
rolla.clearSession { result ->
    result.onSuccess { Log.d("RollaSDK", "Session cleared") }
    result.onFailure { Log.e("RollaSDK", "Failed to clear: $it") }
}
```

---

**Previous:** [Configuration](05-configuration.md) | **Next:** [Engine Lifecycle](07-engine-lifecycle.md) | **Home:** [README](README.md)
