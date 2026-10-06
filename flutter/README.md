# Rolla SDK — Flutter Integration Guide

A complete, step-by-step guide to integrating the Rolla SDK into your Flutter application through the official Dart package, [`rolla_sdk`](https://pub.dev/packages/rolla_sdk). This guide covers everything from initial setup through the full Dart API: initialization, placing the SDK widget, configuration, tokens, host-driven navigation, and the headless calls.

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

`rolla_sdk` embeds the **same** SDK that the [iOS](../ios/README.md) and [Android](../android/README.md) guides document, as a package your app compiles and runs on its own Flutter engine. There is no separate engine to manage and no native wrapper: your app calls the static `RollaSDK` API and renders one widget, `RollaSdkHome`. Everything that happens inside your `ios/` and `android/` projects (usage strings, entitlements, manifest entries, the Live Activity extension) works exactly as in a native app, so this guide links to the native pages for that and documents the Dart surface itself.

**Package version this guide targets:** `rolla_sdk` **0.1.15** (pub.dev)
**SDK version it contains:** `0.1.15` — the package is released in lockstep with the native SDK and carries the same number. See [Prerequisites → Versioning](01-prerequisites.md#versioning).

## Table of Contents

0. **[Quick Start](00-quick-start.md)** — Minimal integration in under 10 minutes
1. **[Prerequisites](01-prerequisites.md)** — Flutter and Dart floor, native platform floors, versioning, partner credentials
2. **[Installation](02-installation.md)** — `flutter pub add rolla_sdk`, the iOS deployment target, the Gradle changes, verifying the build
3. **[Permissions](03-permissions.md)** — `Info.plist` keys and entitlements on iOS; Mapbox token and manifest entries on Android
4. **[Code Integration](04-code-integration.md)** — `RollaSDK.initializeWithToken(...)`, placing `RollaSdkHome`, host dismissal, logout
5. **[Configuration](05-configuration.md)** — Every `initializeWithToken` option: branding, language, module disabling, data-source hiding, UI chrome
6. **[Token Management](06-token-management.md)** — Token lifecycle, `onTokenExpired`, `onSessionExpired`, pushing new tokens, logging out
7. **[API Reference](07-api-reference.md)** — Every `RollaSDK` member, `RollaSdkHome`, host-driven navigation, headless calls, types and enums
8. **[Troubleshooting](08-troubleshooting.md)** — Flutter-specific symptoms and remedies, debug logs, support

---

## Suggested Reading Order

1. Start with [Prerequisites](01-prerequisites.md) to verify your Flutter project and toolchain
2. Follow [Installation](02-installation.md) to add the package and apply the platform floors
3. Configure [Permissions](03-permissions.md) — the keys and manifest entries your app must declare itself
4. Implement [Code Integration](04-code-integration.md) in your app
5. Shape the SDK with [Configuration](05-configuration.md), then wire up [Token Management](06-token-management.md)

For detailed API information, see [API Reference](07-api-reference.md).
For common issues, see [Troubleshooting](08-troubleshooting.md).

## Reference Integration

A complete working integration lives at [`rolla-sdk-demo-flutter`](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-flutter): the full `Info.plist` and `AndroidManifest.xml`, the Gradle changes, a login and token-refresh flow, explicit branding, and the launch screen this guide builds on. When this guide and your build disagree, the demo is the source of truth.

---

## Support

For issues or questions, contact Rolla support.

**See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)
