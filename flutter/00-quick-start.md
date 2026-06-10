# Quick Start

A minimal end-to-end Flutter integration in 20–30 minutes. Run on physical devices — Bluetooth and GPS features do not work in simulators or emulators.

> **Before you begin:** verify your project meets the [Prerequisites](01-prerequisites.md). The floors are **Flutter 3.35.6 / Dart 3.9.2**, **iOS 14.0**, and **Android minSdk 26** — they come from the bundled `health` plugin and are not negotiable. Your backend must also mint Rolla access tokens (`POST /api/login`); see [Auth API — Authentication](../sdk-auth-api/02-authentication.md).

## 1. Add the package

```sh
flutter pub add rolla_sdk
```

This pins `rolla_sdk: ^0.1.12` in your `pubspec.yaml`. Then fetch it:

```sh
flutter pub get
```

## 2. Configure iOS

Unlike a native add-to-app host, a Flutter host consumes the Dart package, so **you must declare the iOS usage strings yourself** — the `flutter create` scaffold ships none. Add the following to `ios/Runner/Info.plist`:

```xml
<key>NSBluetoothAlwaysUsageDescription</key>
<string>Connect to your fitness band to sync health data and activity metrics.</string>
<key>NSBluetoothPeripheralUsageDescription</key>
<string>Connect to your fitness band to sync health data and activity metrics.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Track outdoor activities like running and cycling while the app is open.</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Record complete routes for outdoor activities even when the phone is locked.</string>
<key>NSMotionUsageDescription</key>
<string>Count steps and measure cadence when no fitness band is connected.</string>
<key>NSHealthShareUsageDescription</key>
<string>Read your Apple Health data to provide personalised health insights.</string>
<key>NSPhotoLibraryAddUsageDescription</key>
<string>Save your activity images to Photos.</string>
<key>UIBackgroundModes</key>
<array>
  <string>location</string>
  <string>bluetooth-central</string>
</array>
```

> **Without these keys iOS aborts with SIGABRT** the moment the SDK touches the corresponding API — there is no Dart-side error you can catch. This was confirmed on-device: the fresh `flutter create` scaffold crashed immediately on the missing Bluetooth usage description.

Set the deployment target to **14.0** in `ios/Podfile` (`platform :ios, '14.0'`), then install pods:

```sh
cd ios && pod install
```

For the exact strings, rationale (used in App Store privacy questionnaires), and the Mapbox `MBXAccessToken`, see [iOS Permissions & Entitlements](../ios/03-permissions-and-entitlements.md).

## 3. Configure Android

In `android/app/build.gradle.kts` set **`minSdk = 26`** and enable **core-library desugaring** (required by the `health` plugin):

```kotlin
android {
    compileOptions {
        isCoreLibraryDesugaringEnabled = true
    }
    defaultConfig {
        minSdk = 26
    }
}

dependencies {
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

Your build must run on **JDK 17+** with **Kotlin 2.2.0+**. See [Prerequisites](01-prerequisites.md) and [Android Gradle Setup](../android/02-gradle-setup.md).

As with iOS, a Flutter host declares the Android permissions itself — they are **not** merged in for you. Add them to `android/app/src/main/AndroidManifest.xml`, inside the `<manifest>` element above `<application>`:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
<uses-permission android:name="android.permission.ACTIVITY_RECOGNITION"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_LOCATION"/>
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
```

For the complete list (including pre-API-31 Bluetooth flags and Health Connect declarations), the Mapbox token, and the per-permission rationale for your Play Console **Data safety** form, see [Android Permissions](../android/03-permissions.md). The SDK requests the runtime prompts itself when the user starts a feature — your Dart code calls no permission API.

> **Mapbox gotcha:** after an SDK version bump, the consumer Gradle cache can reuse stale Mapbox metadata. Fix with `./gradlew --refresh-dependencies` — this is **not** a CI toggle.

## 4. Wire a screen

Fetch a token from your backend, initialize the SDK, then render `RollaSdkHome`. Because a pure-Flutter host embeds the SDK on its own `Navigator`, pass `showBackButton: true` **and** `onRequestDismiss` so the SDK's back button returns to your app — without `onRequestDismiss` the back button does nothing.

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart' as sdk;

class RollaLaunchScreen extends StatefulWidget {
  const RollaLaunchScreen({super.key, required this.userId});

  /// The logged-in user's id, from your backend.
  final String userId;

  @override
  State<RollaLaunchScreen> createState() => _RollaLaunchScreenState();
}

class _RollaLaunchScreenState extends State<RollaLaunchScreen> {
  bool _initializing = true;
  String? _error;

  @override
  void initState() {
    super.initState();
    _initializeSdk();
  }

  Future<void> _initializeSdk() async {
    setState(() {
      _initializing = true;
      _error = null;
    });
    try {
      // Mint Rolla tokens on YOUR backend; never hold a partner secret in the app.
      final session = await myBackend.fetchRollaTokens();

      await sdk.RollaSDK.initializeWithToken(
        accessToken: session.accessToken,
        refreshToken: session.refreshToken,
        tokenExpiresIn: Duration(seconds: session.expiresIn),
        userId: widget.userId,
        partnerId: 'your-partner-id',
        environment: sdk.RollaEnvironment.rnd, // rnd sandbox during integration
        // Show the SDK's top-bar back button...
        showBackButton: true,
        // ...and pop back to the host when it is tapped.
        onRequestDismiss: () {
          if (mounted) Navigator.of(context).pop();
        },
        // The SDK asks the host to refresh on 401. Return null to force logout.
        onTokenExpired: () async {
          try {
            final r = await myBackend.fetchRollaTokens();
            return sdk.TokenRefreshResult(
              accessToken: r.accessToken,
              refreshToken: r.refreshToken,
              expiresIn: Duration(seconds: r.expiresIn),
            );
          } catch (_) {
            return null;
          }
        },
        // Called when the user logs out from inside the SDK.
        onLogout: () {
          if (mounted) Navigator.of(context).pop();
        },
      );

      if (!mounted) return;
      setState(() => _initializing = false);
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _initializing = false;
        _error = e.toString();
      });
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_error != null) {
      return Scaffold(
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text('Could not start Rolla:\n$_error', textAlign: TextAlign.center),
              const SizedBox(height: 16),
              FilledButton(onPressed: _initializeSdk, child: const Text('Retry')),
            ],
          ),
        ),
      );
    }
    if (_initializing) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    // Hand off to the SDK. RollaSdkHome owns everything from here.
    return sdk.RollaSdkHome(userId: widget.userId);
  }
}
```

> **Do not wrap `RollaSdkHome` in your own `MaterialApp`.** It builds its own `MaterialApp.router` internally. Push it as a route from your existing app:
>
> ```dart
> Navigator.of(context).push(
>   MaterialPageRoute<void>(
>     builder: (_) => RollaLaunchScreen(userId: currentUserId),
>   ),
> );
> ```

## 5. Build and run on a physical device

```sh
flutter run --release -d <your-device-id>
```

List connected devices with `flutter devices`. Use `--release` on iOS for hardware-backed features; Debug requires the Dart VM reachable from the device. Navigate to your launch screen — Rolla should initialize and render, and its back button should return you to your app.

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
