# Token Management

The Rolla SDK manages the access-token lifecycle internally and surfaces it through Dart callbacks you pass to `RollaSDK.initializeWithToken(...)`. The conceptual model is identical to the native SDKs — see [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md) for the underlying state machine.

## How it works

1. **Initialization** — you pass `accessToken`, optionally `refreshToken`, and optionally `tokenExpiresIn` to `RollaSDK.initializeWithToken(...)`. Unlike the native SDKs (where expiry is `TimeInterval?` / `Int?` seconds), Flutter's `tokenExpiresIn` is a **`Duration?`**.
2. **Expiry / 401** — when an API call returns `401` (or the token is past `tokenExpiresIn`), the SDK invokes your `onTokenExpired` callback. Your job is to mint a fresh token (from your own backend) and return it as a `TokenRefreshResult`. The SDK persists the returned credentials and retries.
3. **Refresh failed** — if your callback returns `null` (or throws), the SDK treats the session as unauthenticated and triggers logout (it also calls your `onLogout`, if set).
4. **Logout** — call `RollaSDK.logout()` when the user signs out of your app. It clears all SDK-persisted tokens and disposes the SDK instance.

> **Never re-prompt the user for credentials inside `onTokenExpired`.** Point it at a backend endpoint that exchanges your stored refresh token for a fresh access token. The app should never hold a partner password.

## The refresh callback

`onTokenExpired` is a `Future<TokenRefreshResult?> Function()`. Return a `TokenRefreshResult` to keep the session alive, or `null` to force logout.

```dart
import 'package:rolla_sdk/rolla_sdk.dart';

await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,
  // Flutter takes a Duration, not seconds.
  tokenExpiresIn: Duration(seconds: session.expiresIn),
  userId: 'user_123',
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd, // rnd sandbox during integration

  // The SDK calls this on 401 / expiry. Fetch fresh tokens from YOUR backend
  // and hand them back. Return null to force a logout.
  onTokenExpired: () async {
    try {
      final refreshed = await yourBackend.refreshToken();
      return TokenRefreshResult(
        accessToken: refreshed.accessToken,
        refreshToken: refreshed.refreshToken,                  // optional
        expiresIn: Duration(seconds: refreshed.expiresIn),     // optional, a Duration
      );
    } catch (_) {
      return null; // refresh failed → SDK logs the user out
    }
  },

  // Called when the SDK logs the user out (including after a null refresh).
  // Update your auth state and navigate back to your login screen.
  onLogout: () {
    if (mounted) Navigator.of(context).pop();
  },
);
```

This mirrors `rolla-sdk-demo-flutter`'s launch screen, which re-uses its `TokenService` for the refresh step.

### `TokenRefreshResult`

```dart
class TokenRefreshResult {
  final String accessToken;     // required
  final String? refreshToken;   // optional — only set if your backend rotates it
  final Duration? expiresIn;    // optional — Duration, not seconds
}
```

If your backend returns expiry as seconds (or an `expiresAt` epoch), convert to a `Duration` before passing it in:

```dart
// seconds → Duration
Duration(seconds: expiresInSeconds);

// epoch seconds → Duration
final secs = (expiresAtEpoch - DateTime.now().millisecondsSinceEpoch ~/ 1000);
Duration(seconds: secs.clamp(0, secs));
```

> If you omit `tokenExpiresIn`, the SDK still recovers via the `401` → `onTokenExpired` path; it just refreshes reactively rather than proactively.

## Pushing a new token

If you refresh tokens outside the SDK (e.g. during a background refresh in your app), push the new credentials in at any time with the static `updateToken`:

```dart
await RollaSDK.updateToken(
  accessToken: newAccessToken,
  refreshToken: newRefreshToken,        // optional
  expiresIn: Duration(seconds: 1800),   // optional
);
```

It is a no-op if the SDK has not been initialized.

## Logging out

```dart
await RollaSDK.logout();
```

This clears all SDK-persisted tokens and disposes the SDK instance. The next `RollaSDK.initializeWithToken(...)` starts a fresh session with whatever credentials you pass — there is no implicit token re-use.

## Pre-warm before launch

Fetch (or refresh) the token immediately before `initializeWithToken(...)` so the SDK starts with the maximum remaining lifetime — exactly what the demo's `RollaLaunchScreen` does:

```dart
final session = await yourBackend.fetchTokens();
await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,
  tokenExpiresIn: Duration(seconds: session.expiresIn),
  userId: widget.userId,
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd, // rnd sandbox during integration
  onTokenExpired: _onTokenExpired,
  onLogout: _onLogout,
);
```

See [API Reference → RollaSDK.initializeWithToken](08-api-reference.md) for the full parameter list.

---

**Next:** [Permissions Gate](07-permissions-gate.md) | **Home:** [README](README.md)
