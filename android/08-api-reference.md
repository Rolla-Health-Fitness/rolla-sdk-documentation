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

## Error Recovery Guide

Recommended host app actions for each `RollaError` subclass:

| Error Class | Code | Meaning | Host App Recovery |
|-------------|------|---------|-------------------|
| `EngineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `InitializationFailed(details)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `FlutterError(errorCode, errorMessage)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `AlreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `InvalidContext` | `INVALID_CONTEXT` | Activity/Fragment not in a valid state | Ensure the Activity is resumed or the Fragment is attached before calling `show()`. |
| `Underlying(Throwable)` | `UNDERLYING_ERROR` | Wraps a platform-native error | Inspect the wrapped throwable. Handle based on underlying cause. |
| `Unknown(details?)` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## Close Reason Reference

When each `RollaCloseReason` subclass is triggered:

| Close Reason | When Triggered |
|-------------|----------------|
| `FlutterRequested(reason)` | SDK's internal UI initiated the close (e.g., user tapped close/done). Optional `reason` may provide context. |
| `HostNavigationBack` | User pressed the system back button. |
| `HostModalDismiss` | User dismissed the modal via gesture or system action. |
| `Programmatic` | Host app called `dismiss()` programmatically. |
| `HostStackReplaced` | Host app replaced the navigation/activity stack while SDK was presenting. |
| `Unknown` | Close reason could not be determined. |

---

**Previous:** [Engine Lifecycle](07-engine-lifecycle.md) | **Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
