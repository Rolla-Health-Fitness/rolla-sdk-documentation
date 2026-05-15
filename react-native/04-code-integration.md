# Code Integration

The wrapper exposes a single `Rolla` namespace with promise-style methods and an event emitter. There is no class to instantiate — the underlying engine is a singleton owned by the native side.

## Import

```ts
import { Rolla } from '@rolla-health/react-native-sdk';
```

TypeScript types are bundled. No additional `@types/...` install is needed.

## Present the modal

```ts
const close = await Rolla.show({
  token: 'access-token-from-your-backend',
  refreshToken: 'optional-refresh-token',
  tokenExpiresIn: 1800,                    // seconds; optional
  partnerId: 'your-partner-id',
  environment: 'production',                // 'production' | 'staging' | 'rnd'
  branding: { /* optional, see Branding & Modules */ },
});

console.log('Rolla closed because:', close.reason);
```

`Rolla.show()` returns a promise that resolves with `{ reason, detail? }` when the modal is dismissed. The `reason` is one of the `RollaCloseReasonKind` enum values — see [API Reference → RollaCloseEvent](08-api-reference.md#rollacloseevent).

## Subscribe to lifecycle events

```tsx
import { useEffect } from 'react';
import { Rolla } from '@rolla-health/react-native-sdk';

useEffect(() => {
  const closeSub = Rolla.addListener('onClose', (e) => {
    console.log('Closed:', e.reason);
  });
  const errorSub = Rolla.addListener('onError', (e) => {
    console.warn('SDK error:', e.code, e.message);
  });
  return () => {
    closeSub.remove();
    errorSub.remove();
  };
}, []);
```

Available events: `onClose`, `onError`, `onTokenRefreshed`, `onTokenExpired`. Payloads in [API Reference → Events](08-api-reference.md#events).

**Always remove subscriptions in your effect cleanup.** Leaked listeners receive events after the component unmounts.

## Handle token refresh

The wrapper exposes the same token lifecycle as the native SDKs. Subscribe to `onTokenExpired` and push fresh credentials with `Rolla.updateToken()`:

```ts
useEffect(() => {
  const sub = Rolla.addListener('onTokenExpired', async () => {
    const fresh = await yourBackend.refreshToken();
    await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
  });
  return () => sub.remove();
}, []);
```

Detailed lifecycle in [Token Management](06-token-management.md). The model is identical to [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md).

## Programmatic dismiss

```ts
await Rolla.dismiss();
```

Useful when navigating away from the screen that owns the modal. The pending `Rolla.show()` promise will resolve with `reason: 'programmatic'`.

## Engine teardown

The Flutter engine that powers the modal stays warm in memory across `show()` / dismiss cycles. To reclaim that memory:

```ts
await Rolla.destroyEngine();
```

See [Engine Lifecycle](07-engine-lifecycle.md). The trade-off (reclaim ~30 MB vs. add 1–2 s to the next `show()`) is the same as [iOS Engine Lifecycle](../ios/08-engine-lifecycle.md) and [Android Engine Lifecycle](../android/07-engine-lifecycle.md).

## Clear session

When the user logs out:

```ts
await Rolla.clearSession();
```

Removes all SDK-persisted tokens and session data. Safe to call when no session is active.

## Check presentation state

```ts
const isOpen = await Rolla.isPresenting();
```

Returns `true` while the modal is on screen.

## Native SDK version

```ts
const v = await Rolla.getNativeSdkVersion();
// → '0.1.10'
```

Useful in your app's diagnostics or to confirm the wrapper is wired correctly. If this throws `Invariant Violation: TurboModuleRegistry.getEnforcing('RollaWrapper') could not be found`, autolinking has not picked up the wrapper — see [Troubleshooting](09-troubleshooting.md#turbomoduleregistry-getenforcing-rollawrapper-could-not-be-found).

## Full minimal integration

```tsx
import { useEffect, useState } from 'react';
import { Pressable, Text, View } from 'react-native';
import { Rolla } from '@rolla-health/react-native-sdk';

export default function App() {
  const [status, setStatus] = useState('idle');

  useEffect(() => {
    const tokenSub = Rolla.addListener('onTokenExpired', async () => {
      const fresh = await yourBackend.refreshToken();
      await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
    });
    return () => tokenSub.remove();
  }, []);

  return (
    <View>
      <Pressable
        onPress={async () => {
          try {
            setStatus('opening…');
            const tokens = await yourBackend.fetchTokens();
            const close = await Rolla.show({
              token: tokens.accessToken,
              refreshToken: tokens.refreshToken,
              tokenExpiresIn: tokens.expiresIn,
              partnerId: 'your-partner-id',
              environment: 'production',
            });
            setStatus(`closed: ${close.reason}`);
          } catch (e: any) {
            setStatus(`err: ${e?.code ?? '?'} — ${e?.message ?? e}`);
          }
        }}>
        <Text>Open Rolla</Text>
      </Pressable>
      <Text>{status}</Text>
    </View>
  );
}
```

---

**Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
