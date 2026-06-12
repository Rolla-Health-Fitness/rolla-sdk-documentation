# Quick Start — Flutter

Get the Rolla SDK running in your Flutter app in under 10 minutes.

> **This guide covers the minimal integration.** For permissions, branding, modules, and token details, see the [full documentation](README.md).

## Prerequisites

- **Flutter 3.35.6+ / Dart 3.9.2+**
- **iOS 14.0+** deployment target (`platform :ios, '14.0'` in `ios/Podfile`)
- **Android `minSdk 26`**, **build JDK 17+**, **Kotlin 2.2.0+**, and **core library desugaring** — see [Installation](02-installation.md)
- **Partner ID and sandbox credentials** from your Rolla SDK starter package (contact [support@rolla.app](mailto:support@rolla.app))
- **Physical devices** for hardware-backed features (Bluetooth, GPS)

Before running on a device, add the SDK's permission declarations to `ios/Runner/Info.plist` and `android/app/src/main/AndroidManifest.xml` — a Flutter host declares these itself, and iOS aborts with SIGABRT if a usage string is missing. See [Permissions](03-permissions.md).

## 1. Add the package

```sh
flutter pub add rolla_sdk
```

This adds the current release to your `pubspec.yaml`:

```yaml
dependencies:
  rolla_sdk: ^0.1.12
```

Then apply the platform floors (iOS deployment target, Android `minSdk` and desugaring) from [Installation](02-installation.md).

## 2. Get a token

Your Rolla SDK starter package contains your **Partner ID** and **rnd sandbox credentials**. Use them to obtain tokens from the sandbox auth API:

```bash
curl -X POST "https://ross-rnd.rolla.cloud/api/login" \
  -H "Partner-ID: your-partner-id" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "email=user@example.com&password=SecurePassword123"
```

The response contains everything step 3 needs:

- `access_token` → `accessToken`
- `refresh_token` → `refreshToken`
- `expires_in` → `tokenExpiresIn` (seconds; step 3 wraps it in a `Duration`)
- `userId` — the access token is a JWT whose `sub` claim is the Rolla user ID; decode it (or use your own user id / email)

See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the full flow (`/api/register` → `/api/login` → tokens).

## 3. Initialize the SDK

Call `RollaSDK.initializeWithToken(...)` with the tokens before rendering any SDK UI:

```dart
import 'package:rolla_sdk/rolla_sdk.dart';

// Inside a State<...> method — see step 4 for the full screen.
final session = await myBackend.fetchRollaTokens(); // your POST /api/login call

await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,
  tokenExpiresIn: Duration(seconds: session.expiresIn),
  userId: 'your-user-id',            // the login JWT's `sub` claim (step 2), or your own id / email
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd, // sandbox; .production for release builds

  // Let the user exit the SDK back to your app (see step 4).
  showBackButton: true,
  onRequestDismiss: () => Navigator.of(context).pop(),

  // Hand back fresh tokens when the SDK asks; null signals refresh failed.
  onTokenExpired: () async {
    final r = await myBackend.fetchRollaTokens();
    return TokenRefreshResult(
      accessToken: r.accessToken,
      refreshToken: r.refreshToken,
      expiresIn: Duration(seconds: r.expiresIn),
    );
  },

  // User logged out from inside the SDK — return to your app.
  onLogout: () => Navigator.of(context).pop(),
);
```

> **Give the user a way back to your app.** `showBackButton: true` renders a back button in the SDK's top bar. `onRequestDismiss` is the callback that button invokes — typically `Navigator.pop()` to close the SDK screen. Pass both: without `onRequestDismiss` the back button renders but does nothing. See [Code Integration → Host dismissal](04-code-integration.md#host-dismissal--showbackbutton--onrequestdismiss).

> **Use `RollaEnvironment.rnd` while integrating.** Your starter-package credentials are sandbox credentials, minted against `https://ross-rnd.rolla.cloud` — and the Dart default is `.production`, so set this explicitly. Switch to `.production` once Rolla provisions your production credentials.

## 4. Use the SDK

Once initialization completes, render `RollaSdkHome` — the single widget that hosts the entire SDK experience. It is a regular widget, so place it however fits your app: pushed as its own screen (this quick start and the demo), as your app's root, or behind a `FutureBuilder` — see [Code Integration → Placing `RollaSdkHome`](04-code-integration.md#placing-rollasdkhome). Whatever the placement, initialize first.

```dart
class RollaLaunchScreen extends StatefulWidget {
  const RollaLaunchScreen({super.key, required this.userId});
  final String userId;

  @override
  State<RollaLaunchScreen> createState() => _RollaLaunchScreenState();
}

class _RollaLaunchScreenState extends State<RollaLaunchScreen> {
  bool _initializing = true;

  @override
  void initState() {
    super.initState();
    _initializeSdk(); // step 3, then setState(() => _initializing = false)
  }

  @override
  Widget build(BuildContext context) {
    if (_initializing) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }
    // Hand off to the SDK. RollaSdkHome owns everything from here.
    return RollaSdkHome(userId: widget.userId);
  }
}
```

Push it from your app as an ordinary route:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(
    builder: (_) => RollaLaunchScreen(userId: currentUser.id),
  ),
);
```

> **Do not wrap `RollaSdkHome` in your own `MaterialApp`.** It builds its own `MaterialApp.router` internally and owns navigation and theming from that point on.

Run on a physical device (`flutter run --release -d <device-id>`) — the SDK initializes and renders, and its back button returns the user to your app.

For the production-ready version of this screen (error handling, retry), see [Code Integration](04-code-integration.md).

## Next Steps

- **Permissions:** Add the required Info.plist keys and manifest entries — [Permissions](03-permissions.md)
- **Branding:** Customize colors, logos, and enabled modules — [Branding & Modules](05-branding-and-modules.md)
- **Token details:** Full token lifecycle and edge cases — [Token Management](06-token-management.md)
- **API Reference:** All methods, callbacks, and types — [API Reference](08-api-reference.md)

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
