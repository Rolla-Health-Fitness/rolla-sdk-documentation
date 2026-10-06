# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS, Android, and Flutter apps.

**Latest SDK Version:** 0.1.15
**Latest Flutter package:** [`rolla_sdk@0.1.15`](flutter/README.md)

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
| 5 | [Configuration](ios/05-configuration.md) | Branding, language, modules, data sources |
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
| 5 | [Configuration](android/05-configuration.md) | Branding, language, modules, data sources |
| 6 | [Token Management](android/06-token-management.md) | Auth lifecycle, refresh, session clear |
| 7 | [Engine Lifecycle](android/07-engine-lifecycle.md) | Flutter engine, dismiss, memory |
| 8 | [API Reference](android/08-api-reference.md) | Rolla class, listener, errors, close reasons |
| 9 | [Troubleshooting](android/09-troubleshooting.md) | Common issues & support |

---

## Flutter Integration

[**Go to Flutter Guide →**](flutter/README.md)

The official Dart package [`rolla_sdk`](https://pub.dev/packages/rolla_sdk) embeds the same SDK directly in your Flutter app, running on your app's own Flutter engine. It documents the Dart surface and links to the iOS / Android guides for everything inside the native projects.

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](flutter/00-quick-start.md) | Minimal integration in under 10 minutes |
| 1 | [Prerequisites](flutter/01-prerequisites.md) | Flutter/Dart floor, native platform floors, versioning, partner credentials |
| 2 | [Installation](flutter/02-installation.md) | `flutter pub add rolla_sdk`, iOS deployment target, Gradle deltas + desugaring |
| 3 | [Permissions](flutter/03-permissions.md) | Info.plist keys and entitlements; Mapbox token and manifest entries |
| 4 | [Code Integration](flutter/04-code-integration.md) | `RollaSDK.initializeWithToken(...)`, placing `RollaSdkHome`, host dismissal, logout |
| 5 | [Configuration](flutter/05-configuration.md) | Branding, language, modules, data sources, UI chrome |
| 6 | [Token Management](flutter/06-token-management.md) | Auth lifecycle, `onTokenExpired`, `onSessionExpired`, refresh, logout |
| 7 | [API Reference](flutter/07-api-reference.md) | `RollaSDK`, `RollaSdkHome`, host-driven navigation, headless calls, types |
| 8 | [Troubleshooting](flutter/08-troubleshooting.md) | Flutter-specific issues & support |

> **Versions:** `rolla_sdk@0.1.15` contains SDK `0.1.15`; the package version always equals the SDK version it carries. Flutter hosts declare the platform permissions themselves — a missing iOS usage string aborts the app with SIGABRT. See [Flutter Permissions](flutter/03-permissions.md).

---

## Auth API — SDK Authentication

[**Go to Auth API Guide →**](sdk-auth-api/README.md)

| # | Section | Description |
|---|---------|-------------|
| 1 | [Overview](sdk-auth-api/01-overview.md) | Auth architecture, base URLs, environments, onboarding |
| 2 | [Authentication](sdk-auth-api/02-authentication.md) | Register users, log in, obtain tokens, refresh tokens |
| 3 | [Profile](sdk-auth-api/03-profile.md) | Set profile data in advance, skip the SDK's onboarding |
| 4 | [Error Handling](sdk-auth-api/04-error-handling.md) | Error format, status codes, retry strategies, checklist |

> **Server-to-server data integration:** Rolla also offers a Partner API for backend-to-backend access to user health data, activity data, and user management. This is separate from the SDK integration. Contact [support@rolla.app](mailto:support@rolla.app) for Partner API access.

---

## Platform Capabilities

Feature support comparison between iOS and Android. The Flutter package embeds the same SDK, so every row applies to a Flutter host on the respective platform; the Dart surface differs where the [Flutter guide](flutter/README.md) says so (no host event callbacks, no `notificationTarget`, no engine lifecycle).

| Feature | iOS | Android | Notes |
|---------|:---:|:-------:|-------|
| Core SDK (present, dismiss, token management) | Yes | Yes | |
| Custom Branding | Yes | Yes | App name (`hostAppName`), primary color, theme, logo, privacy link, Rolla Band wording (`removeRollaBandReferences`) — all optional, per-field overrides |
| Module Disabling | Yes | Yes | `disabledModules`; `weight`, `bloodPressure`, `leaderboards`, and `insights` can currently be disabled |
| Data Source Hiding | Yes | Yes | `disabledDataSources`; hide band/Garmin/Oura/Apple Health/Health Connect connect options |
| Host-Controlled Language | Yes | Yes | `language` (`RollaLanguage`); force one of the SDK's 8 languages, or leave it profile-driven |
| Leaderboards | Yes | Yes | Opt-in weekly/monthly rankings on Health Score / Active Points; hide via `disabledModules` |
| Insights | Yes | Yes | Personalized insights feed with a Home-screen entry and unread badge; hide via `disabledModules` |
| Goals on Home | Yes | Yes | `showGoalsSection` (default `false`): the user's goals with an edit action at the bottom of Home |
| `show()` Transition Option | Yes | Yes | `RollaTransition`: `default` or `fade` open/close animation |
| Host-Driven Navigation | Yes | Yes | `openScreen`: open the SDK directly on a specific screen — insights, activity history, goals, Home, or the last-opened state |
| Notification Tap Routing | Yes | Yes | `notificationTarget`: recognize a tapped Rolla notification and resolve its destination — an SDK screen for `openScreen`, or the OS app-settings page |
| External Heart Rate Monitors | Yes | Yes | Standard Bluetooth HR chest straps and arm bands as a workout's heart rate source |
| Manual Sleep Logging | Yes | Yes | Users can log or correct a night from the sleep detail screen (last 7 days) |
| Historical Data Import | Yes | Yes | One-time backfill offered when a data source is connected; restartable from Data Sources |
| Headless Methods | Yes | Yes | `warmUpEngine`, `syncHealthData`, `getBandBatteryLevel`, `getPairedBandInfo` — no SDK UI needed |
| Host Event Callbacks | Yes | Yes | 12 observational delegate/listener callbacks: activity lifecycle, band pairing & connection, sync results, goals, profile |
| Apple Health (HealthKit) | Yes | **No** | 14 data types, read-only |
| Health Connect | No | Yes | Host app declares the manifest entries |
| Live Activities (Lock Screen / Dynamic Island) | Yes | **No** | Requires iOS 16.1+ |
| Bluetooth Band Sync | Yes | Yes | Background mode on iOS; foreground service on Android |
| Smartphone-Only Workout Tracking | Yes | Yes | Needs `NSMotionUsageDescription` (iOS) / `ACTIVITY_RECOGNITION` (Android, SDK-declared) |
| Mapbox Maps | Yes | Yes | Token via `Info.plist` (iOS) / `strings.xml` (Android) |
| Background Location | Yes | Yes | |

## Version Compatibility

| Requirement | iOS | Android | Flutter |
|-------------|-----|---------|---------|
| **Min OS** | iOS 14.0 | API 26 (Android 8.0) | iOS 14.0 / API 26 |
| **Flutter** | — | — | 3.35.6+ / Dart 3.9.2+ |
| **IDE** | Xcode 14.0+ | Android Studio Hedgehog (2023.1)+ | both |
| **Dependency Manager** | CocoaPods | Gradle 8.0+ | pub + both |
| **Language** | Swift | Kotlin 2.2.0+ (JDK 17+ to build) | Dart (Kotlin 2.2.0+ for the Android build) |
| **Compile / Target SDK** | — | API 36 | API 36 |
| **Core Library Desugaring** | — | `com.android.tools:desugar_jdk_libs:2.0.4` | same as Android |
| **SDK Artifact** | `pod 'RollaSDK', '<version>'` | `com.rolla.sdk:android_release:<version>` | `rolla_sdk: ^<version>` (pub.dev) |

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
2. **Register the user** — your app calls `POST /api/register` with the user's email and password. Profile data (name, DOB, weight, height, gender, timezone) is collected within the SDK UI — or your app [sets it in advance](sdk-auth-api/03-profile.md) after login so the SDK's onboarding is skipped.
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
