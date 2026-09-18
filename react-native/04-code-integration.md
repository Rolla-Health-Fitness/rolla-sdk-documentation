# Code Integration

This section covers importing the wrapper, creating a configuration, presenting the SDK, subscribing to its events, and handling errors.

## Import

```ts
import { Rolla } from '@rolla-health/react-native-sdk';
```

`Rolla` is a namespace of static, promise-returning methods plus an event emitter — there is no class to instantiate. TypeScript types are bundled; import them from the same package (`RollaConfiguration`, `RollaBranding`, `RollaCloseEvent`, …).

## Authentication & Token Flow

The SDK needs a **user access token** (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API **after** the user has logged in.

- **Typical flow:** User logs in to your app → your app calls your backend → your backend returns `access_token`, `refresh_token`, and `expires_in` from Rolla's auth API → you pass all three into the configuration when opening the SDK.
- **When to fetch:** Before calling `Rolla.show()`. If the user is already logged in, use your existing session (a stored token, or a refresh to get a new access token).
- **What to pass:** All three token fields — `token`, `refreshToken`, and `tokenExpiresIn`. The access token alone opens the SDK, but the other two are what let it keep the session alive on its own — see [Token Management](06-token-management.md#your-apps-responsibilities).
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

> **Note:** You are responsible for authentication — the SDK only consumes the token you provide.

## Create Configuration

```ts
import type { RollaConfiguration } from '@rolla-health/react-native-sdk';

const configuration: RollaConfiguration = {
  token: 'your-access-token',
  refreshToken: 'your-refresh-token',  // Optional
  tokenExpiresIn: 1800,                // Optional: token expiry in seconds
  partnerId: 'your-partner-id',
  environment: 'production',           // or 'rnd' for development
};
```

These are the identity and auth essentials. `RollaConfiguration` also takes `branding`, `language`, `disabledModules`, `disabledDataSources`, `userId`, `showOptionsButton` and `showGoalsSection` — see [Configuration](05-configuration.md) for the full reference.

### Environment Values

| Value | Description |
|-------|-------------|
| `'production'` | Live / release builds |
| `'rnd'` | Development and QA (sandbox environment) |

If omitted, defaults to `'rnd'`. Use `'rnd'` during development and QA to test against a sandbox environment without affecting production data. Switch to `'production'` for release builds.

> **Why `'rnd'`?** The name stands for "Research and Development" — it is the SDK's internal label for the non-production sandbox environment.

## Present the SDK

```ts
const close = await Rolla.show(configuration);
console.log('Rolla closed because:', close.reason);
```

`Rolla.show()` presents the SDK UI and resolves with a `RollaCloseEvent` — `{ reason, detail? }` — once it is dismissed; `reason` is one of the [`RollaCloseReasonKind`](08-api-reference.md#rollaclosereason) values. Pass `{ transition: 'fade' }` as the second argument to cross-fade instead of the platform's default animation — see [Configuration → Transition](05-configuration.md#transition).

Every entry point takes the configuration it runs under, exactly as a native host builds a `Rolla(configuration)` per call. The SDK engine itself is process-wide: the first call starts it, later calls reuse it, and `destroyEngine()` is how a changed configuration takes effect — see [Engine Lifecycle](07-engine-lifecycle.md).

Instead of `show()`, `openScreen()` opens the SDK directly on a specific screen (insights, activity history, goals, …) — for example from your own menu entries. The SDK's own notifications fit the same model: each names the screen it should open, the wrapper delivers that target to JavaScript, and a screen target goes to `openScreen()` — see [Host-Driven Navigation](08-api-reference.md#host-driven-navigation) and [Notification Taps](08-api-reference.md#notification-taps).

> **Call `show()` and `openScreen()` from a settled screen.** On iOS the SDK UI is presented on top of the frontmost view controller. If the call is triggered from inside a React Native `Modal` (a bottom sheet, a picker) that is closing at the same time, the SDK is presented on that modal's view controller and dismissed together with it — the call appears to do nothing, or closes immediately with `hostModalDismiss`. Wait for the modal's `onDismiss` before calling. Android's `Modal` is a dialog window and is not affected.

## Subscribe to Events

Events are the counterpart of the native `RollaDelegate` / `RollaListener` callbacks. Subscribe with `Rolla.addListener()` and remove the subscription in your effect cleanup — leaked listeners keep receiving events after the component unmounts:

```tsx
import { useEffect } from 'react';
import { Rolla } from '@rolla-health/react-native-sdk';

useEffect(() => {
  const closed = Rolla.addListener('onClose', (e) => {
    // The SDK UI was dismissed — the same event resolves the show() promise
    console.log('Closed:', e.reason);
  });

  const failed = Rolla.addListener('onError', (e) => {
    console.warn('Rolla SDK error:', e.code, e.message);
  });

  const refreshed = Rolla.addListener('onTokenRefreshed', (e) => {
    // The SDK refreshed the token internally — store the new pair for future use
    SessionManager.updateToken(e.token, e.refreshToken, e.expiresIn);
  });

  const expired = Rolla.addListener('onTokenExpired', async () => {
    // The token expired and the SDK cannot refresh it. Obtain fresh tokens from
    // the Rolla auth API (/api/login), directly or through your backend, and push them:
    const fresh = await YourAPI.fetchNewToken();
    await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
  });

  return () => {
    closed.remove();
    failed.remove();
    refreshed.remove();
    expired.remove();
  };
}, []);
```

The four above are the presentation and token events — the minimum for a production integration. Twelve more observational events push SDK happenings to your app (activity lifecycle, sync results, band pairing and live link state, primary source, goals, profile), plus `onNotificationTap` for the SDK's notifications — see [API Reference → Events](08-api-reference.md#events). Subscribe once, at your app root or in a custom hook, rather than in every screen that calls `Rolla.show()`.

`onClose` is also delivered as the `show()` promise resolution. Subscribing to it is useful when several components need to react to the same close — the promise can only be awaited once.

## Handle Errors

`Rolla.show()` rejects when the SDK UI could not be presented; the error carries a `code`:

```ts
try {
  await Rolla.show(configuration);
} catch (e: any) {
  // e.code: 'INVALID_CONFIG' | 'ALREADY_PRESENTING' | 'NO_PRESENTER' (iOS) | 'NO_ACTIVITY' (Android)
  //         | a native RollaError code such as 'ENGINE_FAILED' or 'INIT_FAILED'
  console.warn(e.code, e.message);
}
```

The same failure also fires `onError` with `presentationFailed: true`; errors raised while the SDK UI is running arrive through `onError` alone, with `presentationFailed: false`. A value the SDK does not know — a misspelled module name, an unparsable color, an unknown transition — rejects with `INVALID_CONFIG` instead of being silently ignored. The full list of codes and the recommended recovery for each is in [API Reference → Errors](08-api-reference.md#errors).

## Threading

There are no threading rules to follow from JavaScript. Every `Rolla` method dispatches to the native main thread internally, every promise settles on the JavaScript thread, and every event is delivered there — update your state directly inside the handlers.

## Cross-Platform Note: `tokenExpiresIn`

`tokenExpiresIn` and the `expiresIn` field of `onTokenRefreshed` are **seconds until expiry**, as a JavaScript `number` — the same meaning as `TimeInterval` on iOS and `Int` on Android. If your backend returns `expires_in` in seconds, pass it straight through. If you store tokens as an absolute `expiresAt` (epoch seconds), convert first:

```ts
const tokenExpiresIn = Math.max(0, Math.floor((expiresAt * 1000 - Date.now()) / 1000));
```

---

**Previous:** [Permissions & Entitlements](03-permissions.md) | **Next:** [Configuration](05-configuration.md) | **Home:** [README](README.md)
