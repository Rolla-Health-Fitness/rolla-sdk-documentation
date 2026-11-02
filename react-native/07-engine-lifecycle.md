# Engine Lifecycle

The SDK uses a Flutter engine internally, owned by the native side and shared by every `Rolla` call in your app. This section covers engine creation, dismissal, and cleanup strategies. The model is identical to [iOS Engine Lifecycle](../ios/08-engine-lifecycle.md) and [Android Engine Lifecycle](../android/07-engine-lifecycle.md).

## Default Behavior

- **First `Rolla.show()`** — Creates the Flutter engine from the configuration you pass and presents the SDK UI.
- **Dismissal** — Closing the SDK UI, by the user or via `dismiss()`, **keeps the engine alive** in the background. The next `show()` presents the SDK instantly in its last state (no reload).
- This is the recommended behavior for most apps.

The trade-off: a warm engine keeps its memory; a destroyed engine costs a brief cold start on the next `show()`. Reclaim memory only when you know the user is done with the SDK for a while.

## Warming Up the Engine

The engine also starts automatically on the first headless call (`getBandBatteryLevel`, `getPairedBandInfo`, `syncHealthData` — see [API Reference → Headless Methods](08-api-reference.md#headless-methods)) and on `openScreen()`, so none of them require a prior `show()`. Call `warmUpEngine()` to pay the start-up cost early — typically right after login — so the first `show()` presents instantly:

```ts
await Rolla.warmUpEngine(configuration);
// Engine configured and ready — headless calls and host events now have zero start-up latency.
```

Safe to call repeatedly: a repeat call for the same user is a no-op that preserves the session. The warmed engine holds memory until `destroyEngine()`. Rejects with the SDK's `RollaError` code when start-up fails.

## Programmatic Dismiss

```ts
await Rolla.dismiss();
```

Closes the SDK UI if it is on screen; the pending `show()` promise resolves with `{ reason: 'programmatic' }`. Safe to call when nothing is presented (a no-op). Use `Rolla.isPresenting()` to check whether the SDK UI is currently on screen.

## Destroying the Engine

If you need to free memory (e.g. on user logout, or when the user won't return to the SDK for a while):

```ts
await Rolla.destroyEngine();
```

- This fully tears down the Flutter engine and frees its resources.
- The next `show()` (or `openScreen()`, or headless call) creates a fresh engine automatically from the configuration it is given, with a brief loading time.
- Call this **after** the SDK UI has closed, not while it is presenting.
- Host-event delivery (see [API Reference → Events](08-api-reference.md#events)) also stops here — events flow for the engine's lifetime.
- Destroying the engine is also how a new `RollaConfiguration` is applied — a changed language, branding, or module set takes effect on the next engine start. See [Configuration](05-configuration.md).

## `clearSession` vs `destroyEngine`

| Method | What It Does | Engine Stays Alive | When to Use |
|--------|-------------|:------------------:|-------------|
| `clearSession(config?)` | Purges stored tokens and auth metadata from secure storage | Yes | User logs out and you plan to reinitialize with new credentials |
| `destroyEngine()` | Fully tears down the Flutter engine and frees all memory | No | Freeing memory, or after `clearSession()` when the user won't return to the SDK |

> **Important:** After `clearSession()`, pass a configuration with fresh tokens to your next `show()`. The engine is still alive but has no valid credentials — presenting without new tokens will fail.

## Recommended Usage

```ts
// User closes the SDK UI → onClose fires and show() resolves
Rolla.addListener('onClose', (e) => {
  // Engine stays alive — fast re-launch next time
});

// User logs out of your app
async function logout() {
  await Rolla.clearSession(configuration); // warms a cold engine first, then clears
  await Rolla.destroyEngine();             // tear the engine down only after the clear resolved
}
```

> **Order matters on logout.** `clearSession` completes asynchronously — `await` it, then call `destroyEngine()`, never the other way round. Destroying the engine first cancels the pending clear, and the session data silently survives. Passing the configuration to `clearSession` covers the user who logs out before ever opening the SDK in this app session: the wrapper warms the engine, clears, and only then resolves — see [Token Management → Clearing the Session](06-token-management.md#clearing-the-session).

---

**Previous:** [Token Management](06-token-management.md) | **Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
