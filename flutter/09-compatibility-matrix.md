# Compatibility Matrix

The toolchain and platform floors required by each `rolla_sdk` package version. The floors are inherited from the SDK's bundled native plugins; building below any floor fails at `flutter pub get`, the Gradle/Kotlin compile, or the iOS pod install rather than at runtime.

## Supported versions

| `rolla_sdk` | Flutter | Dart | iOS deployment target | Android `minSdk` | Kotlin | Build JDK |
| --- | --- | --- | --- | --- | --- | --- |
| **0.1.12** | 3.35.6+ | 3.9.2+ | 14.0+ | 26 | 2.2.0+ | 17+ |

All values are **minimum** floors — newer releases above the floor are expected to work. The values are verified against the `rolla-sdk-demo-flutter` reference app.

For the build changes each floor requires (including core library desugaring), see [Installation](02-installation.md); for the rationale, see [Android Prerequisites](../android/01-prerequisites.md) and [iOS Prerequisites](../ios/01-prerequisites.md).

## Pinning the package

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

After bumping `rolla_sdk` to a new version, refresh Gradle's cached transitive metadata before rebuilding:

```sh
cd android && ./gradlew --refresh-dependencies
```

---

**Home:** [README](README.md)
