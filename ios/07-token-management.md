# Token Management

The SDK manages the token lifecycle by itself. However, the refresh tokens issued by the Rolla auth API are single-use, so your app still carries a small set of obligations — they are what keeps the session healthy beyond the first access-token expiry. This page describes exactly what the SDK handles on its own and what your app **must** implement.

## How It Works

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` in `RollaConfiguration` — always the newest pair your app has stored (see [Your App's Responsibilities](#your-apps-responsibilities)). On iOS, `tokenExpiresIn` is `TimeInterval?` (seconds as `Double`); on Android, it is `Int?`.
2. **Internal refresh:** The SDK refreshes the access token automatically — proactively, shortly before the token expires (based on `tokenExpiresIn`), and reactively, when a request receives HTTP 401. In the reactive case, the failed request is retried with the new token, invisibly to the user. After every successful internal refresh while the SDK UI is presented, the SDK hands the **new token pair** to your app via `rollaDidRefreshToken`. A rotation that happens with the UI closed (e.g. during a headless sync) is persisted inside the SDK but not delivered — harmless: the staleness guard (responsibility 4) means an older pair your app re-sends can never overwrite it.
3. **Expired session (SDK cannot refresh):** If the internal refresh fails (typically because the refresh token was consumed outside the SDK, or has expired), the SDK calls `rollaDidRequestTokenRefresh` and holds the failing request for up to 10 seconds while you answer. Obtain a fresh token pair from the Rolla auth API — by re-authenticating via [`/api/login`](../sdk-auth-api/02-authentication.md#log-in), directly or through your backend — and push it with `updateToken()`: a push inside that window retries the request with the new tokens, and the user sees no error. If the window passes unanswered, screens that request backend data show an error state and recover on their next load after the tokens arrive.

   > **Avoiding this state:** pass the refresh token to the SDK, but never spend it yourself — let the SDK do all refreshing. An integration that keeps the refresh token exclusive to the SDK and persists every rotated pair (responsibilities 2 and 5) reaches this state only when the refresh token expires after 30 days without a refresh — a state that requires the user to log in again anyway.
4. **Logout / session clear:** Call `clearSession()` when the user logs out; it removes all SDK-persisted tokens and session data from secure storage.

## Token Facts

| Fact | Value |
|------|-------|
| Access token lifetime | 30 minutes (`expires_in: 1800` in auth responses — always use the returned value rather than hardcoding it) |
| Refresh token lifetime | 30 days (`refresh_expires_in: 2592000`) |
| Refresh token reuse | **Single-use.** Every successful [`/api/refresh_token`](../sdk-auth-api/02-authentication.md#refresh-token) call returns a *new* refresh token and permanently invalidates the one that was used — regardless of whether the SDK or your own code made the call. |

## Your App's Responsibilities

All of the following are required. An integration that skips any of them usually appears to work — until the access token expires for the first time, about 30 minutes after it was issued (see [Symptoms](#symptoms-of-incorrect-token-wiring)).

1. **Pass all three token fields at initialization.** `token` alone is enough to open the SDK, but without `refreshToken` the SDK cannot refresh at all on its own and every expiry escalates to `rollaDidRequestTokenRefresh`; without `tokenExpiresIn` the SDK cannot refresh proactively and only recovers after the first 401.
2. **Implement `rollaDidRefreshToken` and persist the delivered pair.** The SDK's internal refresh rotates the tokens, so the pair your app stored earlier is stale from that moment on. Overwrite it with the delivered pair, which is now the only pair that can refresh.
3. **Implement `rollaDidRequestTokenRefresh` and answer it with `updateToken()`.** This is the recovery path when the SDK cannot help itself. Obtain a fresh pair from the Rolla auth API ([`/api/login`](../sdk-auth-api/02-authentication.md#log-in)), directly or through your backend, and push it with `updateToken()`. The session then recovers in place, without the user leaving the SDK.
4. **Always initialize with the newest pair you have.** The SDK compares the tokens you pass against the pair it already holds (through the tokens' own JWT claims) and **ignores anything older** — so a re-sent original login pair can no longer overwrite a newer pair the SDK obtained by rotation. Initializing with the newest persisted pair still matters in the other direction: when your app re-authenticates, the fresh pair outranks the SDK's stored one and is what re-arms the session. Initialize with what responsibility 2 persisted, never with a cached login response.
   - Note: if you pass `refreshToken: nil` on a re-initialization, the SDK keeps the refresh token it already holds; an access token older than the stored one is ignored entirely.
5. **Keep the SDK's refresh token exclusive to the SDK.** If your backend uses the same refresh token for its own session refresh, whichever side refreshes first invalidates the token for the other. Issue your own session credentials separately, or route all refreshes through a single owner.
6. **Call `clearSession()` on logout** so the next user cannot inherit tokens or data from the previous one.

## Delegate Callbacks

Both callbacks are part of `RollaDelegate` (attach it via `rolla.delegate = ...` before showing the SDK):

```swift
// Called when the SDK refreshes the token internally
func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
    // The SDK rotated the tokens — persist this pair; it replaces every
    // previously stored pair, including the original login response.
    SessionManager.shared.updateToken(token, refreshToken: refreshToken, expiresIn: expiresIn)
}

// Called when the SDK cannot refresh the token — your app must provide a new one
func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
    // Obtain fresh tokens from the Rolla auth API (/api/login), directly or
    // through your backend, and hand them over — the session recovers in place.
    YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn in
        rolla.updateToken(token: newToken, refreshToken: newRefreshToken, expiresIn: TimeInterval(expiresIn)) { result in
            // Handle success/failure
        }
    }
}
```

## Pushing a New Token

If you refresh tokens outside the SDK (e.g. during a background refresh in your app), push the new pair to the SDK at any time — you don't need to wait for a callback. Two caveats: `updateToken()` needs a running engine (any prior `show(from:)`, `openScreen`, `warmUpEngine()`, or headless call — before that, simply initialize with the newest pair instead), and a success result means the push was delivered, not that it was applied — a pair older than the one the SDK already holds is ignored by design (see [responsibility 4](#your-apps-responsibilities)) and still resolves as success:

```swift
rolla.updateToken(
    token: newAccessToken,
    refreshToken: newRefreshToken,  // Optional: nil keeps the SDK's stored refresh token
    expiresIn: TimeInterval(1800)   // Optional: seconds until expiry (TimeInterval)
) { result in
    switch result {
    case .success:
        print("SDK token updated")
    case .failure(let error):
        print("Failed: \(error)")
    }
}
```

## Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data. Like `updateToken()`, it needs a running engine — on a cold engine the call fails and the stored tokens remain, so if the user logs out before the engine has started in this app session, call `warmUpEngine()` first and clear from its completion:

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

## Symptoms of Incorrect Token Wiring

| Symptom | Cause | Fix |
|---------|-------|-----|
| Works at first, but after ~30 minutes SDK screens show `Access denied (HTTP 401)`; exiting and re-entering the SDK fixes it — until it recurs | The SDK's refresh token was consumed outside the SDK (or never passed), and `rollaDidRequestTokenRefresh` goes unanswered. Re-entry only masks the problem: your app passes a fresh access token at initialization, which works for another 30 minutes | Responsibilities 2–4 |
| `rollaDidRequestTokenRefresh` fires repeatedly in bursts | Same cause — every request that fails after an unsuccessful internal refresh escalates to your app | Answer it with `updateToken()`; verify the refresh token you pass at initialization is the latest one |
| 401 errors start right after your own backend refreshes its session | Your backend consumed the refresh token the SDK was holding (single-use rotation) | Responsibility 5 |
| `rollaDidRequestTokenRefresh` fires immediately at launch | The access token was already expired and the refresh token invalid at initialization | Initialize with the newest persisted pair (a `tokenExpiresIn` that overshoots the token's own expiry is corrected automatically) |

---

**Previous:** [Apple Health Integration](06-apple-health.md) | **Next:** [Engine Lifecycle](08-engine-lifecycle.md) | **Home:** [README](README.md)
