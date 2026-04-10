# API Reference

Complete reference for the Rolla SDK classes, methods, interfaces, error types, and close reasons.

## Native API Reference (Android)

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `Rolla(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var listener: RollaListener?` | Set the listener for callbacks |
| `val isPresenting: Boolean` | Whether the SDK is currently visible |
| `show(activity: Activity)` | Present the SDK from an Activity |
| `show(fragment: Fragment)` | Present the SDK from a Fragment |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token, refreshToken?, expiresIn?, callback?)` | Push fresh credentials to the SDK |
| `clearSession(callback?)` | Clear all persisted session data |
| `companion fun destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaListener Interface

| Method | Description |
|--------|-------------|
| `onRollaClosed(rolla, reason)` | Called when the SDK UI is dismissed |
| `onRollaError(rolla, error)` | Called when an error occurs |
| `onTokenRefreshed(rolla, token, refreshToken?, expiresIn?)` | Called when the SDK refreshes tokens internally |
| `onTokenExpired(rolla)` | Called when the host app must provide new tokens |

All methods have default empty implementations.

## Error Handling

The SDK provides detailed error information through RollaError:

```kotlin
sealed class RollaError(val code: String, val message: String) : Exception(message) {
    class EngineFailedToStart
    class InitializationFailed(details: String)
    class FlutterError(errorCode: String, errorMessage: String)
    class AlreadyPresenting
    class InvalidContext
    class Underlying(throwable: Throwable)
    class Unknown(details: String?)
}
```

## Close Reasons

The SDK provides close reasons through RollaCloseReason:

```kotlin
sealed class RollaCloseReason {
    data class FlutterRequested(val reason: String?)
    data object HostNavigationBack
    data object HostModalDismiss
    data object Programmatic
    data object HostStackReplaced
    data object Unknown
}
```

---

**Previous:** [Engine Lifecycle](07-engine-lifecycle.md) | **Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
