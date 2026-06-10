# Rolla SDK — Flutter Integration Guide

Integration steps for embedding the Rolla SDK in a Flutter app via the official Dart package, `rolla_sdk`.

> **See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

`rolla_sdk` is a pure-Dart Flutter package: your host app depends on it directly, builds the SDK's screens into your widget tree, and ships its native dependencies (Apple Health, Health Connect, Mapbox, Bluetooth) as transitive Flutter plugins. Because the host app compiles the SDK rather than consuming a prebuilt AAR/pod, **the host app is responsible for its own permission strings and platform floors** — most of which match the native iOS/Android guides. This guide covers what's specific to Flutter and links out for the rest.

**Package version this guide targets:** `rolla_sdk: ^0.1.12` (pub.dev)
**Toolchain floor:** Flutter `3.35.6` / Dart `3.9.2`
**Native floors (from the bundled `health` plugin):** iOS deployment target `14.0` · Android `minSdk 26`, JDK `17+`, Kotlin `2.2.0+`, core library desugaring required.

## Table of Contents

0. [Quick Start](00-quick-start.md) — Minimal integration in 20–30 minutes
1. [Prerequisites](01-prerequisites.md) — Flutter/Dart floor, native platform floors, partner credentials
2. [Installation](02-installation.md) — `flutter pub add rolla_sdk`, iOS Podfile/deployment target, Gradle deltas + desugaring
3. [Permissions](03-permissions.md) — Info.plist keys and AndroidManifest entries you DO add, with cross-links to the native docs
4. [Code Integration](04-code-integration.md) — `RollaSDK.initializeWithToken(...)`, rendering `RollaSdkHome`, `onRequestDismiss` back-navigation
5. [Branding & Modules](05-branding-and-modules.md) — `Branding(...)` config shape, `disabledModules`; links to native module lists
6. [Token Management](06-token-management.md) — `onTokenExpired` → `TokenRefreshResult`, `updateToken()`, logout
7. [Permissions Gate](07-permissions-gate.md) — where permission gating lives (open design; pending product decision)
8. [API Reference](08-api-reference.md) — Public Dart API: `RollaSDK`, `RollaSdkHome`, `Branding`, enums, types
9. [Troubleshooting](09-troubleshooting.md) — Flutter-specific symptoms and remedies
10. [Compatibility Matrix](10-compatibility-matrix.md) — SDK version ↔ required Flutter / Dart / iOS / Android / Kotlin / JDK

---

## Quick Start

1. Start with [Prerequisites](01-prerequisites.md) to verify your Flutter/Dart and native floors.
2. Follow [Installation](02-installation.md) for `flutter pub add rolla_sdk` plus the iOS Podfile and Android Gradle deltas (including core library desugaring).
3. Configure [Permissions](03-permissions.md) — add the required Info.plist keys and AndroidManifest entries to your host app.
4. Implement [Code Integration](04-code-integration.md): call `RollaSDK.initializeWithToken(...)`, then render `RollaSdkHome(userId: ...)`. Do **not** wrap it in another `MaterialApp` — it builds its own `MaterialApp.router`.

For detailed API information, see [API Reference](08-api-reference.md).
For common issues, see [Troubleshooting](09-troubleshooting.md).

---

**Next:** [Quick Start](00-quick-start.md) | **Home:** [README](README.md)
