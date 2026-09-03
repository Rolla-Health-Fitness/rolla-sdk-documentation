# Gradle Setup

This guide walks you through configuring Gradle to integrate the Rolla SDK into your Android project.

## Add Maven Repositories

Add the required Maven repositories to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()

        // Rolla SDK repository
        maven {
            url = uri("https://rolla-health-fitness.github.io/rolla-sdk-release-android/maven/")
        }

        // Flutter engine artifacts
        maven {
            url = uri("https://storage.googleapis.com/download.flutter.io")
        }

        // Mapbox SDK for maps
        maven {
            url = uri("https://api.mapbox.com/downloads/v2/releases/maven")
        }
    }
}
```

## Add SDK Dependency

Add the Rolla SDK dependency to your `app/build.gradle.kts`:

```kotlin
dependencies {
    // Core library desugaring (required by SDK)
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")

    // Rolla SDK
    implementation("com.rolla.sdk:android_release:0.1.15")
}
```

> Check the [Android release repo](https://github.com/Rolla-Health-Fitness/rolla-sdk-release-android) for the latest version.

Enable core library desugaring and set `minSdk = 26`

```kotlin
android {
    defaultConfig {
        minSdk = 26
    }

    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}
```

## Kotlin Version

The host app must build with **Kotlin 2.2.0 or newer**. In a typical project this is set in `gradle/libs.versions.toml`:

```toml
[versions]
kotlin = "2.2.0"   # or any patch ≥ 2.2.0 (e.g. 2.2.10)
```

The floor is a transitive requirement of the bundled `health` Flutter plugin — the SDK itself is also built against Kotlin 2.2.0. Any patch ≥ 2.2.0 is forward-compatible (2.2.10, 2.3.x, …). Kotlin ≥ 2.1.0 should work as well because of the [version tolerance](https://kotlinlang.org/docs/metadata-jvm.html#maven) rule, but 2.2.0 is still the recommended minimum.

## Build JDK

The SDK `0.1.15` AAR ships Java 17 bytecode (class file major version 61). Your build JDK must be **17 or newer**.

## Sync Project

Click "Sync Now" in the Gradle notification bar, or run:

```
./gradlew --refresh-dependencies
```

## ProGuard / R8

The SDK bundles consumer ProGuard rules (`consumer-rules.pro`) that are automatically applied to your release build when you enable minification. **No manual ProGuard configuration is required.**

The bundled rules keep the SDK's public API classes, Pigeon-generated bridge code, host API implementations, and Flutter plugin registration intact. You do not need to add any `-keep` rules for the Rolla SDK.

**Testing release builds:** Always verify your release build with `minifyEnabled true` before shipping. While the SDK's consumer rules cover its own classes, conflicts with your app's custom rules or aggressive R8 optimizations can surface only in minified builds. Run through the full SDK flow (show, dismiss, token refresh) in a release build during QA.

---

**Previous:** [Prerequisites](01-prerequisites.md) | **Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
