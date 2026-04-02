# Rolla SDK — iOS Integration Guide

A complete, step-by-step guide to integrating the Rolla SDK into your iOS application. This guide covers everything from initial setup through advanced features like Live Activities and token management.

## Table of Contents

1. **[Prerequisites](01-prerequisites.md)** — iOS version requirements, CocoaPods, Xcode, and partner credentials
2. **[CocoaPods Setup](02-cocoapods-setup.md)** — Add the SDK via CocoaPods, configure build settings, install dependencies
3. **[Permissions & Entitlements](03-permissions-and-entitlements.md)** — Configure Info.plist and .entitlements files for permissions, Bluetooth, location, Mapbox, and HealthKit
4. **[Code Integration](04-code-integration.md)** — Import the SDK, create configuration, initialize, present, and implement delegate callbacks
5. **[Branding & Modules](05-branding-and-modules.md)** — Custom branding with RollaBranding, module configuration, and the complete list of 23 available modules
6. **[Apple Health Integration](06-apple-health.md)** — Set up Apple Health, supported data types (14 types), availability, and platform notes
7. **[Token Management](07-token-management.md)** — Token lifecycle, delegate callbacks, pushing new tokens, clearing sessions
8. **[Engine Lifecycle](08-engine-lifecycle.md)** — Flutter engine initialization, dismissal, memory management, and recommended usage patterns
9. **[Live Activities](09-live-activities.md)** — Real-time workout tracking on Lock Screen and Dynamic Island (iOS 16.1+), full widget setup with SwiftUI code
10. **[API Reference](10-api-reference.md)** — Rolla class methods, RollaDelegate protocol, RollaError enum, RollaCloseReason enum
11. **[Troubleshooting](11-troubleshooting.md)** — Solutions for common issues: SDK startup, Apple Health, build errors, and support contact info

---

## Quick Start

1. Start with [Prerequisites](01-prerequisites.md) to verify your environment
2. Follow [CocoaPods Setup](02-cocoapods-setup.md) to add the SDK
3. Configure [Permissions & Entitlements](03-permissions-and-entitlements.md)
4. Implement [Code Integration](04-code-integration.md) in your app
5. Optionally enable [Apple Health](06-apple-health.md) and [Live Activities](09-live-activities.md)

For detailed API information, see [API Reference](10-api-reference.md).
For common issues, see [Troubleshooting](11-troubleshooting.md).

---

## Support

For issues or questions, contact Rolla support.
