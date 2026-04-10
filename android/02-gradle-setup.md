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

---

**Previous:** [Prerequisites](01-prerequisites.md) | **Next:** [Permissions](03-permissions.md) | **Home:** [README](README.md)
