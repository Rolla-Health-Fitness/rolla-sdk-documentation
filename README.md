# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS and Android apps.

> **Latest SDK Version:** `0.1.6`

---

## iOS Integration

[**Go to iOS Guide →**](ios/README.md)

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](ios/00-quick-start.md) | Minimal integration in under 10 minutes |
| 1 | [Prerequisites](ios/01-prerequisites.md) | iOS version, CocoaPods, Xcode |
| 2 | [CocoaPods Setup](ios/02-cocoapods-setup.md) | Add SDK dependency, build settings |
| 3 | [Permissions & Entitlements](ios/03-permissions-and-entitlements.md) | Info.plist, Bluetooth, Location, Mapbox, HealthKit |
| 4 | [Code Integration](ios/04-code-integration.md) | Import, configure, present, delegate |
| 5 | [Branding & Modules](ios/05-branding-and-modules.md) | Custom theming, available modules |
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
| 0 | [Quick Start](android/00-quick-start.md) | Minimal integration in under 10 minutes |
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

## Auth API — SDK Authentication

[**Go to Auth API Guide →**](sdk-auth-api/README.md)

| # | Section | Description |
|---|---------|-------------|
| 1 | [Overview](sdk-auth-api/01-overview.md) | Auth architecture, base URLs, environments, onboarding |
| 2 | [Authentication](sdk-auth-api/02-authentication.md) | Register users, log in, obtain tokens, refresh tokens |
| 3 | [Error Handling](sdk-auth-api/03-error-handling.md) | Error format, status codes, retry strategies, checklist |

> **Server-to-server data integration:** Rolla also offers a Partner API for backend-to-backend access to user health data, activity data, and user management. This is separate from the SDK integration. Contact [support@rolla.app](mailto:support@rolla.app) for Partner API access.

---

## Platform Capabilities

Feature support comparison between iOS and Android (SDK version `0.1.6`).

| Feature | iOS | Android | Notes |
|---------|:---:|:-------:|-------|
| Core SDK (present, dismiss, token management) | Yes | Yes | |
| Custom Branding & Modules (all modules currently always enabled) | Yes | Yes | All modules currently always enabled |
| Apple Health (HealthKit) | Yes | **No** | 14 data types, read-only |
| **Health Connect** | **No** | **No** | **Not implemented on either platform. Android has no health platform integration — all health data comes from the Rolla band only.** |
| Live Activities (Lock Screen / Dynamic Island) | Yes | **No** | Requires iOS 16.1+ |
| Bluetooth Band Sync | Yes | Yes | Background mode on iOS; foreground service on Android |
| Blood Pressure | Yes | No | Via Apple Health only |
| Weight Tracking | Yes | No | Via Apple Health (Body Mass) only |
| Workout Routes | Yes | No | Via Apple Health only |
| Cycling Cadence / Power | Yes | No | Apple Health, iOS 17+ |
| Running Speed | Yes | No | Apple Health, iOS 16+ |
| Mapbox Maps | Yes | Yes | Token via `Info.plist` (iOS) / `strings.xml` (Android) |
| Background Location | Yes | Yes | |

> **Critical limitation:** Android has **no** Health Connect integration. On Android, all health data comes exclusively from the Rolla Bluetooth band. iOS users get data from both the band and Apple Health (14 data types). This is not feature parity.

## Version Compatibility

| Requirement | iOS | Android |
|-------------|-----|---------|
| **Min OS** | iOS 14.0 | API 24 (Android 7.0) |
| **IDE** | Xcode 14.0+ | Android Studio Hedgehog (2023.1)+ |
| **Dependency Manager** | CocoaPods | Gradle 8.0+ |
| **Language** | Swift | Kotlin (JVM target 11) |
| **Compile / Target SDK** | — | API 36 |
| **Core Library Desugaring** | — | `com.android.tools:desugar_jdk_libs:2.0.4` |
| **SDK Artifact** | `pod 'RollaSDK', '0.1.6'` | `com.rolla.sdk:android_release:0.1.6` |

| Feature | Minimum Version | Platform |
|---------|----------------|----------|
| Core SDK | iOS 14.0 / API 24 | Both |
| Apple Health | iOS 14.0 | iOS only |
| Running Speed | iOS 16.0 | iOS only |
| **Live Activities** | **iOS 16.1** | **iOS only** |
| Cycling Cadence & Power | iOS 17.0 | iOS only |

> **Note:** Live Activities require iOS 16.1+. If your deployment target is below 16.1, the widget extension compiles but only activates on devices running 16.1+.

---

## Overview

The Rolla SDK provides a complete health and fitness experience embedded inside partner apps. Built on Flutter with native wrappers for iOS (Swift) and Android (Kotlin), the SDK manages its own UI, Bluetooth band communication, data syncing, and authentication lifecycle.

### Integration Flow

1. **Obtain your Partner ID** — contact [support@rolla.app](mailto:support@rolla.app) to receive your `partner_id` during onboarding
2. **Register the user** — your app calls `POST /api/register` with the user's email and password. Profile data (name, DOB, weight, height, gender, timezone) is collected within the SDK UI, not at registration time.
3. **Log in** — your app calls `POST /api/login` with the user's email, password, and `Partner-ID` header to obtain an access token and refresh token
4. **Present the SDK** — initialize with the tokens and call `show()` — the SDK handles everything from there

See [Auth API — Authentication](sdk-auth-api/02-authentication.md) for full details on each endpoint.

### Environments

| Environment | Value | Use |
|-------------|-------|-----|
| Production | `"production"` | Release builds |
| Development | `"rnd"` | Development and QA |

If omitted, defaults to `"rnd"`.

---

## Support

For issues or questions, contact Rolla support at [support@rolla.app](mailto:support@rolla.app).
