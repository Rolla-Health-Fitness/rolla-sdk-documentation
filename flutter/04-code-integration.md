# Code Integration

The entire integration is two calls:

1. `await RollaSDK.initializeWithToken(...)` — once, with the tokens your backend minted.
2. Place `RollaSdkHome(userId: ...)` wherever the SDK should appear.

`RollaSdkHome` does not work before initialization completes — everything it renders depends on the session that `initializeWithToken` sets up. How you place the widget is up to you; the common placements are in [Placing `RollaSdkHome`](#placing-rollasdkhome) below.

## Import

```dart
import 'package:rolla_sdk/rolla_sdk.dart';
```

Everything you need — `RollaSDK`, `RollaSdkHome`, `RollaEnvironment`, `TokenRefreshResult`, `Branding`, `RollaLanguage`, `RollaDisabledModule`, `RollaDataSource`, `RollaScreen`, `JwtDecoder` — is exported from this one barrel file.

## Authentication & token flow

The SDK needs a **user access token** (JWT) to identify the user and authorize API calls. You obtain this token from Rolla's auth API **after** the user has logged in.

- **Typical flow:** User logs in to your app → your app calls your backend → your backend returns `access_token`, `refresh_token`, and `expires_in` from Rolla's auth API → you pass all three into `initializeWithToken`.
- **When to fetch:** Right before initializing, so the SDK starts with the maximum remaining lifetime. If the user is already logged in, use your existing session (a stored pair, or a refresh to get a new access token).
- **What to pass:** All three token fields — `accessToken`, `refreshToken`, and `tokenExpiresIn`. The access token alone opens the SDK, but the other two are what let it keep the session alive on its own — see [Token Management](06-token-management.md#your-apps-responsibilities).
- **Partner ID:** Use the partner ID Rolla gave you. It is fixed per partner, not per user.

> **Note:** You are responsible for authentication — the SDK only consumes the token you provide.

## Initialize with a token

`RollaSDK.initializeWithToken(...)` is the entry point. Your backend mints the SDK tokens via the [Auth API](../sdk-auth-api/02-authentication.md); you pass them in along with your partner ID and the lifecycle callbacks:

```dart
await RollaSDK.initializeWithToken(
  accessToken: session.accessToken,
  refreshToken: session.refreshToken,                   // lets the SDK refresh on its own
  tokenExpiresIn: Duration(seconds: session.expiresIn), // enables proactive refresh
  userId: JwtDecoder.extractUserId(session.accessToken)!, // or your own stable user id
  partnerId: 'your-partner-id',
  environment: RollaEnvironment.rnd,                    // .rnd while integrating; .production when live
  branding: myBranding,                                 // optional, see Configuration
  onTokenExpired: () async { /* return fresh tokens */ },
  onSessionExpired: () { /* the session is dead: show your login */ },
  onLogout: () { /* return to your app */ },
);
```

These are the identity and auth essentials. `initializeWithToken` also takes `branding`, `language`, `disabledModules`, `disabledDataSources`, `showOptionsButton`, `showGoalsSection`, `removeRollaBandReferences`, `isProfileComplete` and the UI chrome flags — see [Configuration](05-configuration.md) for the full reference.

For `userId`, pass the Rolla user ID — the `sub` claim of the login JWT, which `JwtDecoder.extractUserId` reads for you — or a stable identifier of your own. It namespaces the SDK's persisted data per user on shared devices and is never sent to the backend. An empty string makes the SDK fall back to the JWT `sub` claim itself; if neither resolves, `initializeWithToken` throws an `ArgumentError` rather than mixing users' data.

> **Use `RollaEnvironment.rnd` while integrating.** Your starter-package credentials belong to the `rnd` sandbox and won't authenticate against production. The parameter defaults to `.production`, so set it explicitly. Switch to `.production` once Rolla provisions your production credentials.

### Environment values

| Value | Description |
|-------|-------------|
| `RollaEnvironment.production` | Live / release builds (`https://ross.rolla.cloud`) — the default |
| `RollaEnvironment.rnd` | Development and QA sandbox (`https://ross-rnd.rolla.cloud`) |

### Re-initialization

`initializeWithToken` returns once the SDK is ready; after it completes, `RollaSDK.isInitialized` is `true`. Calling it again disposes the previous instance and rebuilds the SDK — kick it off from `initState()` (or a button handler) and show a spinner while it runs; do not call it on every rebuild. Re-initializing for a **different** user first wipes the previous user's local caches (band state, metrics), so two accounts on one device never see each other's data; re-initializing for the same user keeps them.

## Placing `RollaSdkHome`

`RollaSdkHome(userId: ...)` is a regular widget — place it whichever way fits your app. Whatever the placement, the same rules apply:

- **Initialize first.** Render the widget only after `initializeWithToken` completes — guard on your own state flag or `RollaSDK.isInitialized`.
- **Do not wrap it in another `MaterialApp`.** It builds its own `MaterialApp.router` internally and owns navigation, theming, and routing from that point on.
- **Wire the exit for your placement.** A pushed screen needs `showBackButton` + `onRequestDismiss` (next section); an app-root placement exits through `onLogout` and `onSessionExpired` instead.

### Option A — push it as a screen

The pattern the demo app uses, and the right fit when Rolla is one feature of your app: a launch screen initializes the SDK in `initState`, shows a spinner, then returns `RollaSdkHome` from `build`. Your app pushes that screen as an ordinary route:

```dart
Navigator.of(context).push(
  MaterialPageRoute<void>(
    builder: (_) => const RollaLaunchScreen(), // initializes, then shows RollaSdkHome
  ),
);
```

The full launch screen, including error handling and retry, is the [Complete Example](#complete-example) below.

### Option B — make it your app's root

For deployments where Rolla *is* the main experience: run your own login flow, initialize the SDK, then return `RollaSdkHome` as the authenticated home:

```dart
@override
Widget build(BuildContext context) {
  if (!auth.isLoggedIn) return const LoginScreen();
  if (!RollaSDK.isInitialized) return const SplashScreen(); // initializeWithToken in flight
  return RollaSdkHome(userId: auth.userId); // the SDK is the app from here
}
```

There is nothing to dismiss in this topology — leave `showBackButton` off and route back to your login screen from `onLogout` and `onSessionExpired`. This placement is also the one where the SDK's own notification taps land on the right screen, because `RollaSdkHome` is on screen whenever the app is — see [API Reference → Notification taps](07-api-reference.md#notification-taps).

### Option C — gate it with a `FutureBuilder`

A compact variant of Option A. Create the init future **once** (a field — never in `build`, since re-running `initializeWithToken` disposes and rebuilds the SDK):

```dart
class _RollaScreenState extends State<RollaScreen> {
  late final Future<void> _init = RollaSDK.initializeWithToken(/* ... */);

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<void>(
      future: _init,
      builder: (context, snapshot) {
        if (snapshot.connectionState != ConnectionState.done) {
          return const Scaffold(body: Center(child: CircularProgressIndicator()));
        }
        return RollaSdkHome(userId: widget.userId);
      },
    );
  }
}
```

Add error handling via `snapshot.hasError` — or use Option A's explicit state fields, which make the retry flow easier to express.

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

> **Both are required for a pure-Flutter host.** `showBackButton: true` alone renders the button, but tapping it does nothing — the SDK has no way to dismiss itself without `onRequestDismiss`, and the user is left with no way back to your app. (Native add-to-app hosts receive the dismiss over a method channel instead and don't need the callback.)

`onRequestDismiss` is also what the SDK calls when the user presses back on a screen you opened directly with `RollaSDK.openScreen` — that screen is the root of the SDK UI, so back exits to your app. Pass the callback whenever you use `openScreen`, even with `showBackButton` off. See [API Reference → Host-driven navigation](07-api-reference.md#host-driven-navigation).

## Handle logout and session expiry

Pass `onLogout` to learn when the user signs out from inside the SDK, so you can clear your own auth state and route back to your login screen:

```dart
onLogout: () {
  if (mounted) Navigator.of(context).pop(); // or navigate to your login route
},
```

`onLogout` fires after the SDK has already cleared its own tokens and session. To clear the SDK from your side (e.g. when the user logs out of *your* app), call `RollaSDK.logout()`.

`onSessionExpired` is the other way a session ends: the SDK received a `401`, could not refresh on its own, got nothing usable from `onTokenExpired`, and found nothing newer in storage — typically because the session was revoked server-side. Treat it like a logout you did not initiate: clear your session and show your login. It never fires during a deliberate `RollaSDK.logout()`.

## Control the SDK UI chrome

`initializeWithToken` accepts flags that tune what chrome the SDK renders:

| Flag | Default | Effect when changed |
| --- | --- | --- |
| `showBackButton` | `false` | `true` renders a back button in the SDK top bar (pair with `onRequestDismiss`, above). |
| `hideBottomNavigation` | `false` | `true` hides the Home / Profile tabs, leaving only the activity (＋) button — a minimal embed where your app provides the surrounding navigation. |
| `showOptionsButton` | `true` | `false` hides the three-dot options action on the Home app bar (its sheet links to Data Sources, Goals, Leaderboards and the FAQ) — use it if you surface those elsewhere. |
| `showGoalsSection` | `false` | `true` shows the user's goals with an edit action at the bottom of Home. |
| `showAccountSettings` | `false` | `true` exposes credential-management screens (change/reset password, change email, delete account). |

All of them are documented with the rest of the options in [Configuration](05-configuration.md).

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
    return null; // nothing fresher available
  }
},
```

The full lifecycle (internal refresh, `onSessionExpired`, `RollaSDK.updateToken()`, logout) is in [Token Management](06-token-management.md).

## Complete Example

This is the reference launch screen from the `rolla-sdk-demo-flutter` demo app (`lib/screens/rolla_launch_screen.dart`): initialize in `initState`, show a spinner while it runs, surface errors with a retry, then hand off to `RollaSdkHome`.

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart';

class RollaLaunchScreen extends StatefulWidget {
  const RollaLaunchScreen({super.key});

  @override
  State<RollaLaunchScreen> createState() => _RollaLaunchScreenState();
}

class _RollaLaunchScreenState extends State<RollaLaunchScreen> {
  bool _initializing = true;
  String? _error;

  /// The Rolla user id from the login JWT's `sub` claim. A partner with
  /// their own user system can pass its id instead.
  String? _userId;

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
      final userId = JwtDecoder.extractUserId(session.accessToken)!;

      await RollaSDK.initializeWithToken(
        accessToken: session.accessToken,
        refreshToken: session.refreshToken,
        tokenExpiresIn: Duration(seconds: session.expiresIn),
        userId: userId,
        partnerId: 'your-partner-id',
        environment: RollaEnvironment.rnd, // sandbox during integration
        branding: myBranding,              // see Configuration
        // Show the SDK's back button and pop our route when it's tapped.
        showBackButton: true,
        onRequestDismiss: _leaveSdk,
        // Hand back fresh tokens when the SDK asks; null means we have none.
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
        // The user signed out inside the SDK, or the session is unrecoverable.
        onLogout: _leaveSdk,
        onSessionExpired: _leaveSdk,
      );

      if (!mounted) return;
      setState(() {
        _initializing = false;
        _userId = userId;
      });
    } catch (e) {
      if (!mounted) return;
      setState(() {
        _initializing = false;
        _error = e.toString();
      });
    }
  }

  void _leaveSdk() {
    if (mounted) Navigator.of(context).pop();
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

    final userId = _userId;
    if (_initializing || userId == null) {
      return const Scaffold(body: Center(child: CircularProgressIndicator()));
    }

    // Hand off to the SDK. RollaSdkHome owns everything from here.
    return RollaSdkHome(userId: userId);
  }
}
```

Push it from your own screen as an ordinary route ([Option A](#option-a--push-it-as-a-screen)) — Rolla lives alongside your UI, it does not replace your app. Before running on a device, make sure the [Permissions](03-permissions.md) are configured.

---

**Previous:** [Permissions](03-permissions.md) | **Next:** [Configuration](05-configuration.md) | **Home:** [README](README.md)
