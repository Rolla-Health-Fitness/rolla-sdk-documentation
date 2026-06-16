# API Reference

This section provides a comprehensive reference of the Rolla SDK's public API, including the main Rolla class, delegate protocol, and error types.

## Native API Reference

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `init(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var delegate: RollaDelegate?` | Set the delegate for callbacks |
| `var isPresenting: Bool` | Whether the SDK is currently visible |
| `show(from: UIViewController)` | Present the SDK modally |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token:refreshToken:expiresIn:completion:)` | Push fresh credentials to the SDK. The `completion` handler is optional but recommended — if omitted, the update still executes but your app receives no success/failure feedback. |
| `clearSession(completion:)` | Clear all persisted session data |
| `static destroyEngine()` | Destroy the Flutter engine and free memory |

### RollaDelegate Protocol

| Method | Description |
|--------|-------------|
| `rollaDidClose(_:reason:)` | Called when the SDK UI is dismissed |
| `rolla(_:didFailWithError:)` | Called when an error occurs |
| `rollaDidRefreshToken(_:token:refreshToken:expiresIn:)` | Called when the SDK refreshes tokens internally |
| `rollaDidRequestTokenRefresh(_:)` | Called when the host app must provide new tokens |

All methods have default empty implementations.

## RollaConfiguration

The `RollaConfiguration` struct defines all parameters for SDK initialization. See [Code Integration](04-code-integration.md) for usage examples.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `token` | `String` | Yes | — | JWT access token from `POST /api/login` |
| `partnerId` | `String` | Yes | — | Partner identifier provided by Rolla |
| `refreshToken` | `String?` | No | `nil` | Refresh token for automatic credential renewal |
| `tokenExpiresIn` | `TimeInterval?` | No | `nil` | Token lifetime in seconds. Note: Android uses `Int` for this parameter |
| `userId` | `String?` | No | Extracted from JWT | User identifier for local data namespacing (per-user storage isolation); defaults to the `sub` claim in the JWT if not provided. Not sent as a request header |
| `environment` | `String?` | No | `"rnd"` | Target environment. See [Code Integration](04-code-integration.md) for available values |
| `disabledModules` | `Set<RollaDisabledModule>` | No | `[]` (nothing disabled) | Modules whose entire UI is hidden across the SDK. See [Branding and Modules](05-branding-and-modules.md#module-configuration) and the [`RollaDisabledModule`](#rolladisabledmodule) values below |
| `branding` | `RollaBranding?` | No | `nil` | Custom branding configuration. See [Branding and Modules](05-branding-and-modules.md) |
| `showSettingsButton` | `Bool` | No | `true` | Render a Settings button on the Home screen, below the Metrics list. Tapping it opens a bottom sheet with shortcuts to Data Sources and Goals. Defaults to true because most partners need this button. |
| `removeRollaBandReferences` | `Bool` | No | `true` | When `true` (the default), the SDK UI uses generic "fitness device" wording. Set to `false` to show Rolla Band-specific references. See [Branding and Modules](05-branding-and-modules.md#rolla-band-references) |

### RollaDisabledModule

`disabledModules` accepts a set of `RollaDisabledModule` values. Each value passed hides that module's entire UI everywhere it appears in the SDK. Currently supported:

| Value | Hides |
|-------|-------|
| `.weight` | The Weight tracking module |
| `.bloodPressure` | The Blood Pressure tracking module |

More modules will become disable-able in future releases. Pass `[]` (or omit the parameter) to keep every module enabled.

## Error Handling

The SDK provides detailed error information through `RollaError`:

```swift
public enum RollaError: Error {
    case engineFailedToStart
    case initializationFailed(String)
    case flutterError(code: String, message: String)
    case alreadyPresenting
    case invalidPresentationContext
    case underlying(Error)
    case unknown
}
```

## Close Reasons

The SDK provides close reasons through `RollaCloseReason`:

```swift
enum RollaCloseReason {
    case flutterRequested(reason: String?)
    case hostNavigationBack
    case hostModalDismiss
    case programmatic
    case hostStackReplaced
    case unknown
}
```

## Error Recovery Guide

Recommended host app actions for each `RollaError` case:

| Error Case | Code | Meaning | Host App Recovery |
|------------|------|---------|-------------------|
| `.engineFailedToStart` | `ENGINE_FAILED` | Flutter engine failed to start | Retry after a delay. If persistent, call `destroyEngine()` and re-initialize. Check device memory. |
| `.initializationFailed(String)` | `INIT_FAILED` | SDK init failed — detail string explains why | Check detail message. Common causes: invalid credentials, network failure, expired token. Verify config and retry. |
| `.flutterError(code:message:)` | `FLUTTER_ERROR` | Internal Flutter error | Log code and message. Retry. If persistent, `destroyEngine()` and re-init. Report to Rolla support with error code. |
| `.alreadyPresenting` | `ALREADY_PRESENTING` | `show()` called while SDK is already visible | Check `isPresenting` before calling `show()`. Call `dismiss()` first if needed. |
| `.invalidPresentationContext` | `INVALID_CONTEXT` | View controller not in window hierarchy | Ensure the view controller is visible and in the hierarchy before calling `show(from:)`. |
| `.underlying(Error)` | `UNDERLYING_ERROR` | Wraps a native error | Inspect the wrapped error. Handle based on underlying cause. |
| `.unknown` | `UNKNOWN` | Unrecognized error | Log all details. Retry. Report to Rolla support if persistent. |

## Close Reason Reference

When each `RollaCloseReason` is triggered:

| Close Reason | When Triggered |
|-------------|----------------|
| `.flutterRequested(reason:)` | SDK's internal UI initiated the close (e.g., user tapped close/done). Optional `reason` may provide context. |
| `.hostNavigationBack` | User pressed back gesture or navigation back. |
| `.hostModalDismiss` | User dismissed the modal via swipe-down gesture. |
| `.programmatic` | Host app called `dismiss()` programmatically. |
| `.hostStackReplaced` | Host app replaced the navigation stack while SDK was presenting. This can occur if the host app programmatically pops multiple view controllers or pushes a new root while the SDK is on screen. |
| `.unknown` | Close reason could not be determined. |

---

**Previous:** [Live Activities](09-live-activities.md) | **Next:** [Troubleshooting](11-troubleshooting.md) | **Home:** [README](README.md)
