# Token Management

The SDK manages the token lifecycle by itself. However, the refresh tokens issued by the Rolla auth API are single-use, so your app still carries a small set of obligations that will keep the session healthy beyond the first access-token expiry. The model is identical to the native SDKs' — [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md) — surfaced as two events and two methods.

## How It Works

1. **Initialization:** You provide `token`, `refreshToken`, and `tokenExpiresIn` (seconds) in the configuration you pass to `Rolla.show()` or any other entry point — always the newest pair your app has stored (see [Your App's Responsibilities](#your-apps-responsibilities)).
2. **Internal refresh:** The SDK refreshes the access token automatically — proactively, shortly before the token expires (based on `tokenExpiresIn`), and reactively, when a request receives HTTP 401. After every successful internal refresh while the SDK UI is presented, the SDK hands the **new token pair** to your app via the `onTokenRefreshed` event.
3. **Expired session (SDK cannot refresh):** If the internal refresh fails (typically because the refresh token was consumed outside the SDK, or has expired), the SDK emits `onTokenExpired` and holds the failing request for up to 10 seconds while you answer. Obtain a fresh token pair from the Rolla auth API — by re-authenticating via [`/api/login`](../sdk-auth-api/02-authentication.md#log-in), directly or through your backend — and push it with `Rolla.updateToken()`: a push inside that 10 s window retries the request with the new tokens, and the user sees no error. If the window passes unanswered — or answered only with a pair older than the one the SDK already holds, which is ignored (see [responsibility 4](#your-apps-responsibilities)) — screens that request backend data show an error state and recover on their next load after the tokens arrive.

   > **Avoiding this state:** pass the refresh token to the SDK, but never spend it yourself — let the SDK do all refreshing.
4. **Logout / session clear:** Call `Rolla.clearSession()` when the user logs out. It removes all SDK-persisted tokens and session data from secure storage.

## Token Facts

| Fact | Value |
|------|-------|
| Access token lifetime | 30 minutes (`expires_in: 1800` in auth responses) |
| Refresh token lifetime | 30 days (`refresh_expires_in: 2592000`) |
| Refresh token reuse | **Single-use.** Every successful [`/api/refresh_token`](../sdk-auth-api/02-authentication.md#refresh-token) call returns a *new* refresh token and permanently invalidates the one that was just used — regardless of whether the SDK or your own code made the call. |

## Your App's Responsibilities

All of the following should be implemented.

1. **Pass all three token fields in every configuration.** `token` alone is enough to open the SDK, but without `refreshToken` the SDK cannot refresh at all on its own and every expiry escalates to `onTokenExpired`; without `tokenExpiresIn` the SDK cannot refresh proactively and only recovers after the first 401.
2. **Subscribe to `onTokenRefreshed` and persist the delivered pair.** The SDK's internal refresh rotates the tokens, so the pair your app stored earlier is stale from that moment on. Overwrite it with the delivered pair, which is now the only pair that can refresh.
3. **Subscribe to `onTokenExpired` and answer it with `Rolla.updateToken()`.** This is the recovery path when the SDK cannot help itself. Obtain a fresh pair from the Rolla auth API ([`/api/login`](../sdk-auth-api/02-authentication.md#log-in)), directly or through your backend, and push it. The session then recovers in place, without the user leaving the SDK.
4. **Always configure with the newest pair you have.** The SDK compares the tokens you pass against the pair it already holds (through the tokens' own JWT claims) and **ignores anything older** — so a re-sent original login pair can no longer overwrite a newer pair the SDK obtained by rotation. Configuring with the newest persisted pair still matters in the other direction: when your app re-authenticates, the fresh pair outranks the SDK's stored one and is what re-arms the session.
   - Note: if you leave `refreshToken` unset on a later call, the SDK keeps the refresh token it already holds; an access token older than the stored one is ignored entirely.
5. **Keep the SDK's refresh token exclusive to the SDK.** If your backend uses the same refresh token for its own session refresh, whichever side refreshes first invalidates the token for the other. Issue your own session credentials separately, or route all refreshes through a single owner.
6. **Call `Rolla.clearSession()` on logout** so the next user cannot inherit tokens or data from the previous user.

## Token Events

Both events are part of the [event set](08-api-reference.md#events). Subscribe once, at your app root or in a custom hook, and remove the subscriptions in the effect cleanup:

```tsx
import { useEffect } from 'react';
import { Rolla } from '@rolla-health/react-native-sdk';

useEffect(() => {
  // Emitted when the SDK refreshes the token internally
  const refreshed = Rolla.addListener('onTokenRefreshed', (e) => {
    // The SDK rotated the tokens — persist this pair; it replaces every
    // previously stored pair, including the original login response.
    SessionManager.updateToken(e.token, e.refreshToken, e.expiresIn);
  });

  // Emitted when the SDK cannot refresh the token — your app must provide a new one
  const expired = Rolla.addListener('onTokenExpired', async () => {
    // Obtain fresh tokens from the Rolla auth API (/api/login), directly or
    // through your backend, and hand them over — the session recovers in place.
    const fresh = await YourAPI.fetchNewToken();
    await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
  });

  return () => {
    refreshed.remove();
    expired.remove();
  };
}, []);
```

`onTokenExpired` has an empty payload — the contract is that you call `updateToken()` with new credentials. Point `YourAPI.fetchNewToken()` at your own session refresh, not at a prompt for the user's password inside the handler.

## Pushing a New Token

If you refresh tokens outside the SDK (e.g. during a background refresh in your app), push the new pair to the SDK at any time — you don't need to wait for an event. Two caveats:

- `updateToken()` needs a running engine — any prior `show()`, `openScreen()`, `warmUpEngine()`, or headless call. On a cold engine it rejects with `NO_ACTIVE_SESSION`; put the newest pair in your next configuration instead.
- A resolved promise means the push was delivered, not that it was applied. A pair older than the one the SDK already holds is ignored by design (see [responsibility 4](#your-apps-responsibilities)) and still resolves.

```ts
await Rolla.updateToken(
  newAccessToken,
  newRefreshToken,  // Optional: unset keeps the SDK's stored refresh token (if it exists)
  1800,             // Optional: seconds until expiry
);
```

Rejects with `{ code, message }` when the SDK could not accept the push — `NO_ACTIVE_SESSION` on a cold engine; otherwise `UPDATE_TOKEN_FAILED`, or on iOS the SDK's own `RollaError` code when it reports one.

## Clearing the Session

When the user logs out of your app, call `clearSession()` to remove all SDK-persisted tokens and session data. Like `updateToken()`, the native clear needs a running engine. Pass the current configuration and the wrapper handles the cold case for you: it warms the engine first when none is running, then clears — the SDK's documented logout recipe. Then tear the engine down, only once the clear has resolved:

```ts
async function logout() {
  await Rolla.clearSession(configuration); // warms a cold engine first, then purges tokens and session data
  await Rolla.destroyEngine();             // only after the clear has resolved
  yourSession.clear();
}
```

Without a configuration, `clearSession()` clears a running engine and rejects with `NO_ACTIVE_SESSION` on a cold one — rather than reporting a clear that never happened. A native failure rejects with `CLEAR_SESSION_FAILED`, or on iOS with the SDK's own `RollaError` code when it reports one.

> **Order matters on logout.** `clearSession` completes asynchronously — call `Rolla.destroyEngine()` after it resolves, never before. Destroying the engine first cancels the pending clear, and the session data silently survives. See [Engine Lifecycle → `clearSession` vs `destroyEngine`](07-engine-lifecycle.md#clearsession-vs-destroyengine).

After `clearSession()`, the next `show()` starts with whatever credentials you pass at that point — there is no implicit token re-use.

---

**Previous:** [Configuration](05-configuration.md) | **Next:** [Engine Lifecycle](07-engine-lifecycle.md) | **Home:** [README](README.md)
