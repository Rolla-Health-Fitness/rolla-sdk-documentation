# Prerequisites

Before you begin integrating the Rolla SDK into your iOS app, ensure you have the following:

1. **iOS 14.0 or later** — Minimum deployment target
2. **CocoaPods installed** — Dependency manager for iOS projects. Swift Package Manager support is planned; CocoaPods is currently required because Mapbox has not fully migrated to SPM.
3. **Xcode 14.0 or later** — Apple's integrated development environment
4. **Partner ID and API credentials from Rolla** — Required to authenticate SDK requests

This guide walks you through installing the Rolla SDK via CocoaPods, configuring native iOS permissions and entitlements, integrating the SDK into your code, and (optionally) setting up Live Activities for real-time workout tracking on the Lock Screen.

## SDK Binary Size

The Rolla SDK embeds a Flutter engine and several native frameworks (Mapbox maps, Bluetooth/band communication, HealthKit integration, and more), which increase your app's download size after App Store thinning (the raw framework files are larger due to multi-architecture universal binaries). The exact amount depends on your build — see below to measure it.

Key contributors:
- **Flutter engine** — the cross-platform UI runtime
- **Mapbox frameworks** — map rendering for activity tracking
- **Rolla SDK core** — the SDK's own compiled Dart application and native plugins

> **Note:** Exact size varies by SDK version and App Store processing. Use Xcode's "App Thinning Size Report" (`Product > Archive > Distribute App > App Thinning`) for a precise measurement with your specific build.

---

**Previous:** [Quick Start](00-quick-start.md) | **Next:** [CocoaPods Setup](02-cocoapods-setup.md) | **Home:** [README](README.md)
