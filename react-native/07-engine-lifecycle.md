# Engine Lifecycle

The Rolla modal is rendered by an embedded Flutter engine on the native side. The wrapper exposes the same lifecycle controls as the native SDKs.

The model is identical to [iOS Engine Lifecycle](../ios/08-engine-lifecycle.md) and [Android Engine Lifecycle](../android/07-engine-lifecycle.md) — this page covers the JS-side surface.

## Default behaviour

When you call `Rolla.show()` for the first time, the SDK initializes the Flutter engine and renders the modal. On dismiss, the engine stays **warm in memory** — subsequent `Rolla.show()` calls reuse it and feel instant.

The trade-off:
- **Warm engine** — instant `show()`, ~30 MB resident memory
- **Destroyed engine** — first `show()` costs 1–2 s of cold start, memory reclaimed

For most apps the warm-engine default is correct. Reclaim memory only when you know the user is done with the SDK for a long time (e.g. after they navigate to a part of the app that doesn't need Rolla).

## Programmatic dismiss

```ts
await Rolla.dismiss();
```

Closes the modal if it is on screen. The pending `Rolla.show()` promise resolves with `{ reason: 'programmatic' }`. Safe to call when no modal is presented (it is a no-op).

## Destroy the engine

```ts
await Rolla.destroyEngine();
```

Tears down the Flutter engine and reclaims its memory. The next `Rolla.show()` will incur a cold start. Safe to call when the modal is not presented; if the modal is presented, the SDK dismisses it first.

**Recommended trigger points:**
- After the user logs out (paired with `clearSession()`)
- When entering a memory-intensive part of your app (e.g. video playback)
- When the OS issues a memory warning (`AppState`'s `memoryWarning` on iOS, `onLowMemory` on Android)

## Check presentation state

```ts
const isOpen = await Rolla.isPresenting();
```

Returns `true` while the modal is on screen. Useful for guarding navigation actions (don't push a new screen while the modal is presenting).

## Logout flow

```ts
async function logout() {
  await yourSession.clear();
  await Rolla.clearSession();
  await Rolla.destroyEngine();
}
```

`clearSession()` removes SDK-persisted tokens. `destroyEngine()` reclaims the Flutter engine's memory. Both are idempotent.

---

**Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
