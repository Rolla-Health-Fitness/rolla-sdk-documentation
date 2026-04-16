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

### Destroying the Engine

If you need to free memory (e.g., on logout), call:

```kotlin
Rolla.destroyEngine()
```

The engine will be recreated automatically on the next `show()` call.

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
    rolla.clearSession { }
    Rolla.destroyEngine()
}
```

---

**Previous:** [Token Management](06-token-management.md) | **Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
