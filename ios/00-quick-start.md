# Quick Start — iOS

Get the Rolla SDK running in your iOS app in under 10 minutes.

> **This guide covers the minimal integration.** For branding, modules, Apple Health, Live Activities, and other features, see the [full documentation](README.md).

## Prerequisites

- **iOS 14.0+** deployment target
- **Xcode 14.0+**
- **CocoaPods** installed (`sudo gem install cocoapods`)
- **Partner ID** from Rolla (contact [support@rolla.app](mailto:support@rolla.app))
- **Supported runtime configurations**
   - **Release** configuration for physical iPhone devices
   - **Debug** configuration for iPhone simulators

   > **Note:** Hardware-backed features (Bluetooth, etc.) will only work on physical devices.

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the full flow (`/api/register` → `/api/login` → tokens).

## 1. Install the SDK

Add to your `Podfile`:

```ruby
platform :ios, '14.0'

source 'https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios.git'
source 'https://cdn.cocoapods.org/'

target 'YourApp' do
  use_frameworks!
  pod 'RollaSDK', '0.1.12'
end
```

> Check the [iOS release repo](https://github.com/Rolla-Health-Fitness/rolla-sdk-release-ios) for the latest version.

Then run:

```bash
pod install
```

Open the `.xcworkspace` file (not `.xcodeproj`). For build settings and troubleshooting, see [CocoaPods Setup](02-cocoapods-setup.md).

## 2. Configure and Present

Create a minimal `RollaConfiguration`, initialize the SDK, and present it:

```swift
import RollaSDK

class YourViewController: UIViewController {

    private var rolla: Rolla?

    func showRolla() {
        let configuration = RollaConfiguration(
            token: "your-access-token",       // JWT from POST /api/login
            refreshToken: "your-refresh-token", // From POST /api/login
            tokenExpiresIn: TimeInterval(1800),  // Seconds until token expires (TimeInterval)
            partnerId: "your-partner-id",
            environment: "rnd"                // Use "production" for release builds
        )

        let rolla = Rolla(configuration: configuration)
        rolla.delegate = self
        self.rolla = rolla
        rolla.show(from: self)
    }
}
```

## 3. Handle Callbacks

Implement `RollaDelegate` to respond to SDK events:

```swift
extension YourViewController: RollaDelegate {

    func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
        // SDK was dismissed — clean up your reference
        self.rolla = nil
    }

    func rollaDidFailWithError(_ rolla: Rolla, error: RollaError) {
        print("Rolla SDK error: \(error.localizedDescription)")
    }

    func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
        // Token expired and SDK cannot refresh it.
        // Fetch a new token from your backend, then push it to the SDK:
        YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn in
            rolla.updateToken(
                token: newToken,
                refreshToken: newRefreshToken,
                expiresIn: TimeInterval(expiresIn)
            ) { result in
                switch result {
                case .success:
                    print("Token updated")
                case .failure(let error):
                    print("Token update failed: \(error)")
                }
            }
        }
    }

    func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
        // SDK refreshed the token internally — store it for future use
        SessionManager.shared.updateToken(token, refreshToken: refreshToken, expiresIn: expiresIn)
    }
}
```

## 4. Logout Cleanup

When the user logs out, clear the SDK session:

```swift
rolla?.clearSession { result in
    switch result {
    case .success:
        print("SDK session cleared")
    case .failure(let error):
        print("Failed to clear session: \(error)")
    }
}
```

## Complete Example

Here is a single, copy-pasteable `UIViewController` covering configuration, presentation, callbacks, and cleanup:

```swift
import UIKit
import RollaSDK

class RollaViewController: UIViewController, RollaDelegate {

    private var rolla: Rolla?

    // MARK: - Present the SDK

    func showRolla(token: String) {
        let config = RollaConfiguration(
            token: token,
            refreshToken: refreshToken,
            tokenExpiresIn: TimeInterval(1800),
            partnerId: "your-partner-id",
            environment: "rnd"               // "production" for release builds
        )

        let rolla = Rolla(configuration: config)
        rolla.delegate = self
        self.rolla = rolla
        rolla.show(from: self)
    }

    // MARK: - RollaDelegate

    func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
        self.rolla = nil
    }

    func rollaDidFailWithError(_ rolla: Rolla, error: RollaError) {
        print("Rolla error: \(error.localizedDescription)")
    }

    func rollaDidRequestTokenRefresh(_ rolla: Rolla) {
        YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn in
            rolla.updateToken(
                token: newToken,
                refreshToken: newRefreshToken,
                expiresIn: TimeInterval(expiresIn)
            ) { result in
                if case .failure(let err) = result {
                    print("Token update failed: \(err)")
                }
            }
        }
    }

    func rollaDidRefreshToken(_ rolla: Rolla, token: String, refreshToken: String?, expiresIn: TimeInterval?) {
        SessionManager.shared.updateToken(token, refreshToken: refreshToken, expiresIn: expiresIn)
    }

    // MARK: - Logout

    func logout() {
        rolla?.clearSession { _ in }
        rolla = nil
    }
}
```

## Next Steps

- **Permissions:** Set up Bluetooth, location, and HealthKit entitlements — [Permissions & Entitlements](03-permissions-and-entitlements.md)
- **Configuration:** Customize branding, force a UI language, and control module and data-source visibility — [Configuration](05-configuration.md)
- **Apple Health:** Enable health data sync — [Apple Health Integration](06-apple-health.md)
- **Token details:** Full token lifecycle and edge cases — [Token Management](07-token-management.md)
- **API Reference:** All methods, delegates, and enums — [API Reference](10-api-reference.md)

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
