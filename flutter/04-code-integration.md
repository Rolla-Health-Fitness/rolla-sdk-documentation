# Code Integration

The Rolla SDK exposes a static `RollaSDK` class for initialization and a single widget, `RollaSdkHome`, that renders the entire SDK experience. You initialize once with a token, then hand off a route to `RollaSdkHome`.

## Import

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

Everything you need — `RollaSDK`, `RollaSdkHome`, `RollaEnvironment`, `TokenRefreshResult`, `Branding`, `RollaDisabledModule` — is exported from this one barrel file.

## Initialize with a token

`RollaSDK.initializeWithToken(...)` is the primary entry point. Your backend mints the SDK access token (see [Token Management](06-token-management.md)); you pass it in along with your partner ID and the lifecycle callbacks.

```dart
await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,        // optional
  tokenExpiresIn: const Duration(seconds: 1800), // optional, enables proactive refresh
  userId: 'user-123',                         // your logged-in user's id
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd,          // start on .rnd; switch to .production when you go live
  branding: myBranding,                       // optional, see Branding & Modules
  onTokenExpired: () async { /* return fresh tokens */ },
  onLogout: () { /* return to your app */ },
);
```

> **Use `RollaEnvironment.rnd` while integrating.** Rolla issues you **rnd** (research-and-development) sandbox credentials for onboarding — your tokens are minted against `https://ross-rnd.rolla.cloud`, so `environment` must be `.rnd` to match (a mismatch authenticates against the wrong backend and fails). Switch to `.production` only once Rolla provisions your production credentials.

`initializeWithToken` resets any prior instance and returns once the SDK is ready. Call it before you render `RollaSdkHome`. After it completes, `RollaSDK.isInitialized` is `true`.

> **Initialize off the first frame, not in `build()`.** Kick it off from `initState()` (or a button handler) and show a spinner while it runs — the demo's `RollaLaunchScreen` does exactly this. Re-running `initializeWithToken` disposes and rebuilds the SDK, so do not call it on every rebuild.

## Render `RollaSdkHome`

Once initialized, render the SDK by returning `RollaSdkHome(userId: ...)` from a route:

```dart
return RollaSdkHome(userId: widget.userId);
```

> **Do NOT wrap `RollaSdkHome` in another `MaterialApp`.** It builds its own `MaterialApp.router` internally and owns navigation, theming, and routing from that point on. Nesting it inside your own `MaterialApp` breaks routing and theming. Push it onto a route from your existing app instead (`Navigator.push` / `GoRoute`).

## Host dismissal — `showBackButton` + `onRequestDismiss`

Because `RollaSdkHome` owns its own router, a back button inside the SDK cannot pop *your* `Navigator` by itself. To let the SDK's top-bar back button return the user to your app, pass both:

```dart
await RollaSDK.initializeWithToken(
  // ...
  showBackButton: true,                       // render a back button in the SDK top bar
  onRequestDismiss: () {                       // SDK calls this when that button is tapped
    if (mounted) Navigator.of(context).pop();  // pop back to your screen
  },
);
```

> **`onRequestDismiss` is new in 0.1.12 and is required for pure-Flutter hosts.** A Flutter app that embeds `RollaSdkHome` on its own `Navigator` has no native side listening on the `rolla_sdk/init` method channel, so `showBackButton: true` alone does nothing — the SDK has no way to dismiss itself. Pass `onRequestDismiss` to pop your route. (Native add-to-app hosts that present Flutter as a `FlutterViewController`/`FlutterActivity` receive the dismiss over the method channel instead and don't need this callback.)

## Handle logout

Pass `onLogout` to learn when the user signs out from inside the SDK so you can clear your own auth state and route back to your login screen:

```dart
onLogout: () {
  if (mounted) Navigator.of(context).pop(); // or navigate to your login route
},
```

`onLogout` fires after the SDK clears its own tokens and session. If you want to clear the SDK from your side (e.g. when the user logs out of *your* app), call `RollaSDK.logout()`.

## Control the SDK UI chrome

`initializeWithToken` accepts flags that tune what chrome the SDK renders. The **Flutter defaults** are:

| Flag | Flutter default | Effect when set |
| --- | --- | --- |
| `hideBottomNavigation` | `false` | `true` hides the bottom navigation (Home / Profile tabs), leaving only the activity (＋) button — a minimal embed. |
| `showBackButton` | `false` | `true` renders a back button in the SDK top bar (pair with `onRequestDismiss`, above). |
| `showSettingsButton` | `true` | `false` hides the Home **Settings** shortcut (use if you surface Data Sources / Goals elsewhere). |
| `showAccountSettings` | `false` | `true` exposes credential-management screens (change/reset password, change email, delete account). Leave `false` unless your app delegates credential management to the SDK. |

```dart
await RollaSDK.initializeWithToken(
  // ...
  hideBottomNavigation: true,   // minimal chrome: only the activity button
  showSettingsButton: false,    // you surface Data Sources / Goals in your own UI
);
```

> **`hideBottomNavigation` defaults differ by integration path.** Native iOS/Android hosts (via `RollaConfiguration`) default it to **`true`**, but the Flutter `initializeWithToken` defaults it to **`false`** (full bottom navigation shown). If you're matching the minimal chrome the native integrations get out of the box, pass `hideBottomNavigation: true` explicitly.

## Handle token refresh

When the SDK's access token expires and it cannot refresh internally, it calls `onTokenExpired`. Return a `TokenRefreshResult` with fresh credentials, or `null` to force a logout:

```dart
onTokenExpired: () async {
  try {
    final refreshed = await myBackend.fetchTokens();
    return TokenRefreshResult(
      accessToken: refreshed.accessToken,
      refreshToken: refreshed.refreshToken,           // optional
      expiresIn: Duration(seconds: refreshed.expiresIn), // optional Duration
    );
  } catch (_) {
    return null; // forces logout
  }
},
```

The full lifecycle (proactive refresh, `RollaSDK.updateToken()`, forced logout) is in [Token Management](06-token-management.md). The conceptual model is identical to [iOS Token Management](../ios/07-token-management.md) and [Android Token Management](../android/06-token-management.md).

## Full minimal integration

This is the demo's `RollaLaunchScreen`, trimmed to the essentials: initialize in `initState`, show a spinner while it runs, surface errors with a retry, then hand off to `RollaSdkHome`. Source: [`rolla-sdk-demo-flutter`](https://github.com/Rolla-Health-Fitness/rolla-sdk-demo-flutter) (`lib/screens/rolla_launch_screen.dart`).

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
      final session = await myBackend.fetchTokens();

      await RollaSDK.initializeWithToken(
        accessToken: session.accessToken,
        refreshToken: session.refreshToken,
        tokenExpiresIn: Duration(seconds: session.expiresIn),
        userId: widget.userId,
        partnerId: 'your-partner-id',
        environment: RollaEnvironment.rnd, // rnd sandbox during integration
        branding: myBranding, // see Branding & Modules
        // Show the SDK's back button and pop our route when it's tapped.
        showBackButton: true,
        onRequestDismiss: () {
          if (mounted) Navigator.of(context).pop();
        },
        // Hand back fresh tokens when the SDK asks; null forces logout.
        onTokenExpired: () async {
          try {
            final refreshed = await myBackend.fetchTokens();
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

> **Before you build:** if you skipped it, configure [Permissions](03-permissions.md) first — a Flutter host **must** add the iOS usage-description strings and Android manifest permissions itself, or the SDK SIGABRT-crashes on iOS the moment it touches Bluetooth.

---

**Next:** [Branding & Modules](05-branding-and-modules.md) | **Home:** [README](README.md)
