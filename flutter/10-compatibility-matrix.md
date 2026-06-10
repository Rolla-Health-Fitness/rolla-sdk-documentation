# Compatibility Matrix

The toolchain and platform floors required by each `rolla_sdk` package version.

> These floors are not chosen by Rolla — they are inherited from the SDK's bundled native plugins (chiefly `health` for Health Connect, plus Mapbox). Building below any floor fails fast at `flutter pub get`, the Gradle/Kotlin compile, or the iOS pod install rather than at runtime. See [Prerequisites](01-prerequisites.md) for the rationale behind each.

## Supported versions

| `rolla_sdk` | Flutter | Dart | iOS deployment target | Android `minSdk` | Kotlin | Build JDK |
| --- | --- | --- | --- | --- | --- | --- |
| **0.1.12** | 3.35.6+ | 3.9.2+ | 14.0+ | 26 | 2.2.0+ | 17+ |

All rows are **minimum** floors — newer Flutter/Dart/Kotlin/JDK releases above the floor are expected to work. The values above are verified on-device against the [demo app](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-flutter).

> The Android `minSdk = 26` floor comes from Health Connect (the bundled `health: 13.3.1` plugin), not from Rolla code. The Kotlin `2.2.0+` and JDK `17+` floors come from the same plugin's bytecode and `kotlin_module` metadata. See [Android Prerequisites](../android/01-prerequisites.md) and [Android Gradle Setup](../android/02-gradle-setup.md) for the exact failures you hit below these floors and for the required core library desugaring setup.

> The iOS deployment target `14.0+` floor matches the bundled native pods. See [iOS Prerequisites](../ios/01-prerequisites.md).

## Pinned package install

To stay on the version this guide targets, pin with a caret in `pubspec.yaml`:

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

Or add it from the CLI:

```sh
flutter pub add rolla_sdk
```

## Upcoming changes

> **PE-55 — Flutter floor bump pending.** A future `rolla_sdk` release is expected to raise the Flutter/Dart floor above `3.35.6 / 3.9.2`. This row is not finalized; do not pre-emptively upgrade past the verified floor for `0.1.12`. This page will gain a new row when the bump ships.

When you bump `rolla_sdk` to a version that points at a different native artifact, refresh Gradle's cached transitive metadata before rebuilding:

```sh
cd android && ./gradlew --refresh-dependencies
```

Gradle caches transitive metadata (notably for Mapbox) per coordinate, and stale metadata produces confusing resolution errors across SDK bumps. This must run on your machine — it is not a CI toggle.

---

**Next:** [README](README.md) | **Home:** [README](README.md)
