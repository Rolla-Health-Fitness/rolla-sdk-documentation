# Engine Lifecycle

Understand how the Rolla SDK manages its internal Flutter engine and how to control its lifecycle for optimal performance and memory management.

## Programmatic Dismiss

You can dismiss the SDK programmatically:

```kotlin
rolla.dismiss()
```

## Engine Lifecycle

The SDK manages a Flutter engine internally. By default, the engine stays alive for faster re-launch.

### Default Behavior

- **First `show(activity)`** — Creates the Flutter engine and presents the SDK UI.
- **`dismiss()`** — Dismisses the SDK UI but keeps the engine alive. The next `show()` will present the SDK instantly in its last state.

### Warming Up the Engine

The engine also starts automatically on the first headless call (`getBandBatteryLevel`, `getPairedBandInfo`, `syncHealthData` — see [API Reference](08-api-reference.md#headless-methods)), so none of them require a prior `show()`. Call `warmUpEngine()` to pay the start-up cost early — typically right after login — so the first `show()` presents instantly:

```kotlin
rolla.warmUpEngine(context) { result ->
    // Engine configured and ready — headless calls and host events now have zero start-up latency.
}
```

Safe to call repeatedly: a repeat call for the same user is a no-op that preserves the session. The warmed engine holds memory until `destroyEngine()`.

### Destroying the Engine

If you need to free memory (e.g., on logout), call:

```kotlin
Rolla.destroyEngine()
```

The engine will be recreated automatically on the next `show()` call. Host-event delivery (see [API Reference](08-api-reference.md#host-events)) also stops here — events flow for the engine's lifetime.

Destroying the engine is also how a new `RollaConfiguration` is applied — a changed language, branding, or module set takes effect on the next engine start. See [Configuration](05-configuration.md).

### `clearSession` vs `destroyEngine`

| Method | What It Does | Engine Stays Alive | When to Use |
|--------|-------------|:------------------:|-------------|
| `clearSession()` | Purges stored tokens and auth metadata from secure storage | Yes | User logs out and you plan to reinitialize with new credentials |
| `destroyEngine()` | Fully tears down the Flutter engine and frees all memory | No | Freeing memory, or after `clearSession()` when the user won't return to the SDK |

> **Important:** After calling `clearSession()`, you must create a new `Rolla(configuration)` with fresh tokens before calling `show()` again. The engine is still alive but has no valid credentials — calling `show()` without reinitializing will fail.

### Recommended Usage

```kotlin
// User logs out
fun logout() {
    rolla.clearSession { result ->
        // Tear the engine down only after the session is cleared.
        Rolla.destroyEngine()
    }
}
```

> **Order matters on logout.** `clearSession` completes asynchronously — call `Rolla.destroyEngine()` from its callback, never immediately after it. Destroying the engine first cancels the pending clear, and the session data silently survives.

---

**Previous:** [Token Management](06-token-management.md) | **Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
