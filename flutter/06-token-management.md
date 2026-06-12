# Token Management

The SDK manages the access-token lifecycle internally and surfaces it through Dart callbacks you pass to `RollaSDK.initializeWithToken(...)`. Tokens come from Rolla's authentication API — your backend calls `POST /api/login` (and `POST /api/refresh_token`) and hands the results to your app. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the endpoints and token lifetimes. The conceptual model matches the native SDKs: [iOS Token Management](../ios/07-token-management.md) | [Android Token Management](../android/06-token-management.md).

## How it works

1. **Initialization** — pass `accessToken`, optionally `refreshToken`, and optionally `tokenExpiresIn`. Fetch the token immediately before initializing so the SDK starts with the maximum remaining lifetime. Note that `tokenExpiresIn` is a **`Duration`**, not seconds.
2. **Expiry / 401** — the SDK first refreshes internally using the `refreshToken` you supplied (`POST /api/refresh_token`). If you supplied both tokens, you will rarely see the callback fire.
3. **Fallback** — when the internal refresh is unavailable or fails, the SDK invokes your `onTokenExpired` callback. Mint a fresh token from your backend and return it as a `TokenRefreshResult`; the SDK persists the new credentials and retries. If you return `null` (or throw), the refresh has failed: SDK requests error until fresh tokens arrive — treat this as your cue to re-authenticate the user.
4. **Logout** — call `RollaSDK.logout()` when the user signs out of your app. It clears all SDK-persisted tokens and disposes the SDK instance.

> **Never re-prompt the user for credentials inside `onTokenExpired`.** Point it at a backend endpoint that exchanges your stored refresh token for a fresh access token (`POST /api/refresh_token`). The app should never hold partner credentials.

## The refresh callback

`onTokenExpired` is a `Future<TokenRefreshResult?> Function()`. Return a `TokenRefreshResult` to keep the session alive, or `null` if you could not refresh:

```dart
onTokenExpired: () async {
  try {
    final refreshed = await myBackend.fetchRollaTokens();
    return TokenRefreshResult(
      accessToken: refreshed.accessToken,
      refreshToken: refreshed.refreshToken,              // optional
      expiresIn: Duration(seconds: refreshed.expiresIn), // optional
    );
  } catch (_) {
    return null; // refresh failed
  }
},
```

```dart
class TokenRefreshResult {
  final String accessToken;     // required
  final String? refreshToken;   // optional — set if your backend rotates it
  final Duration? expiresIn;    // optional — a Duration, not seconds
}
```

> **Return `null` rather than a token you know is expired.** A non-null result tells the SDK the session is healthy — handing back a stale token strands the user in a session where every request fails.

If you omit `tokenExpiresIn`, the SDK still recovers via the `401` → `onTokenExpired` path; it just refreshes reactively rather than proactively.

## Pushing a new token

If you refresh tokens outside the SDK (e.g. your own API client refreshed in the background), push the new credentials in at any time:

```dart
await RollaSDK.updateToken(
  accessToken: newAccessToken,
  refreshToken: newRefreshToken,        // optional
  expiresIn: Duration(seconds: 1800),   // optional
);
```

It is a no-op if `initializeWithToken` has never been called.

## Logging out

```dart
await RollaSDK.logout();
```

This clears all SDK-persisted tokens and disposes the SDK instance. The next `initializeWithToken(...)` starts a fresh session with whatever credentials you pass.

When logout is initiated *inside* the SDK, the order is reversed: the SDK clears its own session first, then calls your `onLogout`. Use the callback to update your auth state and navigate — don't call SDK-authenticated endpoints from it; the session is already gone.

---

**Next:** [API Reference](07-api-reference.md) | **Home:** [README](README.md)
