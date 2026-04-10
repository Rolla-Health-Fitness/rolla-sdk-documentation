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
    implementation("com.rolla.sdk:android_release:0.1.6")
}
```

Enable core library desugaring in the android block:

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}
```

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
