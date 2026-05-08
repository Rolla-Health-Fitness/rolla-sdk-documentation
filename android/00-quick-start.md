# Quick Start — Android

Get the Rolla SDK running in your Android app in under 10 minutes.

> **This guide covers the minimal integration.** For branding, modules, and other features, see the [full documentation](README.md).

## Prerequisites

- **Android API 26+** (Android 8.0) minimum SDK — required by the bundled Health Connect plugin
- **Android Studio Hedgehog** (2023.1) or later
- **Gradle 8.0+**
- **Build JDK 17+** — SDK `0.1.10` is compiled with Java 17 (class file major version 61)
- **Kotlin 2.2.0+** required by the bundled Health Connect plugin
- **Partner ID** from Rolla (contact [support@rolla.app](mailto:support@rolla.app))

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the full flow (`/api/register` → `/api/login` → tokens).

## 1. Install the SDK

Add the Maven repositories to `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://rolla-health-fitness.github.io/rolla-sdk-release-android/maven/") }
        maven { url = uri("https://storage.googleapis.com/download.flutter.io") }
        maven { url = uri("https://api.mapbox.com/downloads/v2/releases/maven") }
    }
}
```

Add the dependency to `app/build.gradle.kts`:

```kotlin
dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
    implementation("com.rolla.sdk:android_release:0.1.10")
}

android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
}
```

> Check the [Android release repo](https://github.com/Rolla-Health-Fitness/rolla-sdk-release-android) for the latest version.

Sync the project. For build settings and troubleshooting, see [Gradle Setup](02-gradle-setup.md).

## 2. Configure and Present

Create a minimal `RollaConfiguration`, initialize the SDK, and show it:

```kotlin
import com.rolla.sdk.wrapper.Rolla
import com.rolla.sdk.wrapper.RollaConfiguration
import com.rolla.sdk.wrapper.RollaListener
import com.rolla.sdk.wrapper.RollaCloseReason
import com.rolla.sdk.wrapper.RollaError

class YourActivity : AppCompatActivity() {

    private var rolla: Rolla? = null

    fun showRolla() {
        val configuration = RollaConfiguration(
            token = "your-access-token",       // JWT from POST /api/login
            partnerId = "your-partner-id",
            refreshToken = "your-refresh-token", // From POST /api/login
            tokenExpiresIn = 1800,             // Seconds until token expires (Int)
            environment = "rnd"                // Use "production" for release builds
        )

        val rolla = Rolla(configuration)
        rolla.listener = rollaListener
        this.rolla = rolla
        rolla.show(this)
    }
}
```

## 3. Handle Callbacks

Implement `RollaListener` to respond to SDK events:

```kotlin
private val rollaListener = object : RollaListener {

    override fun onRollaClosed(rolla: Rolla, reason: RollaCloseReason) {
        // SDK was dismissed — clean up your reference
        this@YourActivity.rolla = null
    }

    override fun onRollaError(rolla: Rolla, error: RollaError) {
        Log.e("RollaSDK", "Error: ${error.message}")
    }

    override fun onTokenExpired(rolla: Rolla) {
        // Token expired and SDK's internal refresh failed.
        // Fetch a new token from your backend, then push it to the SDK:
        YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn ->
            rolla.updateToken(
                token = newToken,
                refreshToken = newRefreshToken,
                expiresIn = expiresIn
            )
        }
    }

    override fun onTokenRefreshed(rolla: Rolla, token: String, refreshToken: String?, expiresIn: Int?) {
        // SDK refreshed the token internally — store it for future use
        SessionManager.updateToken(token, refreshToken, expiresIn)
    }
}
```

## 4. Logout Cleanup

When the user logs out, clear the SDK session:

```kotlin
rolla?.clearSession { result ->
    result.onSuccess { Log.d("RollaSDK", "Session cleared") }
    result.onFailure { Log.e("RollaSDK", "Failed to clear: $it") }
}
```

## Complete Example

Here is a single, copy-pasteable `Activity` covering configuration, presentation, callbacks, and cleanup:

```kotlin
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity
import com.rolla.sdk.wrapper.Rolla
import com.rolla.sdk.wrapper.RollaCloseReason
import com.rolla.sdk.wrapper.RollaConfiguration
import com.rolla.sdk.wrapper.RollaError
import com.rolla.sdk.wrapper.RollaListener

class RollaActivity : AppCompatActivity() {

    private var rolla: Rolla? = null

    // --- Present the SDK ---

    fun showRolla(token: String) {
        val config = RollaConfiguration(
            token = token,
            partnerId = "your-partner-id",
            refreshToken = refreshToken,
            tokenExpiresIn = 1800,
            environment = "rnd"               // "production" for release builds
        )

        val rolla = Rolla(config)
        rolla.listener = rollaListener
        this.rolla = rolla
        rolla.show(this)
    }

    // --- RollaListener ---

    private val rollaListener = object : RollaListener {
        override fun onRollaClosed(rolla: Rolla, reason: RollaCloseReason) {
            this@RollaActivity.rolla = null
        }

        override fun onRollaError(rolla: Rolla, error: RollaError) {
            Log.e("RollaSDK", "Error: ${error.message}")
        }

        override fun onTokenExpired(rolla: Rolla) {
            YourAPI.fetchNewToken { newToken, newRefreshToken, expiresIn ->
                rolla.updateToken(
                    token = newToken,
                    refreshToken = newRefreshToken,
                    expiresIn = expiresIn
                )
            }
        }

        override fun onTokenRefreshed(rolla: Rolla, token: String, refreshToken: String?, expiresIn: Int?) {
            SessionManager.updateToken(token, refreshToken, expiresIn)
        }
    }

    // --- Logout ---

    fun logout() {
        rolla?.clearSession { _ -> }
        rolla = null
    }
}
```

## Next Steps

- **Permissions:** Configure internet, Mapbox, and manifest settings — [Permissions](03-permissions.md)
- **Branding:** Customize colors, logos, and enabled modules — [Branding & Modules](05-branding-and-modules.md)
- **Token details:** Full token lifecycle and edge cases — [Token Management](06-token-management.md)
- **API Reference:** All methods, listener interface, and enums — [API Reference](08-api-reference.md)

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
