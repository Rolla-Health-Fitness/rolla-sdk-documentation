# Token Management

The SDK manages the token lifecycle by itself. However, the refresh tokens issued by the Rolla auth API are single-use, so your app still carries a small set of obligations that will keep the session healthy beyond the first access-token expiry. The model is the same as the native SDKs' — [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md) — surfaced in Flutter as the callbacks you pass to `initializeWithToken` and one method.

## How It Works

1. **Initialization:** You provide `accessToken`, `refreshToken`, and `tokenExpiresIn` (a `Duration`) to `RollaSDK.initializeWithToken` — always the newest pair your app has (see [Your app's responsibilities](#your-apps-responsibilities)). Fetch the token immediately before initializing so the SDK starts with the maximum remaining lifetime.
2. **Internal refresh:** The SDK refreshes the access token automatically — proactively, shortly before the token expires (based on `tokenExpiresIn`), and reactively, when a request receives HTTP 401 — and persists the rotated pair in its own secure storage. There is no callback for this in Flutter: the SDK owns the pair from here on.
3. **Expired session (SDK cannot refresh):** If the internal refresh fails (typically because the refresh token was consumed outside the SDK, or has expired), the SDK awaits your `onTokenExpired` callback. Return a fresh pair from your backend and the SDK persists it and retries the failed request — the user sees no error. Return `null` (or throw) and the SDK re-reads its storage once more; if nothing newer is there either, it calls `onSessionExpired`: the session is dead, and screens that request backend data show an error state until a new session is established.

   > **Avoiding this state:** pass the refresh token to the SDK, but never spend it yourself — let the SDK do all refreshing.
4. **Logout:** Call `RollaSDK.logout()` when the user logs out of your app. It removes all SDK-persisted tokens and session data and disposes the SDK instance.

## Token Facts

| Fact | Value |
|------|-------|
| Access token lifetime | 30 minutes (`expires_in: 1800` in auth responses) |
| Refresh token lifetime | 30 days (`refresh_expires_in: 2592000`) |
| Refresh token reuse | **Single-use.** Every successful [`/api/refresh_token`](../sdk-auth-api/02-authentication.md#refresh-token) call returns a *new* refresh token and permanently invalidates the one that was just used — regardless of whether the SDK or your own code made the call. |

## Your App's Responsibilities

All of the following should be implemented.

1. **Pass all three token fields on every initialization.** `accessToken` alone is enough to open the SDK, but without `refreshToken` the SDK cannot refresh at all on its own and every expiry escalates to `onTokenExpired`; without `tokenExpiresIn` the SDK cannot refresh proactively and only recovers after the first 401.
2. **Answer `onTokenExpired` with a fresh pair.** This is the recovery path when the SDK cannot help itself. Obtain a fresh pair from the Rolla auth API ([`/api/login`](../sdk-auth-api/02-authentication.md#log-in)), directly or through your backend, and return it as a `TokenRefreshResult`. The session then recovers in place, without the user leaving the SDK.
3. **Handle `onSessionExpired`.** When it fires, nothing can revive the current session: clear your own stored session and route the user to your login. It never fires during a deliberate `RollaSDK.logout()`.
4. **Always initialize with the newest pair you have.** The SDK compares the tokens you pass against the pair it already holds (through the tokens' own JWT claims) and **ignores anything older** — so a re-sent original login pair can no longer overwrite a newer pair the SDK obtained by rotation. Initializing with the newest persisted pair still matters in the other direction: when your app re-authenticates, the fresh pair outranks the SDK's stored one and is what re-arms the session.
   - Note: if you leave `refreshToken` unset on a later call, the SDK keeps the refresh token it already holds; an access token older than the stored one is ignored entirely.
5. **Keep the SDK's refresh token exclusive to the SDK.** If your backend uses the same refresh token for its own session refresh, whichever side refreshes first invalidates the token for the other. Issue your own session credentials separately, or route all refreshes through a single owner.
6. **Call `RollaSDK.logout()` on logout** so the next user cannot inherit tokens or data from the previous user.

## The Refresh Callback

`onTokenExpired` is a `Future<TokenRefreshResult?> Function()`. Return a `TokenRefreshResult` to keep the session alive, or `null` if you could not refresh:

```dart
onTokenExpired: () async {
  try {
    final refreshed = await myBackend.fetchRollaTokens();
    return TokenRefreshResult(
      accessToken: refreshed.accessToken,
      refreshToken: refreshed.refreshToken,              // optional — set if your backend rotates it
      expiresIn: Duration(seconds: refreshed.expiresIn), // optional
    );
  } catch (_) {
    return null; // nothing fresher available
  }
},
```

```dart
class TokenRefreshResult {
  final String accessToken;     // required
  final String? refreshToken;   // optional
  final Duration? expiresIn;    // optional
}
```

> **Return `null` rather than a token you know is expired.** A non-null result tells the SDK the session is healthy — handing back a stale token strands the user in a session where every request fails. The SDK does reject a pair older than the one it holds, but an unexpired-looking token of the same age passes that guard.

The callback is awaited, so the failing request waits for it; point it at a quick backend call, not at a prompt for the user's password. If you omit `tokenExpiresIn`, the SDK still recovers via the `401` → `onTokenExpired` path; it just refreshes reactively rather than proactively.

## Pushing a New Token

If you refresh tokens outside the SDK (e.g. your own API client refreshed in the background), push the new pair in at any time:

```dart
final accepted = await RollaSDK.updateToken(
  accessToken: newAccessToken,
  refreshToken: newRefreshToken,        // optional: unset keeps the SDK's stored refresh token
  expiresIn: Duration(seconds: 1800),   // optional
);
```

It returns `true` when the pair is now the SDK's current one, and `false` when the SDK kept a newer pair it already held (a replayed pair is ignored by design — see [responsibility 4](#your-apps-responsibilities)) or when `initializeWithToken` has never been called.

## Logging Out

```dart
await RollaSDK.logout();
```

This clears all SDK-persisted tokens and disposes the SDK instance. The next `initializeWithToken(...)` starts a fresh session with whatever credentials you pass.

When logout is initiated *inside* the SDK, the order is reversed: the SDK clears its own session first, then calls your `onLogout`. Use the callback to update your auth state and navigate — don't call SDK-authenticated endpoints from it; the session is already gone.

---

**Previous:** [Configuration](05-configuration.md) | **Next:** [API Reference](07-api-reference.md) | **Home:** [README](README.md)
