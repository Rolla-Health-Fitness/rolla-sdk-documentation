# Rolla SDK Integration Guide

Documentation for embedding the Rolla SDK into partner iOS, Android, Flutter and React Native apps.

**Latest SDK Version:** 0.1.15
**Latest Flutter package:** [`rolla_sdk@0.1.15+1`](flutter/README.md)
**Latest React Native wrapper:** [`@rolla-health/react-native-sdk@0.1.16`](react-native/README.md)

---

## iOS Integration

[**Go to iOS Guide →**](ios/README.md)

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](ios/00-quick-start.md) | Minimal integration in under 10 minutes |
| 1 | [Prerequisites](ios/01-prerequisites.md) | iOS version, CocoaPods, Xcode |
| 2 | [CocoaPods Setup](ios/02-cocoapods-setup.md) | Add SDK dependency, build settings |
| 3 | [Permissions & Entitlements](ios/03-permissions-and-entitlements.md) | Info.plist, Bluetooth, Location, Camera, Mapbox, HealthKit |
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
| 3 | [Permissions & Entitlements](flutter/03-permissions.md) | Info.plist keys, entitlements, notification delegate; Mapbox token and manifest entries |
| 4 | [Code Integration](flutter/04-code-integration.md) | `RollaSDK.initializeWithToken(...)`, placing `RollaSdkHome`, host dismissal, logout |
| 5 | [Configuration](flutter/05-configuration.md) | Branding, language, modules, data sources, UI chrome |
| 6 | [Token Management](flutter/06-token-management.md) | Auth lifecycle, `onTokenExpired`, `onSessionExpired`, refresh, logout |
| 7 | [API Reference](flutter/07-api-reference.md) | `RollaSDK`, `RollaSdkHome`, host-driven navigation, headless calls, types |
| 8 | [Troubleshooting](flutter/08-troubleshooting.md) | Flutter-specific issues & support |

> **Versions:** `rolla_sdk@0.1.15+1` contains SDK `0.1.15` (the `+1` is a page-only republish of `0.1.15`, same code); from 0.1.15 on, the package version equals the SDK version. Pin it exactly. See [Flutter Prerequisites → Versioning](flutter/01-prerequisites.md#versioning).

---

## React Native Integration

[**Go to React Native Guide →**](react-native/README.md)

The official wrapper [`@rolla-health/react-native-sdk`](https://www.npmjs.com/package/@rolla-health/react-native-sdk) ships the same native iOS pod and Android Maven artifact behind a TypeScript TurboModule. It documents the JavaScript surface and links to the iOS / Android guides for everything inside the native projects.

| # | Section | Description |
|---|---------|-------------|
| 0 | [Quick Start](react-native/00-quick-start.md) | Minimal integration in under 30 minutes |
| 1 | [Prerequisites](react-native/01-prerequisites.md) | React Native 0.80.3 + New Architecture, React 19.1.0 exact, platform floors, versioning |
| 2 | [Installation](react-native/02-installation.md) | `npm install`, Podfile, `settings.gradle` and `build.gradle` changes, verification |
| 3 | [Permissions & Entitlements](react-native/03-permissions.md) | Info.plist, entitlements, Live Activities; Mapbox token, Health Connect manifest entries |
| 4 | [Code Integration](react-native/04-code-integration.md) | Import, configure, present, events, error handling |
| 5 | [Configuration](react-native/05-configuration.md) | Branding, language, modules, data sources, transitions |
| 6 | [Token Management](react-native/06-token-management.md) | Auth lifecycle, token events, refresh, session clear |
| 7 | [Engine Lifecycle](react-native/07-engine-lifecycle.md) | Warm-up, dismiss, destroy, applying a changed configuration |
| 8 | [API Reference](react-native/08-api-reference.md) | `Rolla` methods, host-driven navigation, notification taps, 17 events, headless methods, errors |
| 9 | [Troubleshooting](react-native/09-troubleshooting.md) | React Native-specific issues & support |

> **Versions:** `@rolla-health/react-native-sdk@0.1.16` links native SDK `0.1.15`; from `0.2.0` on the package version equals the native version. See [React Native Prerequisites → Versioning](react-native/01-prerequisites.md#versioning).

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

Feature support across the four integration paths. The Flutter package embeds the same SDK in your app's own Flutter engine, so the features apply to a Flutter host on the respective platform; the Dart surface differs where the Flutter guide says so — no host event callbacks, no `notificationTarget`, no `warmUpEngine` / engine lifecycle, no `RollaTransition`, and branding is the `Branding` class, which replaces the defaults as a whole rather than merging per field. See [Flutter API Reference → Not Available in Flutter](flutter/07-api-reference.md#not-available-in-flutter). The React Native wrapper ships the same native iOS pod and Android artifact, so a React Native host gets every native capability on the respective platform.

| Feature | iOS | Android | Flutter | React&nbsp;Native | Notes |
|---------|:---:|:-------:|:-------:|:-----------------:|-------|
| Core&nbsp;SDK&nbsp;(present,&nbsp;dismiss,&nbsp;token&nbsp;management) | Yes | Yes | Yes | Yes | |
| Custom&nbsp;Branding | Yes | Yes | Yes | Yes | App name (`hostAppName`), primary color, theme, logo, privacy link, Rolla Band wording (`removeRollaBandReferences`) — all optional, per-field overrides |
| Module&nbsp;Disabling | Yes | Yes | Yes | Yes | `disabledModules`; `weight`, `bloodPressure`, `leaderboards`, `insights`, `nutrition` and `groupActivities` can currently be disabled |
| Data&nbsp;Source&nbsp;Hiding | Yes | Yes | Yes | Yes | `disabledDataSources`; hide band/Garmin/Oura/Apple Health/Health Connect connect options |
| Garmin&nbsp;/&nbsp;Oura&nbsp;In-App&nbsp;Sign-In&nbsp;Return&nbsp;(<code>oauthCallbackScheme</code>) | Yes | Yes | Yes | Yes | Optional; must match the `app_scheme` Rolla configures for your partner account, and Android apps register it on `flutter_web_auth_2`'s `CallbackActivity`; unset keeps the external-browser flow |
| Host-Controlled&nbsp;Language | Yes | Yes | Yes | Yes | `language` (`RollaLanguage`); force one of the SDK's 8 languages, or leave it profile-driven |
| Leaderboards | Yes | Yes | Yes | Yes | Opt-in weekly/monthly rankings on Health Score / Active Points; hide via `disabledModules` |
| Insights | Yes | Yes | Yes | Yes | Personalized insights feed with a Home-screen entry and unread badge; hide via `disabledModules` |
| Nutrition | Yes | Yes | Yes | Yes | On by default: calorie budget, meals, water, weight goals, meal plans, and day templates; AI meal estimates from a photo or description, enabled per partner by Rolla; Home-screen entry beside Insights; hide via `disabledModules`; needs `NSCameraUsageDescription` (iOS) / two CameraX manifest overrides (Android) |
| Indoor&nbsp;Group&nbsp;Sessions | Yes | Yes | Yes | Yes | Create or join a group from the indoor activity setup and follow the participants live; Rolla's backend enables it per tenant; hide via `disabledModules` (`groupActivities`) |
| Goals&nbsp;on&nbsp;Home | Yes | Yes | Yes | Yes | `showGoalsSection` (default `false`): the user's goals with an edit action at the bottom of Home |
| <code>show()</code>&nbsp;Transition&nbsp;Option | Yes | Yes | **No** | Yes | `RollaTransition`: `default` or `fade` open/close animation; in Flutter, the route you push `RollaSdkHome` on owns the animation |
| Host-Driven&nbsp;Navigation | Yes | Yes | Yes | Yes | `openScreen`: open the SDK directly on a specific screen — insights, nutrition, activity history, goals, Home, or the last-opened state |
| Notification&nbsp;Tap&nbsp;Routing | Yes | Yes | **No** | Yes | `notificationTarget`: recognize a tapped Rolla notification and resolve its destination — an SDK screen for `openScreen`, or the OS app-settings page; in Flutter, the SDK routes its own notification taps |
| External&nbsp;Heart&nbsp;Rate&nbsp;Monitors | Yes | Yes | Yes | Yes | Standard Bluetooth HR chest straps and arm bands as a workout's heart rate source |
| Manual&nbsp;Sleep&nbsp;Logging | Yes | Yes | Yes | Yes | Users can log or correct a night from the sleep detail screen (last 7 days) |
| Historical&nbsp;Data&nbsp;Import | Yes | Yes | Yes | Yes | One-time backfill offered when a data source is connected; restartable from Data Sources |
| Headless&nbsp;Methods | Yes | Yes | Yes | Yes | `warmUpEngine`, `syncHealthData`, `getBandBatteryLevel`, `getPairedBandInfo` — no SDK UI needed; in Flutter, `initializeWithToken` takes the place of `warmUpEngine` |
| Host&nbsp;Event&nbsp;Callbacks | Yes | Yes | **No** | Yes | 12 observational delegate/listener callbacks (events in React Native): activity lifecycle, band pairing & connection, sync results, goals, profile; not in Flutter — see [Not Available in Flutter](flutter/07-api-reference.md#not-available-in-flutter) |
| Apple&nbsp;Health&nbsp;(HealthKit) | Yes | **No** | Yes&nbsp;(iOS) | Yes&nbsp;(iOS) | 14 data types, read-only |
| Health&nbsp;Connect | No | Yes | Yes&nbsp;(Android) | Yes&nbsp;(Android) | Host app declares the manifest entries |
| Live&nbsp;Activities&nbsp;(Lock&nbsp;Screen&nbsp;/&nbsp;Dynamic&nbsp;Island) | Yes | **No** | Yes&nbsp;(iOS) | Yes&nbsp;(iOS) | Requires iOS 16.1+ and a widget extension in your iOS project |
| Bluetooth&nbsp;Band&nbsp;Sync | Yes | Yes | Yes | Yes | Background mode on iOS; foreground service on Android |
| Smartphone-Only&nbsp;Workout&nbsp;Tracking | Yes | Yes | Yes | Yes | Needs `NSMotionUsageDescription` (iOS) / `ACTIVITY_RECOGNITION` (Android, SDK-declared) |
| Mapbox&nbsp;Maps | Yes | Yes | Yes | Yes | Token via `Info.plist` (iOS) / `strings.xml` (Android) |
| Background&nbsp;Location | Yes | Yes | Yes | Yes | |

## Version Compatibility

| Requirement | iOS | Android | Flutter | React&nbsp;Native |
|-------------|-----|---------|---------|-------------------|
| **Min&nbsp;OS** | iOS&nbsp;15.0 | API&nbsp;26&nbsp;(Android&nbsp;8.0) | iOS&nbsp;15.0&nbsp;/&nbsp;API&nbsp;26 | iOS&nbsp;15.1&nbsp;/&nbsp;API&nbsp;26 |
| **Framework** | — | — | Flutter&nbsp;3.35.6+&nbsp;(Dart&nbsp;3.9+) | React&nbsp;Native&nbsp;0.80.3&nbsp;+&nbsp;New&nbsp;Architecture |
| **IDE** | Xcode&nbsp;26.0+ | Android&nbsp;Studio&nbsp;Hedgehog&nbsp;(2023.1)+ | Same&nbsp;as&nbsp;iOS&nbsp;/&nbsp;Android | Same&nbsp;as&nbsp;iOS&nbsp;/&nbsp;Android |
| **Dependency&nbsp;Manager** | CocoaPods | Gradle&nbsp;8.0+ | pub&nbsp;(CocoaPods&nbsp;/&nbsp;Gradle&nbsp;underneath) | npm&nbsp;(CocoaPods&nbsp;/&nbsp;Gradle&nbsp;underneath) |
| **Language** | Swift | Kotlin&nbsp;2.2.0+&nbsp;(JDK&nbsp;17+&nbsp;to&nbsp;build) | Dart | TypeScript&nbsp;/&nbsp;JavaScript |
| **Compile&nbsp;/&nbsp;Target&nbsp;SDK** | — | API&nbsp;36 | Same&nbsp;as&nbsp;Android | Same&nbsp;as&nbsp;Android |
| **Core&nbsp;Library&nbsp;Desugaring** | — | <code>com.android.tools:desugar_jdk_libs:2.0.4</code> | Same&nbsp;as&nbsp;Android | Same&nbsp;as&nbsp;Android |
| **SDK&nbsp;Artifact** | <code>pod&nbsp;'RollaSDK',&nbsp;'&lt;version&gt;'</code> | <code>com.rolla.sdk:android_release:&lt;version&gt;</code> | <code>rolla_sdk:&nbsp;&lt;version&gt;</code>&nbsp;(pub.dev,&nbsp;exact) | <code>@rolla-health/react-native-sdk@&lt;version&gt;</code>&nbsp;(npm,&nbsp;exact) |

Flutter and React Native apps build their iOS and Android projects with the same toolchain as the iOS and Android columns, including Kotlin 2.2.0+ and JDK 17 on Android.

| Feature | Minimum Version | Platform |
|---------|----------------|----------|
| Core SDK | iOS 15.0 / API 26 | Both |
| Apple Health | iOS 15.0 | iOS only |
| Health Connect | API 26 | Android only |
| Running Speed | iOS 16.0 | iOS only |
| **Live Activities** | **iOS 16.1** | **iOS only** |
| Cycling Cadence & Power | iOS 17.0 | iOS only |

> **Note:** Live Activities require iOS 16.1+. If your deployment target is below 16.1, the widget extension compiles but only activates on devices running 16.1+.

---

## Overview

The Rolla SDK provides a complete health and fitness experience embedded inside partner apps. Built on Flutter with native wrappers for iOS (Swift) and Android (Kotlin), and available to Flutter apps as the `rolla_sdk` package and to React Native apps through the `@rolla-health/react-native-sdk` wrapper, the SDK manages its own UI, Bluetooth band communication, data syncing, and authentication lifecycle.

### Integration Flow

1. **Obtain your Partner ID** — contact [support@rolla.app](mailto:support@rolla.app) to receive your `partner_id` during onboarding
2. **Register the user** — your app calls `POST /api/register` with the user's email and password. Profile data (name, DOB, weight, height, gender, timezone) is collected within the SDK UI — or your app [sets it in advance](sdk-auth-api/03-profile.md) after login so the SDK's onboarding is skipped.
3. **Log in** — your app calls `POST /api/login` with the user's email, password, and `Partner-ID` header to obtain an access token and refresh token
4. **Present the SDK** — initialize with the tokens and present the SDK UI: on iOS and Android, create a `Rolla` with a `RollaConfiguration` and call `show()`; in Flutter, call `RollaSDK.initializeWithToken`, then render `RollaSdkHome`; in React Native, call `Rolla.show(configuration)` — the SDK handles everything from there

See [Auth API — Authentication](sdk-auth-api/02-authentication.md) for full details on each endpoint.

### Environments

| Environment | Value | Use |
|-------------|-------|-----|
| Production | `"production"` | Release builds |
| Research and Development | `"rnd"` | Development and QA |

If omitted, the native SDKs default to `"rnd"`; the Flutter package defaults to `RollaEnvironment.production`, so set `.rnd` explicitly while integrating.

---

## Support

For issues or questions:

- **Email:** [support@rolla.app](mailto:support@rolla.app)
- **Slack:** If your organization has a dedicated partner Slack channel with Rolla, use it for faster responses on integration questions and live debugging.

If you don't have a partner Slack channel set up yet, ask your Rolla contact or email support to request one.
