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
| `updateToken(token, refreshToken?, expiresIn?, callback?)` | Push fresh credentials to the SDK. The `callback` is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. |
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

## RollaConfiguration

The `RollaConfiguration` class defines all parameters for SDK initialization. See [Code Integration](04-code-integration.md) for usage examples.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `String` | Yes | — | JWT access token from `POST /api/login` |
| `partnerId` | `String` | Yes | — | Partner identifier provided by Rolla |
| `refreshToken` | `String?` | No | `null` | Refresh token for automatic credential renewal |
| `tokenExpiresIn` | `Int?` | No | `null` | Token lifetime in seconds. Note: iOS uses `TimeInterval` (Double) for this parameter |
| `userId` | `String?` | No | Extracted from JWT | User identifier for local data namespacing (per-user storage isolation); defaults to the `sub` claim in the JWT if not provided. Not sent as a request header |
| `environment` | `String` | No | `"rnd"` | Target environment. See [Code Integration](04-code-integration.md) for available values |
| `disabledModules` | `Set<RollaDisabledModule>` | No | `emptySet()` (nothing disabled) | Modules whose entire UI is hidden across the SDK. See [Branding and Modules](05-branding-and-modules.md#module-configuration) and the [`RollaDisabledModule`](#rolladisabledmodule) values below |
| `disabledDataSources` | `Set<RollaDataSource>` | No | `emptySet()` (all offered) | Data sources whose connect option is hidden wherever the user picks a source to connect. See [Branding and Modules](05-branding-and-modules.md#data-source-configuration) and the [`RollaDataSource`](#rolladatasource) values below |
| `branding` | `RollaBranding?` | No | `null` | Custom branding configuration. See [Branding and Modules](05-branding-and-modules.md) |
| `showSettingsButton` | `Boolean` | No | `true` | Render a Settings button on the Home screen, below the Metrics list. Tapping it opens a bottom sheet with shortcuts to Data Sources and Goals. Defaults to true because most partners need this button. |
| `removeRollaBandReferences` | `Boolean` | No | `true` | When `true` (the default), the SDK UI uses generic "fitness device" wording. Set to `false` to show Rolla Band-specific references. See [Branding and Modules](05-branding-and-modules.md#rolla-band-references) |

### RollaDisabledModule

`disabledModules` accepts a set of `RollaDisabledModule` values. Each value passed hides that module's entire UI everywhere it appears in the SDK. Currently supported:

| Value | Hides |
|-------|-------|
| `RollaDisabledModule.WEIGHT` | The Weight tracking module |
| `RollaDisabledModule.BLOOD_PRESSURE` | The Blood Pressure tracking module |

More modules will become disable-able in future releases. Pass `emptySet()` (or omit the parameter) to keep every module enabled.

### RollaDataSource

`disabledDataSources` accepts a set of `RollaDataSource` values. Each value passed hides that source's connect option wherever the user picks a data source to connect (the Data Sources screen and the onboarding data-source step). A source the user has already connected stays visible for viewing/disconnecting; only new connections are suppressed. If you disable every source, the Rolla Band remains available as a floor.

| Value | Hides |
|-------|-------|
| `RollaDataSource.BAND` | The Rolla Band pairing option |
| `RollaDataSource.GARMIN` | Garmin Connect |
| `RollaDataSource.OURA` | Oura |
| `RollaDataSource.APPLE_HEALTH` | Apple Health (iOS only) |
| `RollaDataSource.HEALTH_CONNECT` | Health Connect |

Pass `emptySet()` (or omit the parameter) to offer every data source.

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
| `HostStackReplaced` | Host app replaced the navigation/activity stack while SDK was presenting. This can occur if the host app finishes the current Activity, launches a new task, or clears the back stack while the SDK is on screen. |
| `Unknown` | Close reason could not be determined. |

---

**Previous:** [Engine Lifecycle](07-engine-lifecycle.md) | **Next:** [Troubleshooting](09-troubleshooting.md) | **Home:** [README](README.md)
