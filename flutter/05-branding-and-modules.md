# Branding & Modules

The Flutter SDK takes branding as a real `Branding` object and module tuning as a `Map`, both passed straight into `RollaSDK.initializeWithToken(...)`. There are no build flavors — a partner app constructs the literal itself.

## Custom branding (optional)

Pass a `Branding` instance via the `branding:` parameter. Unlike React Native (which uses hex strings), Flutter colors are real `dart:ui` `Color` values — use the `0xFFRRGGBB` literal, not a `'#RRGGBB'` string.

```dart
import 'package:flutter/material.dart';
import 'package:rolla_sdk/rolla_sdk.dart' show Branding;

const Branding partnerBranding = Branding(
  // Required
  appName: 'Your App Name',
  defaultThemeMode: ThemeMode.system,        // light | dark | system
  primaryColor: Color(0xFF1976D2),           // ARGB literal, not '#1976D2'
  secondaryColor: Color(0xFF625B71),
  accentColor: Color(0xFF7D5260),
  brightness: Brightness.light,              // light | dark

  // Optional
  partnerId: 'your-partner-id',              // Partner-ID header value
  defaultLocale: Locale('en'),               // null falls back to device locale
  termsUrl: 'https://example.com/terms',
  privacyUrl: 'https://example.com/privacy',

  // Optional asset paths (see Branding assets). null = SDK bundled defaults.
  headerLogoAsset: null,
  onboardingImageAsset: null,
  signUpImageAsset: null,
);
```

Then hand it to the initializer (see [Code Integration](04-code-integration.md) for the full call):

```dart
await RollaSDK.initializeWithToken(
  accessToken: token,
  userId: userId,
  partnerId: 'your-partner-id',
  branding: partnerBranding,
);
```

> **Color literals only.** `Color(0xFF1976D2)` is opaque blue (`0xAARRGGBB` — the leading `FF` is the alpha byte). Passing a bare `0x1976D2` yields a fully transparent color. Do not pass a hex string; the field is typed `Color`.

### Required vs. optional fields

| Field | Type | Required |
| --- | --- | --- |
| `appName` | `String` | Yes |
| `defaultThemeMode` | `ThemeMode` | Yes |
| `primaryColor` / `secondaryColor` / `accentColor` | `Color` | Yes |
| `brightness` | `Brightness` | Yes |
| `partnerId` | `String?` | No |
| `defaultLocale` | `Locale?` | No |
| `termsUrl` / `privacyUrl` | `String?` | No |
| `headerLogoAsset` / `onboardingImageAsset` / `signUpImageAsset` | `String?` | No |
| `authTitle` / `authSubtitle` (+ `authTitleI18n` / `authSubtitleI18n`) | `String?` / `Map<String,String>?` | No |

## Branding assets

Image assets referenced by `headerLogoAsset` (and the onboarding/sign-up images) **must be bundled inside the SDK at build time** — they are loaded from the SDK's own asset bundle, not your host app's `pubspec.yaml` assets. Leave the paths `null` to use the SDK defaults.

During onboarding, coordinate with Rolla to supply your logo (SVG preferred) and brand assets. Rolla bundles them into the SDK and gives you the asset path to set here.

See [iOS Branding & Modules → Branding Assets](../ios/05-branding-and-modules.md#branding-assets) and [Android Branding & Modules → Branding Assets](../android/05-branding-and-modules.md) for the rationale — it is the same on Flutter.

## Disabling modules

Pass a `Set<RollaDisabledModule>` to hide a module everywhere in the SDK UI. **Only two modules are disablable today:**

```dart
import 'package:rolla_sdk/rolla_sdk.dart' show RollaDisabledModule;

await RollaSDK.initializeWithToken(
  accessToken: token,
  userId: userId,
  partnerId: 'your-partner-id',
  branding: partnerBranding,
  disabledModules: { RollaDisabledModule.weight, RollaDisabledModule.bloodPressure },
);
```

`RollaDisabledModule` exposes exactly `weight` and `bloodPressure`. Every other module ships always-on; broader selective enablement will arrive in a future release.

## Tuning modules (`moduleConfigs`)

For modules that stay enabled, pass per-module options as a `Map<RollaModuleType, RollaModuleConfig>`. This does not enable or disable modules — it adjusts behaviour within them.

```dart
import 'package:rolla_sdk/rolla_sdk.dart'
    show RollaModuleType, WeightModuleConfig, ProfileModuleConfig;

await RollaSDK.initializeWithToken(
  accessToken: token,
  userId: userId,
  partnerId: 'your-partner-id',
  branding: partnerBranding,
  moduleConfigs: {
    RollaModuleType.weight: WeightModuleConfig(showBMI: false, allowTarget: false),
    RollaModuleType.profile: ProfileModuleConfig(showPicture: false),
  },
);
```

Common config classes: `WeightModuleConfig`, `BloodPressureModuleConfig`, `ProfileModuleConfig`, `GoalsModuleConfig`. The keys are `RollaModuleType` values, and the identifiers match the native SDKs.

> **`disabledModules` vs. `moduleConfigs`.** Use `disabledModules` to remove `weight` / `bloodPressure` entirely; use `moduleConfigs` to keep a module but turn parts of it off. They are independent parameters.

For the complete list of modules and what each one does, see [iOS Branding & Modules → Available Modules](../ios/05-branding-and-modules.md#available-modules) and [Android Branding & Modules → Available Modules](../android/05-branding-and-modules.md#available-modules). The module identifiers are the same across platforms.

---

**Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
