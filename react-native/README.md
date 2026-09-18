# Rolla SDK — React Native Integration Guide

A complete, step-by-step guide to integrating the Rolla SDK into your React Native application through the official wrapper, `@rolla-health/react-native-sdk`. This guide covers everything from initial setup through the full JavaScript API: configuration, events, headless calls, host-driven navigation, and notification taps.

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

The wrapper is a TurboModule over the **same** native iOS pod (`RollaSDK`) and Android Maven artifact (`com.rolla.sdk:android_release`) that the [iOS](../ios/README.md) and [Android](../android/README.md) guides document. The Flutter engine, Bluetooth and GPS handling, foreground services, and all SDK UI live inside those native artifacts — your React Native code never sees Flutter. Everything that happens inside your `ios/` and `android/` projects (entitlements, manifests, widget extensions) works exactly as in a native app, so this guide links to the native pages for that and documents the React Native surface itself.

**Wrapper version this guide targets:** `@rolla-health/react-native-sdk@0.1.16`
**Native SDK it links:** `0.1.15` (iOS pod `RollaSDK` and Android `com.rolla.sdk:android_release`) — pinned by the wrapper, never declared by you

The package is released in lockstep with the native SDK; `0.1.16` is the one React Native-only release, and from `0.2.0` on the package version equals the native version. Pin the exact version. See [Prerequisites → Versioning](01-prerequisites.md#versioning).

## Table of Contents

0. **[Quick Start](00-quick-start.md)** — Minimal integration in under 30 minutes
1. **[Prerequisites](01-prerequisites.md)** — React Native and React versions, New Architecture, platform floors, versioning, partner credentials
2. **[Installation](02-installation.md)** — `npm install`, the Podfile, `settings.gradle` and `build.gradle` changes, verifying the integration
3. **[Permissions & Entitlements](03-permissions.md)** — `Info.plist` keys, entitlements and Live Activities on iOS; Mapbox token and manifest entries on Android
4. **[Code Integration](04-code-integration.md)** — Import, configure, present, subscribe to events, handle errors
5. **[Configuration](05-configuration.md)** — Every `RollaConfiguration` field: branding, language, module disabling, data-source hiding, transitions
6. **[Token Management](06-token-management.md)** — Token lifecycle, the token events, pushing new tokens, clearing sessions
7. **[Engine Lifecycle](07-engine-lifecycle.md)** — Warm-up, dismissal, destroying the engine, applying a changed configuration
8. **[API Reference](08-api-reference.md)** — Every `Rolla` method, host-driven navigation, notification taps, all 17 events, headless methods, errors, close reasons
9. **[Troubleshooting](09-troubleshooting.md)** — React Native-specific symptoms and remedies, debug logs, support

---

## Suggested Reading Order

1. Start with [Prerequisites](01-prerequisites.md) to verify your React Native project
2. Follow [Installation](02-installation.md) to add the wrapper and configure both native projects
3. Configure [Permissions & Entitlements](03-permissions.md)
4. Implement [Code Integration](04-code-integration.md) in your app
5. Shape the SDK with [Configuration](05-configuration.md), then wire up [Token Management](06-token-management.md)

For detailed API information, see [API Reference](08-api-reference.md).
For common issues, see [Troubleshooting](09-troubleshooting.md).

## Reference Integration

A complete working integration lives at [`rolla-sdk-demo-react-native`](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-react-native): the full Podfile, entitlements and Live Activity extension, `settings.gradle` and `AndroidManifest.xml`, a login and token-refresh flow, every event, and the same configuration screen, Public API rows, event timeline and notification routing as the native demo apps. When this guide and your build disagree, the demo is the source of truth.

---

## Support

For issues or questions, contact Rolla support.

**See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)
