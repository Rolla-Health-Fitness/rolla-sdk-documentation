# Token Management

The Rolla wrapper exposes the same token lifecycle as the native SDKs, surfaced as JS events and a single `updateToken()` method. The conceptual model is identical — see [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md) for the underlying state machine.

## How it works

1. **Initialization** — you pass `token`, `refreshToken`, and `tokenExpiresIn` (seconds) in `Rolla.show({ ... })`.
2. **Internal refresh** — when the access token is close to expiring, the SDK attempts to refresh it using the `refreshToken`. On success, the SDK emits `onTokenRefreshed` with the new credentials so your app can persist them.
3. **Expired (cannot refresh)** — if the SDK cannot refresh (e.g. the refresh token itself expired), it emits `onTokenExpired`. Your app must fetch new credentials from your backend and push them with `Rolla.updateToken()`.
4. **Logout** — call `Rolla.clearSession()` to remove all SDK-persisted tokens and session data.

## Subscribe to events

The reference pattern below mirrors `rolla-sdk-demo-react-native/src/HomeScreen.tsx` — it holds the latest tokens in a ref so `onTokenExpired` always starts from the SDK-refreshed values, not from whatever was passed to the initial `Rolla.show()`:

```tsx
import { useEffect, useRef } from 'react';
import { Rolla } from '@rolla-health/react-native-sdk';

type Tokens = { accessToken: string; refreshToken?: string; expiresIn?: number };

const tokensRef = useRef<Tokens | null>(null);

useEffect(() => {
  // SDK refreshed its own tokens — sync them so the next onTokenExpired
  // starts from the latest refresh token.
  const refreshed = Rolla.addListener('onTokenRefreshed', (e) => {
    if (tokensRef.current) {
      tokensRef.current = {
        accessToken: e.token,
        refreshToken: e.refreshToken ?? tokensRef.current.refreshToken,
        expiresIn: e.expiresIn ?? tokensRef.current.expiresIn,
      };
    }
    yourSession.update(e.token, e.refreshToken, e.expiresIn);
  });

  // SDK's internal refresh failed — fetch fresh tokens and push them in.
  const expired = Rolla.addListener('onTokenExpired', async () => {
    const fresh = await yourBackend.refreshToken();
    tokensRef.current = fresh;
    await Rolla.updateToken(fresh.accessToken, fresh.refreshToken, fresh.expiresIn);
  });

  return () => {
    refreshed.remove();
    expired.remove();
  };
}, []);
```

> The demo app re-uses `POST /api/login` for the refresh step. In production, point `yourBackend.refreshToken()` at a dedicated refresh endpoint that exchanges your stored refresh token for a fresh access token — do not re-prompt the user for credentials inside an `onTokenExpired` handler.

`onTokenExpired` has an empty payload — the contract is that you call `updateToken(...)` with new credentials. If you ignore the event, the SDK will surface an `onError` and dismiss with `reason: 'flutterRequested'`.

## Push a new token

If you refresh tokens outside the SDK (e.g. during a background refresh in your app), push the new token to the SDK at any time:

```ts
await Rolla.updateToken(
  newAccessToken,
  newRefreshToken,   // optional
  1800,              // optional: seconds until expiry
);
```

Returns `Promise<void>`. Rejects with `{ code, message }` if the SDK could not accept the token (e.g. modal already closed). See [API Reference → updateToken](08-api-reference.md#rolla-updatetoken).

## Clear the session

```ts
await Rolla.clearSession();
```

Removes all SDK-persisted tokens and session data. Safe to call when no session is active. The next `Rolla.show()` will start with whatever credentials you pass at that point — there is no implicit token re-use.

## Token expiry timestamps

`tokenExpiresIn` is **seconds from now**, not an absolute timestamp. If you store tokens as `expiresAt` (epoch seconds), convert:

```ts
const expiresIn = Math.max(0, Math.floor((expiresAt * 1000 - Date.now()) / 1000));
```

If you omit `tokenExpiresIn`, the SDK assumes a default lifetime (1800 s). You will still get `onTokenExpired` events, just less predictably timed.

## Common patterns

### Pre-warm refresh

If your backend supports it, refresh the token just before calling `Rolla.show()` so the SDK starts with maximum lifetime:

```ts
const fresh = await yourBackend.refreshToken();
await Rolla.show({
  token: fresh.token,
  refreshToken: fresh.refreshToken,
  tokenExpiresIn: fresh.expiresIn,
  partnerId: 'your-partner-id',
  environment: 'production',
});
```

### Single-listener architecture

Subscribe to `onTokenExpired` once at app root (e.g. in your root component or a custom hook) rather than re-subscribing inside every screen that calls `Rolla.show()`. The Rolla wrapper supports multiple listeners but only one of them needs to handle the refresh.

---

**Next:** [Engine Lifecycle](07-engine-lifecycle.md) | **Home:** [README](README.md)
