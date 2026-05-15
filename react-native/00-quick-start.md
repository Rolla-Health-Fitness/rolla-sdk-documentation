# Quick Start

A minimal end-to-end React Native integration. Run on physical devices — Bluetooth and GPS features do not work in simulators or emulators.

> **Before you begin:** verify your project meets the [Prerequisites](01-prerequisites.md). If you are on RN `< 0.80.3`, see the [RN version floor](01-prerequisites.md#react-native-version-floor) section first — iOS will likely work but Android requires `0.80.3+`.

## 1. Install the wrapper

```sh
npm install @rolla-health/react-native-sdk --legacy-peer-deps
```

The `--legacy-peer-deps` flag is required to bypass a `react-test-renderer` peer conflict introduced by the React Native template. See [Installation](02-installation.md#why-legacy-peer-deps).

Pin React to exactly `19.1.0` (no caret) — the RN renderer crashes at launch if React resolves to `19.2.x`:

```sh
npm pkg set 'dependencies.react=19.1.0'
```

## 2. Configure iOS

Edit `ios/Podfile` to add the Rolla CocoaPods source, enable static framework linkage, and apply two ZIPFoundation workarounds. The exact snippet lives in [Installation → iOS](02-installation.md#ios--podfile).

Add the required iOS usage strings to `ios/<YourApp>/Info.plist`. **Without these the app aborts silently with SIGABRT at `Rolla.show()`** — there is no JS-side error you can catch. See [Permissions → iOS](03-permissions.md#ios).

Then:

```sh
cd ios && pod install
```

## 3. Configure Android

Edit `android/settings.gradle` to register three Maven repositories, and `android/build.gradle` + `android/app/build.gradle` to enable core-library desugaring and pin AGP `8.9.1` / Kotlin `2.2.0`. See [Installation → Android](02-installation.md#android--gradle).

`minSdk` must be `26` and `compileSdk` must be `36`.

## 4. Wire the modal

```tsx
import { useEffect, useState } from 'react';
import { Pressable, Text, View } from 'react-native';
import { Rolla } from '@rolla-health/react-native-sdk';

export default function App() {
  const [version, setVersion] = useState('(loading…)');

  useEffect(() => {
    Rolla.getNativeSdkVersion().then(setVersion);

    const closeSub = Rolla.addListener('onClose', (e) => {
      console.log('Closed:', e.reason);
    });
    return () => closeSub.remove();
  }, []);

  return (
    <View>
      <Text>Native SDK: {version}</Text>
      <Pressable
        onPress={async () => {
          const close = await Rolla.show({
            token: 'access-token-from-your-backend',
            refreshToken: 'optional-refresh-token',
            tokenExpiresIn: 1800,
            partnerId: 'your-partner-id',
            environment: 'production',
          });
          console.log('Closed with reason:', close.reason);
        }}>
        <Text>Open Rolla</Text>
      </Pressable>
    </View>
  );
}
```

## 5. Build and run on a physical device

For iOS, build `Release` (Debug requires Metro reachable from the device):

```sh
npx react-native bundle --platform ios --dev false \
  --entry-file index.js --bundle-output ios/main.jsbundle --assets-dest ios/
cd ios && xcodebuild -workspace YourApp.xcworkspace -scheme YourApp \
  -configuration Release -destination 'platform=iOS,id=<your-device-UDID>' \
  -derivedDataPath build DEVELOPMENT_TEAM=<your-team> CODE_SIGN_STYLE=Automatic build
```

For Android:

```sh
cd android && ./gradlew :app:assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

Tap **Open Rolla**. The modal should appear and pair the band.

---

**Next:** [Prerequisites](01-prerequisites.md) | **Home:** [README](README.md)
