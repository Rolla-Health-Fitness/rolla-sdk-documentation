# API Reference

This section provides a comprehensive reference of the Rolla SDK's public API, including the main Rolla class, delegate protocol, and error types.

## 11. Native API Reference

### Rolla Class

| Method / Property | Description |
|-------------------|-------------|
| `init(configuration: RollaConfiguration)` | Initialize with a configuration |
| `var delegate: RollaDelegate?` | Set the delegate for callbacks |
| `var isPresenting: Bool` | Whether the SDK is currently visible |
| `show(from: UIViewController)` | Present the SDK modally |
| `dismiss()` | Dismiss the SDK UI (engine stays alive) |
| `updateToken(token:refreshToken:expiresIn:completion:)` | Push fresh credentials to the SDK |
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

## 12. Error Handling

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

## 13. Close Reasons

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

---

**Previous:** [Live Activities](09-live-activities.md) | **Next:** [Troubleshooting](11-troubleshooting.md) | **Home:** [README](README.md)
