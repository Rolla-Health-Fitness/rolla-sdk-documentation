# Engine Lifecycle

The SDK uses a Flutter engine internally. This section covers engine creation, dismissal, and cleanup strategies.

## Default Behavior

- **First `show(from:)`** — Creates the Flutter engine and presents the SDK UI.
- **`dismiss()`** — Dismisses the SDK UI but **keeps the engine alive** in the background. The next `show(from:)` call will present the SDK instantly in its last state (no reload).
- This is the recommended behavior for most apps.

## Destroying the Engine

If you need to free memory (e.g., on user logout or when the user won't return to the SDK for a while):

```swift
Rolla.destroyEngine()
```

- This fully tears down the Flutter engine and frees its resources.
- The next `show(from:)` call will create a fresh engine automatically (with a brief loading time).
- Call this **after** `dismiss()`, not while the SDK is presenting.

## Recommended Usage

```swift
// User taps "Close" inside the SDK → rollaDidClose is called
func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
    // Engine stays alive — fast re-launch next time
}

// User logs out of your app
func logout() {
    rolla.clearSession { _ in }
    Rolla.destroyEngine()
}
```

---

**Previous:** [Token Management](07-token-management.md) | **Next:** [Live Activities](09-live-activities.md) | **Home:** [README](README.md)
