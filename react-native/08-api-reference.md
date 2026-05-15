# API Reference

Complete TypeScript surface of `@rolla-health/react-native-sdk`. All methods are static on the `Rolla` namespace.

```ts
import { Rolla } from '@rolla-health/react-native-sdk';
```

## Methods

### `Rolla.show(config)`

Presents the Rolla modal. Returns a promise that resolves with `RollaCloseEvent` when the modal is dismissed.

```ts
Rolla.show(config: RollaConfiguration): Promise<RollaCloseEvent>
```

### `Rolla.dismiss()`

Programmatically closes the modal. The pending `show()` resolves with `reason: 'programmatic'`. No-op if no modal is presented.

```ts
Rolla.dismiss(): Promise<void>
```

### `Rolla.updateToken(token, refreshToken?, expiresIn?)`

Push fresh credentials to the SDK after an `onTokenExpired` event or out-of-band refresh.

```ts
Rolla.updateToken(
  token: string,
  refreshToken?: string,
  expiresIn?: number,   // seconds until token expires
): Promise<void>
```

### `Rolla.clearSession()`

Remove all SDK-persisted tokens and session data. Safe to call when no session is active.

```ts
Rolla.clearSession(): Promise<void>
```

### `Rolla.destroyEngine()`

Tear down the embedded Flutter engine and reclaim memory. The next `show()` will cold-start.

```ts
Rolla.destroyEngine(): Promise<void>
```

### `Rolla.isPresenting()`

`true` while the modal is on screen.

```ts
Rolla.isPresenting(): Promise<boolean>
```

### `Rolla.getNativeSdkVersion()`

The version of the underlying native SDK (e.g. `'0.1.10'`). Useful for diagnostics.

```ts
Rolla.getNativeSdkVersion(): Promise<string>
```

### `Rolla.addListener(event, fn)`

Subscribe to lifecycle events. Returns a subscription whose `.remove()` you must call in your cleanup.

```ts
Rolla.addListener<E extends RollaEventName>(
  event: E,
  fn: (payload: RollaEventMap[E]) => void,
): RollaSubscription
```

### `Rolla.removeAllListeners()`

Hard reset all subscriptions. Useful between integration tests; not commonly needed in production code.

```ts
Rolla.removeAllListeners(): void
```

## Types

### `RollaConfiguration`

```ts
interface RollaConfiguration {
  token: string;
  refreshToken?: string;
  tokenExpiresIn?: number;       // seconds until access token expires
  userId?: string;
  partnerId: string;
  environment: RollaEnvironment;
  disabledModules?: string[];    // reserved; not currently honored
  branding?: RollaBranding;
  showSettingsButton?: boolean;
}
```

### `RollaEnvironment`

```ts
type RollaEnvironment = 'production' | 'staging' | 'development' | 'rnd' | string;
```

Must match the environment your token was issued for.

### `RollaBranding`

```ts
interface RollaBranding {
  appName?: string;
  primaryColor?: string;         // '#RRGGBB' or '#RRGGBBAA' — do NOT use processColor()
  secondaryColor?: string;
  accentColor?: string;
  brightness?: 'light' | 'dark';
  defaultThemeMode?: 'light' | 'dark' | 'system';
  defaultLocale?: string;        // BCP-47, e.g. 'en-US'
  headerLogoAsset?: string;      // path provided by Rolla; must be pre-bundled
  termsUrl?: string;
  privacyUrl?: string;
}
```

### `RollaCloseEvent`

```ts
interface RollaCloseEvent {
  reason: RollaCloseReasonKind;
  detail?: string;               // present when reason === 'flutterRequested'
}
```

### `RollaCloseReasonKind`

```ts
type RollaCloseReasonKind =
  | 'flutterRequested'      // SDK initiated dismiss (e.g. user tapped close, or error)
  | 'hostNavigationBack'    // host nav back button (Android)
  | 'hostModalDismiss'      // host swipe-down / parent modal close (iOS)
  | 'programmatic'          // Rolla.dismiss() called
  | 'hostStackReplaced'     // host nav stack replaced (Android)
  | 'unknown';
```

### Events

| Event | Payload |
| --- | --- |
| `onClose` | `RollaCloseEvent` |
| `onError` | `RollaErrorEvent` |
| `onTokenRefreshed` | `RollaTokenRefreshedEvent` |
| `onTokenExpired` | `{}` (empty; you must call `updateToken()`) |

```ts
interface RollaErrorEvent {
  code: string;
  message: string;
}

interface RollaTokenRefreshedEvent {
  token: string;
  refreshToken?: string;
  expiresIn?: number;
}
```

`onClose` is also emitted by the `Rolla.show()` promise resolution. Subscribing to `onClose` is useful when multiple components need to react to the same close event — the promise can only be awaited once.

### `RollaSubscription`

```ts
interface RollaSubscription {
  remove(): void;
}
```

## Errors thrown by `show()` / `updateToken()`

Method promises reject with `{ code: string; message: string }`:

| `code` | When |
| --- | --- |
| `RollaSDK/ALREADY_PRESENTING` | `show()` called while another modal is on screen |
| `RollaSDK/NOT_PRESENTING` | `updateToken()` / `dismiss()` called when no modal is active |
| `RollaSDK/INVALID_CONFIG` | `partnerId` missing or `environment` not accepted |
| `RollaSDK/NATIVE_ERROR` | Underlying native SDK error — `message` includes the native description |

The codes mirror the iOS `RollaError` enum and Android `RollaError` sealed class. See [iOS API Reference](../ios/10-api-reference.md) and [Android API Reference](../android/08-api-reference.md) for the canonical list.

---

**Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
