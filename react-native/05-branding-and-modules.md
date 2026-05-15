# Branding & Modules

The wrapper accepts the same branding shape as the native SDKs, exposed as a plain TypeScript object. Module configuration is forwarded to the native side untouched — the canonical list lives in the platform docs.

## Custom branding (optional)

```ts
import { Rolla, RollaBranding } from '@rolla-health/react-native-sdk';

const branding: RollaBranding = {
  appName: 'Your App Name',
  primaryColor: '#1976D2',
  secondaryColor: '#625B71',
  accentColor: '#7D5260',
  brightness: 'light',           // 'light' | 'dark'
  defaultThemeMode: 'system',    // 'light' | 'dark' | 'system'
  defaultLocale: 'en-US',        // BCP-47 tag; optional
  headerLogoAsset: undefined,    // optional, see below
  termsUrl: 'https://example.com/terms',
  privacyUrl: 'https://example.com/privacy',
};

await Rolla.show({
  token,
  partnerId: 'your-partner-id',
  environment: 'production',
  branding,
});
```

### Color format

Colors are passed as hex strings (`'#RRGGBB'` or `'#RRGGBBAA'`) and parsed natively on both platforms. **Do not** use React Native's `processColor` — it returns a platform-specific integer that the wrapper does not accept.

This differs from the native SDKs:
- iOS native expects `UIColor` instances. The wrapper converts hex → `UIColor` for you.
- Android native expects ARGB `Int` (`0xFF1976D2.toInt()`). The wrapper converts hex → `Int` for you.

## Branding assets

Image assets referenced by `headerLogoAsset` (and any custom icons) **must be pre-bundled inside the native SDK at build time**. They cannot be shipped from the React Native bundle at runtime — the SDK is a separate Flutter engine and does not have access to your RN asset registry.

During onboarding, coordinate with Rolla to supply your logo (SVG preferred) and any other brand assets. Rolla will bundle these into the SDK and provide the asset path to use in `headerLogoAsset`.

See [iOS Branding & Modules → Branding Assets](../ios/05-branding-and-modules.md#branding-assets) and [Android Branding & Modules → Branding Assets](../android/05-branding-and-modules.md) for the rationale.

## Module configuration

The SDK is organized into modules (Health & Fitness, Activity Tracking, Device Management, User & Settings). Currently all modules are enabled and `disabledModules` is reserved for future use:

```ts
await Rolla.show({
  token,
  partnerId: 'your-partner-id',
  environment: 'production',
  // disabledModules: ['WEIGHT'],  // not yet honored
});
```

The wrapper accepts `disabledModules?: string[]` so your TypeScript will compile, but the native side currently ignores it. Selective module enablement will be available in a future release.

For the complete list of modules and what each one does, see [iOS Branding & Modules → Available Modules](../ios/05-branding-and-modules.md#available-modules) and [Android Branding & Modules → Available Modules](../android/05-branding-and-modules.md). The module identifiers are the same across platforms.

## `showSettingsButton`

```ts
await Rolla.show({
  token,
  partnerId: 'your-partner-id',
  environment: 'production',
  showSettingsButton: false,
});
```

When `false`, the SDK hides its in-modal settings entry. Use when your host app already provides the settings affordance and you don't want it duplicated inside the Rolla modal.

---

**Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
