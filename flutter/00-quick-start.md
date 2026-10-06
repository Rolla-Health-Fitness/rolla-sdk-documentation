# Quick Start — Flutter

Get the Rolla SDK running in your Flutter app in under 10 minutes.

> **This guide covers the minimal integration.** For branding, modules, host-driven navigation, the headless calls, and token details, see the [full documentation](README.md).

## Prerequisites

- **Flutter 3.35.6+ / Dart 3.9.2+**
- **iOS 14.0+** deployment target (`platform :ios, '14.0'` in `ios/Podfile`)
- **Android `minSdk 26`**, **Kotlin 2.2.0+**, **JDK 17** to build, and **core library desugaring** — see [Installation](02-installation.md)
- **Partner ID and sandbox (`rnd`) credentials** from your Rolla SDK starter package (contact [support@rolla.app](mailto:support@rolla.app))
- **A physical device** for Bluetooth band pairing and motion-sensor tracking; simulators and emulators run everything else

## 1. Add the Package

```sh
flutter pub add rolla_sdk:0.1.15
```

This pins the current release in your `pubspec.yaml`:

```yaml
dependencies:
  rolla_sdk: 0.1.15
```

Pin the exact version: 0.1.x releases can carry breaking changes, so upgrade deliberately (see [Prerequisites → Versioning](01-prerequisites.md#versioning)). Then apply the platform floors (iOS deployment target, Android `minSdk` and desugaring) from [Installation](02-installation.md).

## 2. Declare the Platform Permissions

A fresh `flutter create` scaffold has none of the declarations the SDK needs. Add the usage-description keys, `MBXAccessToken` and `UIBackgroundModes` to `ios/Runner/Info.plist` and the HealthKit capability to the Runner target; add the Mapbox token to `strings.xml` and the Health Connect entries to `AndroidManifest.xml`. Without the iOS usage strings the app aborts with SIGABRT the moment the SDK starts — there is no Dart error to catch. The complete lists are in [Permissions & Entitlements](03-permissions.md).

## 3. Get a Token

Your Rolla SDK starter package contains your **Partner ID** and **sandbox credentials**. Use them to obtain tokens from the sandbox auth API:

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/login" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com&password=SecurePassword123"
```

The response contains everything step 4 needs:

- `access_token` → `accessToken`
- `refresh_token` → `refreshToken`
- `expires_in` → `tokenExpiresIn` (seconds; step 4 wraps it in a `Duration`)
- `userId` — the access token is a JWT whose `sub` claim is the Rolla user ID. The package exports a decoder for it: `JwtDecoder.extractUserId(accessToken)`. Your own stable user id works too.

See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the full flow (`/api/register` → `/api/login` → tokens).

## 4. Initialize the SDK

Call `RollaSDK.initializeWithToken(...)` with the tokens before rendering any SDK UI:

```dart
import 'package:rolla_sdk/rolla_sdk.dart';

// Inside a State<...> method — see step 5 for the full screen.
final session = await myBackend.fetchRollaTokens(); // your POST /api/login call
final userId = JwtDecoder.extractUserId(session.accessToken)!; // the JWT's `sub` claim (step 3), or your own id

await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,
  tokenExpiresIn: Duration(seconds: session.expiresIn),
  userId: userId,
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd, // sandbox; .production for release builds

  // Let the user exit the SDK back to your app (see step 5).
  showBackButton: true,
  onRequestDismiss: _leaveSdk,

  // Hand back fresh tokens when the SDK asks; null means you have none.
  onTokenExpired: () async {
    final r = await myBackend.fetchRollaTokens();
    return TokenRefreshResult(
      accessToken: r.accessToken,
      refreshToken: r.refreshToken,
      expiresIn: Duration(seconds: r.expiresIn),
    );
  },

  // The user signed out inside the SDK, or the session is unrecoverable — return to your app.
  onLogout: _leaveSdk,
  onSessionExpired: _leaveSdk,
);
```

`_leaveSdk` pops the SDK route and is safe to call more than once (`onSessionExpired` can fire again for later failing requests):

```dart
void _leaveSdk() {
  if (mounted) Navigator.of(context).pop();
}
```

> **Give the user a way back to your app.** `showBackButton: true` renders a back button in the SDK's top bar. `onRequestDismiss` is the callback that button invokes — typically `Navigator.pop()` to close the SDK screen. Pass both: without `onRequestDismiss` the back button renders but does nothing. See [Code Integration → Host dismissal](04-code-integration.md#host-dismissal--showbackbutton--onrequestdismiss).

> **Use `RollaEnvironment.rnd` while integrating.** Your starter-package credentials belong to the `rnd` sandbox (`https://ross-rnd.rolla.cloud`) and won't authenticate against production. The parameter defaults to `.production`, so set it explicitly. Switch to `.production` once Rolla provisions your production credentials.

## 5. Use the SDK

Once initialization completes, render `RollaSdkHome` — the single widget that hosts the entire SDK experience. Pass it the same `userId` you passed to `initializeWithToken`. It is a regular widget, so place it however fits your app: pushed as its own screen (this quick start and the demo), as your app's root, or behind a `FutureBuilder` — see [Code Integration → Placing `RollaSdkHome`](04-code-integration.md#placing-rollasdkhome). Whatever the placement, initialize first.

```dart
class RollaLaunchScreen extends StatefulWidget {
  const RollaLaunchScreen({super.key});

  @override
  State<RollaLaunchScreen> createState() => _RollaLaunchScreenState();
}

class _RollaLaunchScreenState extends State<RollaLaunchScreen> {
  String? _userId; // set by step 4 once initializeWithToken completes

  void _leaveSdk() {
    if (mounted) Navigator.of(context).pop();
  }

  @override
  void initState() {
    super.initState();
    _initializeSdk(); // step 4, then setState(() => _userId = userId)
  }

  @override
  Widget build(BuildContext context) {
    final userId = _userId;
    if (userId == null) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    // Hand off to the SDK. RollaSdkHome owns everything from here.
    return RollaSdkHome(userId: userId);
  }
}
```

Push it from your app as an ordinary route:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(builder: (_) => const RollaLaunchScreen()),
);
```

> **`RollaSdkHome` is a complete app shell** — it brings its own navigation and theming. Your app's root `MaterialApp` stays as it is; just don't wrap `RollaSdkHome` in a new one — return it directly from your route.

Run the app (`flutter run`) — the SDK initializes and renders, and its back button returns the user to your app. Bluetooth band pairing and motion-sensor tracking need a physical device; simulators and emulators run everything else.

For the production-ready version of this screen (error handling, retry), see [Code Integration](04-code-integration.md).

## 6. Logout Cleanup

When the user logs out of your app, clear the SDK session so the next user cannot inherit tokens or data:

```dart
await RollaSDK.logout();
```

See [Token Management → Logging Out](06-token-management.md#logging-out).

## Next Steps

- **Configuration:** Customize branding, force a UI language, and control module and data-source visibility — [Configuration](05-configuration.md)
- **Token details:** Full token lifecycle and edge cases — [Token Management](06-token-management.md)
- **Beyond the Home screen:** `openScreen`, the headless calls, notification taps — [API Reference](07-api-reference.md)

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
