# Rolla SDK — Android Integration Guide

Integration steps for embedding the Rolla SDK in an Android app (Gradle + Maven).

> **See also:** [iOS Integration Guide](../ios/README.md) | [Overview](../README.md)

> **New here? Start with the [Quick Start guide](00-quick-start.md).**

## Table of Contents

0. [Quick Start](00-quick-start.md) — Minimal integration in under 10 minutes
1. [Prerequisites](01-prerequisites.md) — Requirements and dependencies (Android API 26+, JDK 17+, Kotlin 2.2.0+, Android Studio Hedgehog, Gradle 8.0+, Partner ID and API credentials)
2. [Gradle Setup](02-gradle-setup.md) — Configure Maven repositories, add SDK dependency, set Kotlin/JDK floor, enable desugaring
3. [Permissions](03-permissions.md) — Configure internet, Mapbox token, Health Connect manifest entries, and `<queries>` block
4. [Code Integration](04-code-integration.md) — Import SDK, create configuration, initialize, implement RollaListener, and Fragment support
5. [Branding & Modules](05-branding-and-modules.md) — Custom branding configuration and module enablement
6. [Token Management](06-token-management.md) — Token lifecycle, callbacks, refreshing, and session management
7. [Engine Lifecycle](07-engine-lifecycle.md) — Programmatic dismiss, engine lifecycle, and memory management
8. [API Reference](08-api-reference.md) — Complete Rolla class and RollaListener interface reference, error handling, and close reasons
9. [Troubleshooting](09-troubleshooting.md) — Common issues and solutions

---

**See also:** [iOS Integration Guide](../ios/README.md) | [Overview](../README.md)
