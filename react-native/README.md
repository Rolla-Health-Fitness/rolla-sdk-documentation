# Rolla SDK — React Native Integration Guide

Integration steps for embedding the Rolla SDK in a React Native app via the official Rolla wrapper, `@rolla-health/react-native-sdk`.

> **See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

The Rolla wrapper is a thin TurboModule bridge over the same native iOS pod (`RollaSDK`) and Android Maven artifact (`com.rolla.sdk:android_release`) used by the iOS and Android demos. Most platform configuration is the same as the native guides — this guide covers what's specific to React Native and links out for the rest.

**Wrapper version this guide targets:** `@rolla-health/react-native-sdk@^0.1.2`
**Native SDK version pinned by the wrapper:** `0.1.10` (iOS pod and Android artifact)

## Table of Contents

0. [Quick Start](00-quick-start.md) — Minimal integration in 20–30 minutes
1. [Prerequisites](01-prerequisites.md) — RN version floor, React pin, new arch, partner credentials, RN < 0.80 caveats
2. [Installation](02-installation.md) — `npm install`, Podfile snippet, settings.gradle, `app/build.gradle` deltas
3. [Permissions](03-permissions.md) — iOS Info.plist keys + cross-links to native permissions docs
4. [Code Integration](04-code-integration.md) — `Rolla.show()`, listeners, token-refresh handler, useEffect cleanup
5. [Branding & Modules](05-branding-and-modules.md) — `branding` config shape; links to native module lists
6. [Token Management](06-token-management.md) — `onTokenExpired` event + `updateToken()` push flow
7. [Engine Lifecycle](07-engine-lifecycle.md) — `destroyEngine()` semantics; links to native memory docs
8. [API Reference](08-api-reference.md) — TypeScript types, method signatures, event payloads
9. [Troubleshooting](09-troubleshooting.md) — RN-specific symptoms and remedies

---

## Quick Start

1. Start with [Prerequisites](01-prerequisites.md) to verify your RN floor and React pin
2. Follow [Installation](02-installation.md) for `npm install` plus the Podfile and Gradle deltas
3. Configure [Permissions](03-permissions.md) — almost entirely cross-links to the iOS/Android guides
4. Implement [Code Integration](04-code-integration.md) in your app

For detailed API information, see [API Reference](08-api-reference.md).
For common issues, see [Troubleshooting](09-troubleshooting.md).

---

**See also:** [iOS Integration Guide](../ios/README.md) | [Android Integration Guide](../android/README.md) | [Overview](../README.md)
