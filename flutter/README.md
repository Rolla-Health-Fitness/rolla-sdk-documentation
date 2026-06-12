# Rolla SDK — Flutter Integration Guide

Integration steps for embedding the Rolla SDK in a Flutter app via the official Dart package, [`rolla_sdk`](https://pub.dev/packages/rolla_sdk).

> **See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

`rolla_sdk` is a Flutter package: your app depends on it directly and renders the SDK in its own widget tree. Because the host app compiles the SDK (rather than consuming a prebuilt AAR/pod), it declares the platform permissions itself. This guide covers what's specific to Flutter and links to the native guides for the rest.

**Package version this guide targets:** `rolla_sdk: ^0.1.12` (pub.dev)
**Toolchain floor:** Flutter `3.35.6` / Dart `3.9.2` · iOS `14.0` · Android `minSdk 26`, JDK `17+`, Kotlin `2.2.0+`, core library desugaring

## Table of Contents

0. [Quick Start](00-quick-start.md) — Minimal integration in under 10 minutes
1. [Prerequisites](01-prerequisites.md) — Flutter/Dart floor, native platform floors, partner credentials
2. [Installation](02-installation.md) — `flutter pub add rolla_sdk`, iOS deployment target, Gradle deltas + desugaring
3. [Permissions](03-permissions.md) — Info.plist keys and AndroidManifest entries the host app adds
4. [Code Integration](04-code-integration.md) — `RollaSDK.initializeWithToken(...)`, placing `RollaSdkHome` (screen, app root, `FutureBuilder`), host dismissal
5. [Branding & Modules](05-branding-and-modules.md) — `Branding(...)` config, `disabledModules`
6. [Token Management](06-token-management.md) — `onTokenExpired` → `TokenRefreshResult`, `updateToken()`, logout
7. [Permission Gating](07-permissions-gate.md) — How and when the SDK requests runtime permissions
8. [API Reference](08-api-reference.md) — Public Dart API: `RollaSDK`, `RollaSdkHome`, `Branding`, enums, types
9. [Troubleshooting](09-troubleshooting.md) — Flutter-specific symptoms and remedies
10. [Compatibility Matrix](10-compatibility-matrix.md) — Package version ↔ Flutter / Dart / iOS / Android floors

---

## Quick Start

1. Start with [Prerequisites](01-prerequisites.md) to verify your Flutter/Dart and native floors
2. Follow [Installation](02-installation.md) for `flutter pub add rolla_sdk` plus the iOS and Android deltas
3. Configure [Permissions](03-permissions.md) — the Info.plist keys and manifest entries your app must declare
4. Implement [Code Integration](04-code-integration.md): initialize with a token, then render `RollaSdkHome`

For detailed API information, see [API Reference](08-api-reference.md).
For common issues, see [Troubleshooting](09-troubleshooting.md).

---

**See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)
