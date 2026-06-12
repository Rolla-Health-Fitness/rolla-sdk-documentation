# Branding & Modules

Branding is a `Branding` object and module tuning a `Map`, both passed straight into `RollaSDK.initializeWithToken(...)`. There are no build flavors — your app constructs the values itself.

## Custom branding (optional)

Pass a `Branding` instance via the `branding:` parameter. Colors are `dart:ui` `Color` values — use the `0xAARRGGBB` literal, not a hex string:

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart' show Branding;

const Branding partnerBranding = Branding(
  // Required
  appName: 'Your App Name',
  defaultThemeMode: ThemeMode.system,        // light | dark | system
  primaryColor: Color(0xFF1976D2),
  secondaryColor: Color(0xFF625B71),
  accentColor: Color(0xFF7D5260),
  brightness: Brightness.light,              // light | dark

  // Optional
  partnerId: 'your-partner-id',              // Partner-ID header value
  defaultLocale: Locale('en'),               // null falls back to device locale
  termsUrl: 'https://example.com/terms',
  privacyUrl: 'https://example.com/privacy',

  // Optional header logo — an SVG bundled inside the SDK (see Branding assets).
  headerLogoAsset: null,
);
```

Then hand it to the initializer (see [Code Integration](04-code-integration.md) for the full call):

```dart
await RollaSDK.initializeWithToken(
  // ... credentials, environment, and callbacks ...
  branding: partnerBranding,
);
```

> **Color literals only.** `Color(0xFF1976D2)` is opaque blue — the leading `FF` is the alpha byte. Passing a bare `0x1976D2` yields a fully transparent color.

The full field list (including the auth-screen text and background overrides) is in [API Reference → Branding](08-api-reference.md#branding).

## Branding assets

The header logo must be an **SVG** — the SDK renders it with its own SVG widget, so raster formats will not load. It also **must be bundled inside the SDK at build time**: assets are loaded from the SDK's own bundle, not your host app's `pubspec.yaml` assets. Leave the path `null` to use the SDK default.

During onboarding, send Rolla your logo as an SVG; Rolla bundles it into the SDK and gives you the asset path to set here. The `Branding` class also declares onboarding/sign-up image slots (`onboardingImageAsset`, `signUpImageAsset`) — these are reserved and not rendered by this version of the SDK. See [iOS Branding & Modules → Branding Assets](../ios/05-branding-and-modules.md#branding-assets) for the same model on the native side.

## Disabling modules

Pass a `Set<RollaDisabledModule>` to hide a module everywhere in the SDK UI. Two modules can be disabled in this release:

```dart
import 'package:rolla_sdk/rolla_sdk.dart' show RollaDisabledModule;

await RollaSDK.initializeWithToken(
  // ...
  disabledModules: { RollaDisabledModule.weight, RollaDisabledModule.bloodPressure },
);
```

`RollaDisabledModule` exposes exactly `weight` and `bloodPressure`; every other module is always on. Broader selective enablement is planned for a future release.

For the complete list of modules and what each one does, see [iOS Branding & Modules → Available Modules](../ios/05-branding-and-modules.md#available-modules) and [Android Branding & Modules → Available Modules](../android/05-branding-and-modules.md#available-modules). The module identifiers are the same across platforms.

---

**Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
