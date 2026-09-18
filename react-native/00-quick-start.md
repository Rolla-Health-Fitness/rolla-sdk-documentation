# Quick Start — React Native

Get the Rolla SDK running in your React Native app in under 30 minutes.

> **This guide covers the minimal integration.** For branding, modules, headless calls, host-driven navigation, notification taps, and other features, see the [full documentation](README.md).

## Prerequisites

- **React Native 0.80.3** with the **New Architecture** enabled (`newArchEnabled=true`) and **React 19.1.0** pinned exactly — see [Prerequisites](01-prerequisites.md)
- **iOS 15.1+** deployment target, CocoaPods
- **Android** `minSdk 26`, `compileSdk 36`, Kotlin `2.2.0`, JDK 17
- **Partner ID** from Rolla (contact [support@rolla.app](mailto:support@rolla.app))
- **Physical devices** for Bluetooth and GPS features — simulators and emulators do not validate the full integration

Your app must register users and obtain access tokens from Rolla's authentication API. See [Auth API — Authentication](../sdk-auth-api/02-authentication.md) for the full flow (`/api/register` → `/api/login` → tokens).

## 1. Install the Wrapper

```sh
npm install @rolla-health/react-native-sdk@0.1.16 --save-exact
# or
yarn add @rolla-health/react-native-sdk@0.1.16
```

> A fresh React Native 0.80 template needs `--legacy-peer-deps` with npm because its `react-test-renderer@18` conflicts with React 19 — see [Installation](02-installation.md#1-install-the-package).

The package pins the native iOS pod and Android artifact for you; you never declare those versions yourself. For which native SDK each package version links, see [Prerequisites → Versioning](01-prerequisites.md#versioning).

## 2. Configure iOS

Edit `ios/Podfile` to add the Rolla CocoaPods source, enable static framework linkage, and apply the two `ZIPFoundation` hooks — the complete snippet is in [Installation → iOS](02-installation.md#2-ios--podfile). Then:

```sh
cd ios && pod install
```

Add the usage-description keys, `MBXAccessToken` and `UIBackgroundModes` to `ios/YourApp/Info.plist`, and the HealthKit capability to your app target. Without the usage strings the app aborts at `Rolla.show()` with no JavaScript error to catch — see [Permissions & Entitlements → iOS](03-permissions.md#ios).

## 3. Configure Android

Register the three Maven repositories in `android/settings.gradle`, set the Kotlin version and SDK levels in `android/build.gradle`, and enable core-library desugaring in `android/app/build.gradle` — see [Installation → Android](02-installation.md#4-android--settingsgradle).

Add the Mapbox token to `res/values/strings.xml` and the Health Connect entries to `AndroidManifest.xml` — see [Permissions & Entitlements → Android](03-permissions.md#android).

## 4. Configure and Present

```tsx
import { Rolla } from '@rolla-health/react-native-sdk';

async function showRolla() {
  const close = await Rolla.show({
    token: 'your-access-token',          // JWT from POST /api/login
    refreshToken: 'your-refresh-token',  // From POST /api/login
    tokenExpiresIn: 1800,                // Seconds until the token expires
    partnerId: 'your-partner-id',
    environment: 'rnd',                  // 'production' for release builds
  });
  console.log('Rolla closed:', close.reason);
}
```

`Rolla.show()` presents the SDK UI and resolves when it closes, with the close reason. It rejects when the SDK UI could not be presented — see [Code Integration → Handle Errors](04-code-integration.md#handle-errors).

## 5. Handle Events

Subscribe to the token events so the session stays healthy beyond the first access-token expiry:

```tsx
import { useEffect } from 'react';
import { Rolla } from '@rolla-health/react-native-sdk';

useEffect(() => {
  // The SDK refreshed the token internally — store the new pair for future use.
  const refreshed = Rolla.addListener('onTokenRefreshed', (e) => {
    SessionManager.updateToken(e.token, e.refreshToken, e.expiresIn);
  });

  // The SDK could not refresh the token. Obtain fresh tokens from the Rolla
  // auth API (/api/login), directly or through your backend, and push them.
  const expired = Rolla.addListener('onTokenExpired', async () => {
    const fresh = await YourAPI.fetchNewToken();
    await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
  });

  const failed = Rolla.addListener('onError', (e) => {
    console.warn('Rolla SDK error:', e.code, e.message);
  });

  return () => {
    refreshed.remove();
    expired.remove();
    failed.remove();
  };
}, []);
```

## 6. Logout Cleanup

When the user logs out, clear the SDK session, then tear the engine down:

```ts
await Rolla.clearSession(configuration); // purges the SDK's stored tokens and session data
await Rolla.destroyEngine();             // only after the clear has resolved
```

`clearSession` needs a running engine; passing the configuration lets the wrapper warm one first when the SDK was never opened in this app session — see [Token Management → Clearing the Session](06-token-management.md#clearing-the-session).

## Complete Example

A single, copy-pasteable component covering configuration, presentation, events, and cleanup:

```tsx
import { useCallback, useEffect, useState } from 'react';
import { Pressable, Text, View } from 'react-native';
import { Rolla, RollaConfiguration } from '@rolla-health/react-native-sdk';

export function RollaScreen({ session }: { session: Session }) {
  const [opening, setOpening] = useState(false);

  const configuration: RollaConfiguration = {
    token: session.accessToken,
    refreshToken: session.refreshToken,
    tokenExpiresIn: session.expiresIn,
    partnerId: 'your-partner-id',
    environment: 'rnd', // 'production' for release builds
  };

  useEffect(() => {
    const refreshed = Rolla.addListener('onTokenRefreshed', (e) => {
      session.update(e.token, e.refreshToken, e.expiresIn);
    });
    const expired = Rolla.addListener('onTokenExpired', async () => {
      const fresh = await YourAPI.fetchNewToken();
      await Rolla.updateToken(fresh.token, fresh.refreshToken, fresh.expiresIn);
    });
    const failed = Rolla.addListener('onError', (e) => {
      console.warn('Rolla error:', e.code, e.message);
    });
    return () => {
      refreshed.remove();
      expired.remove();
      failed.remove();
    };
  }, [session]);

  const showRolla = useCallback(async () => {
    setOpening(true);
    try {
      const close = await Rolla.show(configuration);
      console.log('Rolla closed:', close.reason);
    } catch (e: any) {
      console.warn('Rolla could not be presented:', e.code, e.message);
    } finally {
      setOpening(false);
    }
  }, [configuration]);

  const logout = useCallback(async () => {
    await Rolla.clearSession(configuration);
    await Rolla.destroyEngine();
    session.clear();
  }, [configuration, session]);

  return (
    <View>
      <Pressable onPress={showRolla} disabled={opening}>
        <Text>Open Rolla</Text>
      </Pressable>
      <Pressable onPress={logout}>
        <Text>Log out</Text>
      </Pressable>
    </View>
  );
}
```

## Next Steps

- **Permissions:** Set up the `Info.plist` keys, entitlements, Mapbox token and manifest entries — [Permissions & Entitlements](03-permissions.md)
- **Configuration:** Customize branding, force a UI language, and control module and data-source visibility — [Configuration](05-configuration.md)
- **Token details:** Full token lifecycle and edge cases — [Token Management](06-token-management.md)
- **Beyond the modal:** Headless calls, `openScreen`, notification taps and the 12 host events — [API Reference](08-api-reference.md)

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
