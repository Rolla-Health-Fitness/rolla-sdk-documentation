# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS, Android, and Flutter apps.

**Latest SDK Version:** 0.1.12

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

## Flutter Integration

[**Go to Flutter Guide →**](flutter/README.md)

The official Dart package [`rolla_sdk`](https://pub.dev/packages/rolla_sdk) embeds the SDK directly in your Flutter widget tree.

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](flutter/00-quick-start.md) | Minimal integration in under 10 minutes |
| 1 | [Prerequisites](flutter/01-prerequisites.md) | Flutter/Dart floor, native platform floors, partner credentials |
| 2 | [Installation](flutter/02-installation.md) | `flutter pub add rolla_sdk`, iOS deployment target, Gradle deltas + desugaring |
| 3 | [Permissions](flutter/03-permissions.md) | Info.plist keys and AndroidManifest entries the host app adds |
| 4 | [Code Integration](flutter/04-code-integration.md) | `RollaSDK.initializeWithToken(...)`, placing `RollaSdkHome`, host dismissal |
| 5 | [Branding & Modules](flutter/05-branding-and-modules.md) | `Branding(...)` config, `disabledModules` |
| 6 | [Token Management](flutter/06-token-management.md) | `onTokenExpired` → `TokenRefreshResult`, `updateToken()`, logout |
| 7 | [API Reference](flutter/07-api-reference.md) | Public Dart API: `RollaSDK`, `RollaSdkHome`, `Branding`, enums, types |
| 8 | [Troubleshooting](flutter/08-troubleshooting.md) | Flutter-specific symptoms and remedies |
| 9 | [Compatibility Matrix](flutter/09-compatibility-matrix.md) | Package version ↔ Flutter / Dart / iOS / Android floors |

> **Flutter hosts declare their own permissions.** Because the app consumes the Dart package (not a prebuilt AAR/pod), the SDK's permission strings must be added to `ios/Runner/Info.plist` and `android/app/src/main/AndroidManifest.xml` — a missing iOS usage string aborts the app with SIGABRT. See [Flutter Permissions](flutter/03-permissions.md).

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

Feature support comparison between iOS and Android.

| Feature | iOS | Android | Notes |
|---------|:---:|:-------:|-------|
| Core SDK (present, dismiss, token management) | Yes | Yes | |
| Custom Branding & Modules (all modules currently always enabled) | Yes | Yes | All modules currently always enabled |
| Apple Health (HealthKit) | Yes | **No** | 14 data types, read-only |
| Health Connect | No | Yes | Added in `0.1.10`; host app declares the manifest entries |
| Live Activities (Lock Screen / Dynamic Island) | Yes | **No** | Requires iOS 16.1+ |
| Bluetooth Band Sync | Yes | Yes | Background mode on iOS; foreground service on Android |
| Mapbox Maps | Yes | Yes | Token via `Info.plist` (iOS) / `strings.xml` (Android) |
| Background Location | Yes | Yes | |

## Version Compatibility

| Requirement | iOS | Android |
|-------------|-----|---------|
| **Min OS** | iOS 14.0 | API 26 (Android 8.0) |
| **IDE** | Xcode 14.0+ | Android Studio Hedgehog (2023.1)+ |
| **Dependency Manager** | CocoaPods | Gradle 8.0+ |
| **Language** | Swift | Kotlin 2.2.0+ (JDK 17+ to build) |
| **Compile / Target SDK** | — | API 36 |
| **Core Library Desugaring** | — | `com.android.tools:desugar_jdk_libs:2.0.4` |
| **SDK Artifact** | `pod 'RollaSDK', '<version>'` | `com.rolla.sdk:android_release:<version>` |

| Feature | Minimum Version | Platform |
|---------|----------------|----------|
| Core SDK | iOS 14.0 / API 26 | Both |
| Apple Health | iOS 14.0 | iOS only |
| Health Connect | API 26 | Android only |
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
| Research and Development | `"rnd"` | Development and QA |

If omitted, defaults to `"rnd"`.

---

## Support

For issues or questions:

- **Email:** [support@rolla.app](mailto:support@rolla.app)
- **Slack:** If your organization has a dedicated partner Slack channel with Rolla, use it for faster responses on integration questions and live debugging.

If you don't have a partner Slack channel set up yet, ask your Rolla contact or email support to request one.
