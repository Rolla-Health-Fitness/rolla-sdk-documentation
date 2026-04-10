# Code Integration

This section covers importing the SDK, creating a configuration, initializing and presenting the SDK, and implementing required delegate methods.

## Import SDK

```swift
import RollaSDK
```

The SDK needs a **user access token** (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API **after** the user has logged in.

- **Typical flow:** User logs in the app → app calls backend → backend returns `access_token` (and optionally `refresh_token`, `expires_in`) → you pass that token into `RollaConfiguration` when opening the SDK.
- **When to fetch:** Before calling `rolla.show(from:)`. If the user is already logged in, use your existing session (e.g. stored token or refresh to get a new access token).
- **What to pass:** At minimum, the **access token** (string). For better behavior, also pass `refreshToken` and `tokenExpiresIn`.
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

> **Note:** You are responsible for authentication — the SDK only consumes the token you provide.

## Create Configuration

```swift
let configuration = RollaConfiguration(
    token: "your-access-token",
    refreshToken: "your-refresh-token",  // Optional
    tokenExpiresIn: TimeInterval(1800),  // Optional: token expiry in seconds (TimeInterval)
    userId: "user-id",  // Optional: extracted from JWT if not provided
    partnerId: "your-partner-id",
    environment: "production",  // or "rnd" for development
    modules: nil,  // Optional: nil enables all modules
    branding: nil  // Optional: custom branding configuration
)
```

### Environment Values

| Value | Description |
|-------|-------------|
| `"production"` | Live / release builds |
| `"rnd"` | Development and QA (sandbox environment) |

If omitted, defaults to `"rnd"`. Use `"rnd"` during development and QA to test against a sandbox environment without affecting production data. Switch to `"production"` for release builds.

> **Why `"rnd"`?** The name stands for "Research and Development" — it is the SDK's internal label for the non-production sandbox environment.

## Initialize and Present SDK

```swift
let rolla = Rolla(configuration: configuration)
rolla.delegate = self
rolla.show(from: self)
```

## Implement RollaDelegate

```swift
extension YourViewController: RollaDelegate {
    func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
        // Handle SDK close
        // Clean up any references
    }

    func rolla(_ rolla: Rolla, didFailWithError error: RollaError) {
        // Handle errors
        print("Rolla SDK error: \(error.localizedDescription)")
    }

    func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
        // Called when the SDK internally refreshes the token
        // Store the new token in your app's session for future use
    }

    func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
        // Called when the token has expired and the SDK cannot refresh it
        // You must fetch a new token from your backend and call:
        rolla.updateToken(token: newToken, refreshToken: newRefreshToken, expiresIn: newExpiresIn) { result in
            switch result {
            case .success:
                print("Token updated successfully")
            case .failure(let error):
                print("Failed to update token: \(error)")
            }
        }
    }
}
```

All delegate methods have default empty implementations, so you only need to implement the ones relevant to your use case.

## Threading

All public SDK methods dispatch to the main thread internally — you can safely call them from any thread:

| Method | Thread-safe | Notes |
|--------|:-----------:|-------|
| `show(from:)` | Yes | Dispatches to main queue before presenting |
| `dismiss()` | Yes | Dispatches to main queue before dismissing |
| `updateToken(...)` | Yes | Dispatches to main queue; completion fires on main thread |
| `clearSession(...)` | Yes | Dispatches to main queue; completion fires on main thread |

**Delegate callbacks** also arrive on the main thread. Flutter's platform channel delivers messages on the main thread, and the SDK does not re-dispatch to a background queue. You can safely update your UI directly inside delegate methods like `rollaDidClose(_:reason:)` or `rolla(_:didFailWithError:)`.

> **Summary:** You do not need to wrap any SDK call or delegate handler in `DispatchQueue.main.async` — the SDK handles this for you.

## Cross-Platform Note: `tokenExpiresIn` Type

On iOS, `tokenExpiresIn` is a `TimeInterval` (a `Double` representing seconds). On Android, it is an `Int` (seconds).

If you maintain a shared backend or cross-platform token logic, be aware of this difference — passing a floating-point value where an integer is expected (or vice versa) can cause subtle bugs. Both platforms interpret the value as **seconds until expiry**.

The same applies to the `expiresIn` parameter in the `rollaDidRefreshToken` delegate callback (`TimeInterval?` on iOS, `Int?` on Android).

---

**Previous:** [Permissions & Entitlements](03-permissions-and-entitlements.md) | **Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
