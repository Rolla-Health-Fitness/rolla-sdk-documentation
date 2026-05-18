# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS, Android, and React Native apps.

**Latest SDK Version:** 0.1.10 (native iOS and Android)
**Latest React Native wrapper:** [`@rolla-health/react-native-sdk@0.1.2`](react-native/README.md)

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

## React Native Integration

[**Go to React Native Guide →**](react-native/README.md)

The official wrapper [`@rolla-health/react-native-sdk`](https://www.npmjs.com/package/@rolla-health/react-native-sdk) ships the same native iOS pod and Android Maven artifact behind a TypeScript TurboModule. Cross-links out to the iOS / Android guides for platform configuration; documents the JS-only surface inline.

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](react-native/00-quick-start.md) | Minimal RN integration in 20–30 minutes |
| 1 | [Prerequisites](react-native/01-prerequisites.md) | RN floor (0.80.3+), React 19.1.0 exact, new arch, partner credentials |
| 2 | [Installation](react-native/02-installation.md) | `npm install`, Podfile snippet, settings.gradle, build.gradle deltas |
| 3 | [Permissions](react-native/03-permissions.md) | iOS Info.plist keys; Android handled by AAR manifest merge |
| 4 | [Code Integration](react-native/04-code-integration.md) | `Rolla.show()`, listeners, token-refresh handler, useEffect cleanup |
| 5 | [Branding & Modules](react-native/05-branding-and-modules.md) | `branding` config shape; module configuration |
| 6 | [Token Management](react-native/06-token-management.md) | `onTokenExpired` event + `updateToken()` push flow |
| 7 | [Engine Lifecycle](react-native/07-engine-lifecycle.md) | `destroyEngine()` semantics, warm-vs-cold trade-off |
| 8 | [API Reference](react-native/08-api-reference.md) | TypeScript types, method signatures, event payloads |
| 9 | [Troubleshooting](react-native/09-troubleshooting.md) | RN-specific symptoms (silent SIGABRT, TurboModule registry, peer deps) |

> **RN version floor:** iOS works on RN 0.77+ with new arch. Android requires RN 0.80.3+ because RollaSDK transitively needs AGP 8.9.1 (RN 0.77–0.79 bundle AGP 8.7.x). See [RN Prerequisites](react-native/01-prerequisites.md#react-native-version-floor).

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

Feature support comparison across iOS, Android, and React Native (via `@rolla-health/react-native-sdk`).

| Feature | iOS | Android | React Native | Notes |
|---------|:---:|:-------:|:------------:|-------|
| Core SDK (present, dismiss, token management) | Yes | Yes | Yes | |
| Custom Branding & Modules | Yes | Yes | Yes | All modules currently always enabled |
| Apple Health (HealthKit) | Yes | **No** | Yes (iOS only) | 14 data types, read-only; auto-exposed via native side |
| Health Connect | No | Yes | Yes (Android only) | Added in `0.1.10`; host app declares the manifest entries |
| Live Activities (Lock Screen / Dynamic Island) | Yes | **No** | **No** | RN wrapper does not yet expose JS bindings; native iOS only |
| Bluetooth Band Sync | Yes | Yes | Yes | Background mode on iOS; foreground service on Android |
| Mapbox Maps | Yes | Yes | Yes | Token via `Info.plist` (iOS) / `strings.xml` (Android) |
| Background Location | Yes | Yes | Yes | |
| New Architecture / Bridgeless (RN only) | — | — | **Required** | Wrapper ships codegen TurboModule; old bridge not supported |

## Version Compatibility

| Requirement | iOS | Android | React Native |
|-------------|-----|---------|--------------|
| **Min OS** | iOS 14.0 (15.1 via RN) | API 26 (Android 8.0) | iOS 15.1 / API 26 |
| **RN floor** | — | — | 0.80.3 (Android); 0.77+ likely works on iOS |
| **React** | — | — | 19.1.0 exact |
| **New Arch** | — | — | Required (`newArchEnabled=true`) |
| **IDE** | Xcode 14.0+ | Android Studio Hedgehog (2023.1)+ | both |
| **Dependency Manager** | CocoaPods | Gradle 8.0+ | npm/Yarn + both |
| **Language** | Swift | Kotlin 2.2.0+ (JDK 17+ to build) | TypeScript |
| **Compile / Target SDK** | — | API 36 | API 36 |
| **Core Library Desugaring** | — | `com.android.tools:desugar_jdk_libs:2.0.4` | same as Android |
| **SDK Artifact** | `pod 'RollaSDK', '<version>'` | `com.rolla.sdk:android_release:<version>` | `@rolla-health/react-native-sdk@^0.1.2` |

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
