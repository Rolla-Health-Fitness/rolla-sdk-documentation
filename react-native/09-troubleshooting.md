# Troubleshooting

React-Native-specific symptoms and remedies. For underlying native build / SDK runtime issues, see [iOS Troubleshooting](../ios/11-troubleshooting.md) and [Android Troubleshooting](../android/09-troubleshooting.md).

## App aborts silently on `Rolla.show()` (iOS)

**Symptom:** the app crashes the moment you tap **Open Rolla**. `xcrun devicectl ... --console` shows only:

```
App terminated due to signal 6.
```

No JS-side error, no JS console output, no backtrace.

**Cause:** missing iOS usage strings. The Rolla SDK touches `CBCentralManager`, `CLLocationManager`, `CMMotionManager`, etc. When iOS sees the native API call without the matching `NSBluetoothAlwaysUsageDescription` / `NSLocationWhenInUseUsageDescription` / `NSMotionUsageDescription` / etc., it calls `abort()` (SIGABRT). The abort is silent in console logs.

**Fix:** add all required keys to `ios/<YourApp>/Info.plist`. See [Permissions → iOS](03-permissions.md#ios) for the complete list.

## `Invariant Violation: TurboModuleRegistry.getEnforcing('RollaWrapper') could not be found`

**Cause:** the native side of the wrapper did not register with the TurboModule registry. Autolinking failed or stale build state.

**Fix:**

```sh
rm -rf ios/Pods ios/Podfile.lock ios/build
rm -rf android/.gradle android/build android/app/build
cd ios && pod install
cd ../android && ./gradlew --refresh-dependencies
```

Then rebuild.

If the error persists after a clean install:
- Confirm `RCTNewArchEnabled` is `true` in `ios/<YourApp>/Info.plist`
- Confirm `newArchEnabled=true` in `android/gradle.properties`
- Confirm the wrapper is listed in `react-native config` output (`npx react-native config | grep RollaWrapper`)

## `Incompatible React versions: react-native-renderer: 19.1.0`

**Cause:** React resolved to `19.2.x` (or another non-`19.1.0` version) because your `package.json` has a caret: `"react": "^19.1.0"`. The RN 0.80.3 renderer hard-checks for exact `19.1.0`.

**Fix:**

```sh
npm pkg set 'dependencies.react=19.1.0'
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

## Android build fails with `Dependency 'androidx.activity:activity:1.12.x' requires Android Gradle plugin 8.9.1 or higher`

**Cause:** your React Native version is below `0.80.3`. RN 0.77–0.79 bundle a gradle plugin pinned to AGP 8.7.x, which cannot satisfy the AndroidX 1.18 / activity 1.12 transitive deps the Rolla SDK pulls.

**Fix:** bump to React Native `0.80.3` or later. See [Prerequisites → RN version floor](01-prerequisites.md#react-native-version-floor).

## `npm install` fails with `ERESOLVE` / peer dependency conflict on `react-test-renderer`

**Cause:** RN template pins `react-test-renderer@18.x` whose `peerDependencies.react` is `^18.2.0`, conflicting with your `react@19.1.0`.

**Fix:**

```sh
npm install @rolla-health/react-native-sdk --legacy-peer-deps
```

Or drop `react-test-renderer` from `devDependencies` if you don't run snapshot tests.

## `pod install` fails with `dyld: Library not loaded: @rpath/ZIPFoundation.framework/ZIPFoundation`

**Cause:** your Podfile is missing the `pre_install` hook that forces `ZIPFoundation` to a dynamic framework. Under static linkage, `ZIPFoundation` is statically linked into the app binary and not embedded in `Frameworks/`. `NordicDFU.xcframework` (vendored by RollaSDK) was pre-built expecting a dynamic ZIPFoundation, so dyld fails at launch.

**Fix:** add the `pre_install` block from [Installation → iOS](02-installation.md#ios--podfile).

## `pod install` fails with `compiling for iOS 14.0, but module 'ZIPFoundation' has a minimum deployment target of iOS 15.1`

**Cause:** missing `post_install` hook to pin `ZIPFoundation`'s deployment target.

**Fix:** add the `post_install` block from [Installation → iOS](02-installation.md#ios--podfile), specifically the `if target.name == 'ZIPFoundation'` branch.

## iPhone install rejects with `parent bundle has the same identifier as sub-bundle`

**Cause:** you passed `PRODUCT_BUNDLE_IDENTIFIER=...` as a global `xcodebuild` override. The override propagates to Pods sub-frameworks (e.g. `ZIPFoundation`) and `devicectl` refuses to install.

**Fix:** set the bundle ID directly in the app target's `project.pbxproj` (in Xcode: **Targets → YourApp → General → Identity → Bundle Identifier**). Never pass it as a global `xcodebuild` arg.

## Hermes script phase fails with `: command not found`

**Cause:** Xcode's script-phase shell does not source your interactive PATH; `command -v node` returns empty.

**Fix:** hard-code an absolute path in `ios/.xcode.env`:

```sh
export NODE_BINARY=/opt/homebrew/bin/node
```

Find your node path with `which node` in your shell.

## Bumping the SDK doesn't pick up the new native version

**Cause:** Gradle caches transitive Mapbox metadata per coordinate. When the wrapper bumps its native pin (e.g. `0.1.10` → `0.1.11`), the cache can serve stale dependency info.

**Fix:**

```sh
cd android && ./gradlew --refresh-dependencies
```

This cannot be fixed on Rolla's side — the refresh must happen on your machine.

## Metro "EADDRINUSE: address already in use :::8081"

**Cause:** another Metro instance (from a sibling RN project) is bound to port 8081.

**Fix:**

```sh
lsof -ti:8081 | xargs kill -9
npx react-native start
```

## Native SDK version reports as `(loading…)` and never resolves

**Cause:** `Rolla.getNativeSdkVersion()` promise is never settling — TurboModule call is hanging because the bridge is not initialized. Usually paired with a pending redbox.

**Fix:** check the JS error console (`adb logcat` on Android, Xcode console on iOS) — there will usually be a `TurboModuleRegistry.getEnforcing` error. See the corresponding entry above.

## Still stuck

Pair the wrapper's diagnostics with the platform troubleshooting guides:

- [iOS Troubleshooting](../ios/11-troubleshooting.md) — build errors, Apple Health, code-signing
- [Android Troubleshooting](../android/09-troubleshooting.md) — Gradle errors, Mapbox cache, ProGuard

Open a support ticket with:
- `@rolla-health/react-native-sdk` version (`npm view @rolla-health/react-native-sdk version`)
- React Native version (`npx react-native --version`)
- Native SDK version reported by `Rolla.getNativeSdkVersion()`
- Verbatim error text (not a paraphrase)
- Platform, OS version, device model

---

**Home:** [README](README.md)
