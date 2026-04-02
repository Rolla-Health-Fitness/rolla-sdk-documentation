# Rolla SDK Integration Guide

Integration documentation for embedding the Rolla SDK into iOS and Android apps.

> **Current SDK Version:** `0.1.5`
>
> **Last Updated:** April 2026

## Platform Guides

| Platform | Guide |
|----------|-------|
| iOS | [iOS Integration Guide](ios/README.md) |
| iOS Live Activities | [Live Activities Setup](ios/live-activities.md) |
| Android | [Android Integration Guide](android/README.md) |

---

## Overview

The Rolla SDK provides a complete health and fitness experience that can be embedded inside partner apps. The SDK is built on Flutter with native wrappers for iOS (Swift) and Android (Kotlin).

Partners integrate the SDK by adding it as a dependency, configuring a few permissions, and presenting it with a single method call. The SDK manages its own UI, data syncing, Bluetooth band communication, and authentication lifecycle internally.

---

## Integration Flow

### 1. User Registration

Every user must be registered once in Rolla's system. The following fields are required:

- First and last name
- Date of birth
- Weight
- Height
- Gender
- Time zone

A separate endpoint will be provided for passwordless registration without email. *(Details to be discussed during onboarding.)*

### 2. User Login

Before presenting the SDK, your app must retrieve an access token for the user:

1. User logs in to your app
2. Your backend calls Rolla's auth API to obtain an `access_token` (and optionally `refresh_token`, `expires_in`)
3. You pass the token into `RollaConfiguration` when initializing the SDK

This process should happen in the background and can be triggered when the user taps the button to open the SDK.

### 3. Present the SDK

Once you have a token, initialize the SDK and present it:

**iOS (Swift):**
```swift
let configuration = RollaConfiguration(
    token: accessToken,
    partnerId: "your-partner-id",
    environment: "production"
)
let rolla = Rolla(configuration: configuration)
rolla.delegate = self
rolla.show(from: self)
```

**Android (Kotlin):**
```kotlin
val configuration = RollaConfiguration(
    token = accessToken,
    partnerId = "your-partner-id",
    environment = "production"
)
val rolla = Rolla(configuration)
rolla.listener = this
rolla.show(activity)
```

See the platform-specific guides for full setup instructions.

---

## Available Modules

The SDK is organized into modules. Currently, all modules are always enabled — pass `nil` (iOS) or `null` (Android) for the `modules` parameter.

Selective module enablement will be available in a future release.

| Module | Description |
|--------|-------------|
| `home` | Main dashboard / home screen |
| `bandPairing` | Bluetooth band connection setup |
| `bandSync` | Band data synchronization |
| `bandFirmware` | Band firmware updates |
| `activityTracking` | Live workout tracking (includes Live Activities on iOS) |
| `activityReview` | Past activity review and history |
| `metrics` | Health metrics overview |
| `weight` | Weight tracking |
| `bloodPressure` | Blood pressure monitoring |
| `goals` | Goal setting and tracking |
| `insights` | Health insights and recommendations |
| `scores` | Health and activity scores |
| `integrations` | External data source integrations (Apple Health, Garmin) |
| `profile` | User profile management |
| `settings` | App settings |
| `branding` | Partner branding and theming |
| `authentication` | Authentication flows |
| `consent` | User consent flows |
| `onboarding` | Onboarding flow |
| `permissions` | Permissions management |
| `support` | Help and support |
| `debugLogs` | Band diagnostics and debug logging |
| `fabMenu` | Floating action button with quick actions (e.g., start workout) |

---

## Branding

Partners can customize the SDK's appearance through `RollaBranding`:

- App name, colors (primary, secondary, accent), brightness, theme mode
- Default locale, terms URL, privacy URL
- Partner logo (via `headerLogoAsset`)

**Important:** Image assets (such as partner logos) must be **pre-bundled inside the SDK** at build time — they cannot be transferred from the host app at runtime. During onboarding, coordinate with Rolla to supply your brand assets (SVG format preferred). Rolla will bundle them into the SDK and provide the correct asset path for your `RollaBranding` configuration.

---

## Environments

The SDK supports two environments:

| Environment | Value | Use |
|-------------|-------|-----|
| Production | `"production"` | Live / release builds |
| Development | `"rnd"` | Development and QA testing |

If omitted, the environment defaults to `"rnd"`.

---

## Support

For issues or questions, contact Rolla support at [support@rolla.app](mailto:support@rolla.app).
