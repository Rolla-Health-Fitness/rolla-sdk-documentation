# Engine Lifecycle

Understand how the Rolla SDK manages its internal Flutter engine and how to control its lifecycle for optimal performance and memory management.

## 9. Programmatic Dismiss

You can dismiss the SDK programmatically:

```kotlin
rolla.dismiss()
```

## 10. Engine Lifecycle

The SDK manages a Flutter engine internally. By default, the engine stays alive for faster re-launch.

### 10.1 Default Behavior

- **First `show(activity)`** — Creates the Flutter engine and presents the SDK UI.
- **`dismiss()`** — Dismisses the SDK UI but keeps the engine alive. The next `show()` will present the SDK instantly in its last state.

### 10.2 Destroying the Engine

If you need to free memory (e.g., on logout), call:

```kotlin
Rolla.destroyEngine()
```

The engine will be recreated automatically on the next `show()` call.

### 10.3 Recommended Usage

```kotlin
// User logs out
fun logout() {
    rolla.clearSession { }
    Rolla.destroyEngine()
}
```

---

**Previous:** [Token Management](06-token-management.md) | **Next:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
