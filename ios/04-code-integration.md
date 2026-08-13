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
    partnerId: "your-partner-id",
    environment: "production"  // or "rnd" for development
)
```

These are the identity and auth essentials. `RollaConfiguration` also takes `branding`, `language`, `disabledModules`, `disabledDataSources`, `userId`, and `showOptionsButton` — see [Configuration](05-configuration.md) for the full reference.

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

Instead of `show(from:)`, `openScreen` opens the SDK directly on a specific screen (insights, activity history, goals, …) — for example from your own menu entries — see [Host-Driven Navigation](10-api-reference.md#host-driven-navigation).

## Implement RollaDelegate

```swift
extension YourViewController: RollaDelegate {
    func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
        // Handle SDK close
        // Clean up any references
    }

    func rollaDidFailWithError(_ rolla: Rolla, error: RollaError) {
        // Handle errors
        print("Rolla SDK error: \(error.localizedDescription)")
    }

    func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
        // Called when the SDK internally refreshes the token
        // Store the new token in your app's session for future use
    }

    func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
        // Called when the token has expired and the SDK cannot refresh it.
        // Obtain fresh tokens from the Rolla auth API (/api/login), directly
        // or through your backend, and call:
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

The four above are the presentation and token callbacks — the minimum for a production integration. `RollaDelegate` has twelve more optional methods that push SDK events to your app (activity lifecycle, sync results, band pairing and live link state, primary source, goals, profile) — see the [RollaDelegate reference](10-api-reference.md#rolladelegate-protocol) and its [Host Events](10-api-reference.md#host-events) delivery semantics.

## Threading

The SDK is thread-safe at its public surface, so you never have to think about threads when calling it:

- **Every method on a `Rolla` instance** — presentation, token, and headless methods alike — dispatches to the main queue internally. Call them from any thread.
- **Every callback** — all `RollaDelegate` methods and every completion handler — arrives on the main thread. You can update your UI directly inside them.
- **The one exception is the static `Rolla.destroyEngine()`**: it runs synchronously on the calling thread, with no internal dispatch — call it from the main thread.

> **Summary:** You never need `DispatchQueue.main.async` around an SDK call or delegate handler.

## Cross-Platform Note: `tokenExpiresIn` Type

Both platforms mean the same thing — **seconds until expiry** — but the declared type follows each platform's idiom: `TimeInterval?` (a `Double` number of seconds) on iOS, `Int?` on Android. The same applies to the `expiresIn` parameter in the `rollaDidRefreshToken` delegate callback. There is no unit difference: if your backend returns `expires_in` in seconds, pass it straight through on both platforms.

---

**Previous:** [Permissions & Entitlements](03-permissions-and-entitlements.md) | **Next:** [Configuration](05-configuration.md) | **Home:** [README](README.md)
