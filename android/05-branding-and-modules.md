# Branding & Modules

Customize the SDK appearance through branding configuration and manage which modules are enabled in your integration.

## 4. Custom Branding (Optional)

Colors are passed as ARGB integers (e.g. `0xFF6750A4.toInt()`):

```kotlin
val branding = RollaBranding(
    appName = "Your App Name",
    primaryColor = 0xFF1976D2.toInt(),         // Blue
    secondaryColor = 0xFF625B71.toInt(),       // Gray
    accentColor = 0xFF7D5260.toInt(),          // Accent
    brightness = "light",                       // or "dark"
    defaultThemeMode = "system",               // "light", "dark", or "system"
    defaultLocale = "en",                      // Optional
    headerLogoAsset = null,                    // Optional: partner logo asset path (provided by Rolla)
    termsUrl = "https://example.com/terms",    // Optional
    privacyUrl = "https://example.com/privacy" // Optional
)

val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    branding = branding
)
```

> **Branding assets:** Image assets (such as partner logos) must be pre-bundled inside the SDK — they cannot be transferred from the host app at runtime. See the [iOS Integration Guide (Branding & Modules)](../ios/05-branding-and-modules.md) for details on coordinating branding assets with Rolla.

## 5. Module Configuration

The SDK is organized into modules. Currently, all modules are always enabled — pass `null` for the `modules` parameter (or omit it entirely).

```kotlin
val configuration = RollaConfiguration(
    token = token,
    partnerId = partnerId,
    modules = null  // All modules enabled (currently the only supported option)
)
```

Selective module enablement will be available in a future release. See the [iOS Integration Guide (Branding & Modules)](../ios/05-branding-and-modules.md) for the full modules table with descriptions.

---

**Previous:** [Code Integration](04-code-integration.md) | **Next:** [Token Management](06-token-management.md) | **Home:** [README](README.md)
