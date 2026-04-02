# Code Integration

This section covers importing the SDK, creating a configuration, initializing and presenting the SDK, and implementing required delegate methods.

## 4.1 Import SDK

```swift
import RollaSDK
```

The SDK needs a **user access token** (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API **after** the user has logged in.

- **Typical flow:** User logs in the app → app calls backend → backend returns `access_token` (and optionally `refresh_token`, `expires_in`) → you pass that token into `RollaConfiguration` when opening the SDK.
- **When to fetch:** Before calling `rolla.show(from:)`. If the user is already logged in, use your existing session (e.g. stored token or refresh to get a new access token).
- **What to pass:** At minimum, the **access token** (string). For better behavior, also pass `refreshToken` and `tokenExpiresIn`.
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

> **Note:** You are responsible for authentication — the SDK only consumes the token you provide.

## 4.2 Create Configuration

```swift
let configuration = RollaConfiguration(
    token: "your-access-token",
    refreshToken: "your-refresh-token",  // Optional
    tokenExpiresIn: TimeInterval(3600),  // Optional: token expiry in seconds (TimeInterval)
    userId: "user-id",  // Optional: extracted from JWT if not provided
    partnerId: "your-partner-id",
    environment: "production",  // or "rnd" for development
    modules: nil,  // Optional: nil enables all modules
    branding: nil  // Optional: custom branding configuration
)
```

> **Note on `environment`:** The SDK supports two environments: `"production"` (live) and `"rnd"` (development/testing). Use `"rnd"` during development and QA to test against a sandbox environment without affecting production data. Switch to `"production"` for release builds. If omitted, the parameter defaults to `"rnd"`.

## 4.3 Initialize and Present SDK

```swift
let rolla = Rolla(configuration: configuration)
rolla.delegate = self
rolla.show(from: self)
```

## 4.4 Implement RollaDelegate

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

---

**Previous:** [Permissions & Entitlements](03-permissions-and-entitlements.md) | **Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
