# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS and Android apps.

> **SDK Version:** `0.1.6`

---

## iOS Integration

[**Go to iOS Guide →**](ios/README.md)

| # | Section | Description |
|---|---------|-------------|
| 1 | [Prerequisites](ios/01-prerequisites.md) | iOS version, CocoaPods, Xcode |
| 2 | [CocoaPods Setup](ios/02-cocoapods-setup.md) | Add SDK dependency, build settings |
| 3 | [Permissions & Entitlements](ios/03-permissions-and-entitlements.md) | Info.plist, Bluetooth, Location, Mapbox, HealthKit |
| 4 | [Code Integration](ios/04-code-integration.md) | Import, configure, present, delegate |
| 5 | [Branding & Modules](ios/05-branding-and-modules.md) | Custom theming, 23 available modules |
| 6 | [Apple Health](ios/06-apple-health.md) | HealthKit integration, 14 data types |
| 7 | [Token Management](ios/07-token-management.md) | Auth lifecycle, refresh, session clear |
| 8 | [Engine Lifecycle](ios/08-engine-lifecycle.md) | Flutter engine, memory management |
| 9 | [Live Activities](ios/09-live-activities.md) | Lock Screen & Dynamic Island (iOS 16.1+) |
| 10 | [API Reference](ios/10-api-reference.md) | Rolla class, delegate, errors, close reasons |
| 11 | [Troubleshooting](ios/11-troubleshooting.md) | Common issues & support |

---

## Android Integration

[**Go to Android Guide →**](android/README.md)

| # | Section | Description |
|---|---------|-------------|
| 1 | [Prerequisites](android/01-prerequisites.md) | Android API level, Android Studio, Gradle |
| 2 | [Gradle Setup](android/02-gradle-setup.md) | Maven repos, SDK dependency, desugaring |
| 3 | [Permissions](android/03-permissions.md) | Internet, Mapbox token, manifest merger |
| 4 | [Code Integration](android/04-code-integration.md) | Import, configure, present, listener |
| 5 | [Branding & Modules](android/05-branding-and-modules.md) | Custom theming, module configuration |
| 6 | [Token Management](android/06-token-management.md) | Auth lifecycle, refresh, session clear |
| 7 | [Engine Lifecycle](android/07-engine-lifecycle.md) | Flutter engine, dismiss, memory |
| 8 | [API Reference](android/08-api-reference.md) | Rolla class, listener, errors, close reasons |
| 9 | [Troubleshooting](android/09-troubleshooting.md) | Common issues & support |

---

## Overview

The Rolla SDK provides a complete health and fitness experience embedded inside partner apps. Built on Flutter with native wrappers for iOS (Swift) and Android (Kotlin), the SDK manages its own UI, Bluetooth band communication, data syncing, and authentication lifecycle.

### Integration Flow

1. **Register the user** — one-time registration with Rolla's system (name, DOB, weight, height, gender, timezone)
2. **Obtain a token** — your backend calls Rolla's auth API to get a JWT access token
3. **Present the SDK** — initialize with the token and call `show()` — the SDK handles everything from there

### Environments

| Environment | Value | Use |
|-------------|-------|-----|
| Production | `"production"` | Release builds |
| Development | `"rnd"` | Development and QA |

If omitted, defaults to `"rnd"`.

---

## Support

For issues or questions, contact Rolla support at [support@rolla.app](mailto:support@rolla.app).
