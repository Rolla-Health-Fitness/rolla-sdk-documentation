# Code Integration

The SDK exposes a static `RollaSDK` class for initialization and a single widget, `RollaSdkHome`, that renders the entire SDK experience. You initialize once with a token, then hand off a route to `RollaSdkHome`.

## Import

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

Everything you need — `RollaSDK`, `RollaSdkHome`, `RollaEnvironment`, `TokenRefreshResult`, `Branding`, `RollaDisabledModule` — is exported from this one barrel file.

## Initialize with a token

`RollaSDK.initializeWithToken(...)` is the entry point. Your backend mints the SDK tokens via the [Auth API](../sdk-auth-api/02-authentication.md); you pass them in along with your partner ID and the lifecycle callbacks:

```dart
await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,            // optional
  tokenExpiresIn: const Duration(seconds: 1800), // optional, enables proactive refresh
  userId: 'user-123',                            // your logged-in user's id or user email
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd,             // .rnd while integrating; .production when live
  branding: myBranding,                          // optional, see Branding & Modules
  onTokenExpired: () async { /* return fresh tokens */ },
  onLogout: () { /* return to your app */ },
);
```

> **Use `RollaEnvironment.rnd` while integrating.** Your starter-package credentials are sandbox credentials, so the environment must be `.rnd` to match — and the Dart default is `.production`, so set it explicitly. Switch to `.production` once Rolla provisions your production credentials.

`initializeWithToken` resets any prior instance and returns once the SDK is ready; after it completes, `RollaSDK.isInitialized` is `true`. Kick it off from `initState()` (or a button handler) and show a spinner while it runs — re-running it disposes and rebuilds the SDK, so do not call it on every rebuild.

## Render `RollaSdkHome`

Once initialized, render the SDK by returning `RollaSdkHome(userId: ...)` from a route:

```dart
return RollaSdkHome(userId: widget.userId);
```

> **Do not wrap `RollaSdkHome` in another `MaterialApp`.** It builds its own `MaterialApp.router` internally and owns navigation, theming, and routing from that point on. Push it onto a route from your existing app instead (`Navigator.push` / `GoRoute`).

## Host dismissal — `showBackButton` + `onRequestDismiss`

Because `RollaSdkHome` owns its own router, a back button inside the SDK cannot pop *your* `Navigator` by itself. To let the user exit the SDK and return to your app, pass both:

```dart
await RollaSDK.initializeWithToken(
  // ...
  showBackButton: true,                        // render a back button in the SDK top bar
  onRequestDismiss: () {                       // SDK calls this when that button is tapped
    if (mounted) Navigator.of(context).pop();  // pop back to your screen
  },
);
```

> **Both are required for a pure-Flutter host.** `showBackButton: true` alone renders the button, but tapping it does nothing — the SDK has no way to dismiss itself without `onRequestDismiss`, and the user is left with no way back to your app. (Native add-to-app hosts receive the dismiss over a method channel instead and don't need the callback.) Requires `rolla_sdk` **0.1.12** or newer.

## Handle logout

Pass `onLogout` to learn when the user signs out from inside the SDK, so you can clear your own auth state and route back to your login screen:

```dart
onLogout: () {
  if (mounted) Navigator.of(context).pop(); // or navigate to your login route
},
```

`onLogout` fires after the SDK has already cleared its own tokens and session. To clear the SDK from your side (e.g. when the user logs out of *your* app), call `RollaSDK.logout()`.

## Control the SDK UI chrome

`initializeWithToken` accepts flags that tune what chrome the SDK renders:

| Flag | Default | Effect when changed |
| --- | --- | --- |
| `showBackButton` | `false` | `true` renders a back button in the SDK top bar (pair with `onRequestDismiss`, above). |
| `hideBottomNavigation` | `false` | `true` hides the Home / Profile tabs, leaving only the activity (＋) button — a minimal embed. |
| `showSettingsButton` | `true` | `false` hides the Home **Settings** shortcut (use if you surface Data Sources / Goals elsewhere). |
| `showAccountSettings` | `false` | `true` exposes credential-management screens (change/reset password, change email, delete account). |

> **`hideBottomNavigation` defaults differ by integration path.** Native iOS/Android hosts default it to `true`, but the Flutter `initializeWithToken` defaults it to `false` (full bottom navigation shown). To match the minimal chrome of the native integrations, pass `hideBottomNavigation: true` explicitly.

## Handle token refresh

When the SDK's access token expires and it cannot refresh internally, it calls `onTokenExpired`. Return a `TokenRefreshResult` with fresh credentials, or `null` if you could not refresh:

```dart
onTokenExpired: () async {
  try {
    final refreshed = await myBackend.fetchRollaTokens();
    return TokenRefreshResult(
      accessToken: refreshed.accessToken,
      refreshToken: refreshed.refreshToken,
      expiresIn: Duration(seconds: refreshed.expiresIn),
    );
  } catch (_) {
    return null; // refresh failed
  }
},
```

The full lifecycle (internal refresh, `RollaSDK.updateToken()`, logout) is in [Token Management](06-token-management.md).

## Complete Example

This is the reference launch screen from the `rolla-sdk-demo-flutter` demo app (`lib/screens/rolla_launch_screen.dart`): initialize in `initState`, show a spinner while it runs, surface errors with a retry, then hand off to `RollaSdkHome`.

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart';

class RollaLaunchScreen extends StatefulWidget {
  const RollaLaunchScreen({super.key, required this.userId});
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
      final session = await myBackend.fetchRollaTokens();

      await RollaSDK.initializeWithToken(
        accessToken: session.accessToken,
        refreshToken: session.refreshToken,
        tokenExpiresIn: Duration(seconds: session.expiresIn),
        userId: widget.userId,
        partnerId: 'your-partner-id',
        environment: RollaEnvironment.rnd, // sandbox during integration
        branding: myBranding,              // see Branding & Modules
        // Show the SDK's back button and pop our route when it's tapped.
        showBackButton: true,
        onRequestDismiss: () {
          if (mounted) Navigator.of(context).pop();
        },
        // Hand back fresh tokens when the SDK asks; null signals refresh failed.
        onTokenExpired: () async {
          try {
            final refreshed = await myBackend.fetchRollaTokens();
            return TokenRefreshResult(
              accessToken: refreshed.accessToken,
              refreshToken: refreshed.refreshToken,
              expiresIn: Duration(seconds: refreshed.expiresIn),
            );
          } catch (_) {
            return null;
          }
        },
        // User logged out from inside the SDK — return to our app.
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
    return RollaSdkHome(userId: widget.userId);
  }
}
```

Push it from your own screen as an ordinary route — Rolla lives alongside your UI, it does not replace your app:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(
    builder: (_) => RollaLaunchScreen(userId: currentUser.id),
  ),
);
```

Before running on a device, make sure the [Permissions](03-permissions.md) are configured.

---

**Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
