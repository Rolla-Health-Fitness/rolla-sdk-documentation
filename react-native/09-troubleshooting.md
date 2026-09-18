# Troubleshooting

React Native-specific symptoms and remedies. For SDK runtime issues inside the native layers — Apple Health, Health Connect, Bluetooth, GPS, background tracking, Gradle and CocoaPods resolution — see [iOS Troubleshooting](../ios/11-troubleshooting.md) and [Android Troubleshooting](../android/09-troubleshooting.md); everything there applies to a React Native host unchanged.

## Common Issues

### App aborts silently at `Rolla.show()` (iOS)

**Symptom:** the app crashes the moment the SDK opens. The device console shows only `App terminated due to signal 6.` — no JavaScript error, no red box, no backtrace.

**Cause:** a missing `Info.plist` usage string. iOS calls `abort()` when the SDK touches `CBCentralManager`, `CLLocationManager` or `CMMotionManager` without the matching `NSBluetoothAlwaysUsageDescription`, `NSLocationWhenInUseUsageDescription` or `NSMotionUsageDescription`.

**Fix:** add every key listed in [Permissions & Entitlements → iOS](03-permissions.md#ios).

### `Invariant Violation: TurboModuleRegistry.getEnforcing('RollaWrapper') could not be found`

**Cause:** the native side of the wrapper did not register with the TurboModule registry — autolinking did not run, or the build state is stale.

**Fix:**

```sh
rm -rf ios/Pods ios/Podfile.lock ios/build
rm -rf android/.gradle android/build android/app/build
cd ios && pod install
cd ../android && ./gradlew --refresh-dependencies
```

Then rebuild. If the error persists, confirm autolinking sees the package with `npx react-native config | grep RollaWrapper`. If `RollaWrapper` does not appear, the package is missing from `node_modules`, or a `react-native.config.js` is overriding autolinking. Also confirm the New Architecture is on (`newArchEnabled=true` in `android/gradle.properties`, `RCTNewArchEnabled` in `Info.plist`) — the wrapper does not run under the legacy bridge.

### `Incompatible React versions: react-native-renderer: 19.1.0`

**Cause:** React resolved to a version other than the one your React Native release pins, because `package.json` has a caret (`"react": "^19.1.0"`). The RN 0.80.3 renderer hard-checks for exactly `19.1.0`.

**Fix:**

```sh
npm pkg set 'dependencies.react=19.1.0'
rm -rf node_modules package-lock.json
npm install --legacy-peer-deps
```

### `npm install` fails with `ERESOLVE` on `react-test-renderer`

**Cause:** the React Native template pins `react-test-renderer@18.x`, whose peer dependency `react@^18.2.0` conflicts with your `react@19.1.0`.

**Fix:** install with `--legacy-peer-deps`, or drop `react-test-renderer` from `devDependencies` if you don't run snapshot tests — see [Installation](02-installation.md#1-install-the-package).

### Android build fails with `Found interface org.jetbrains.kotlin.gradle.dsl.KotlinTopLevelExtension, but class was expected`

**Cause:** your React Native version is below `0.80`. The native SDK needs Kotlin 2.2, and the gradle plugin bundled with RN 0.77–0.79 was compiled against Kotlin 2.0. Overriding Kotlin at the root `build.gradle` does not help.

**Fix:** upgrade to React Native 0.80.3 or later — see [Prerequisites → React Native version floor](01-prerequisites.md#react-native-version-floor).

### `dyld: Library not loaded: @rpath/ZIPFoundation.framework/ZIPFoundation` at launch

**Cause:** your Podfile is missing the `pre_install` hook that forces `ZIPFoundation` to a dynamic framework. Under static linkage it is linked into the app binary and not embedded in `Frameworks/`, while `NordicDFU.xcframework` (vendored by `RollaSDK`) was pre-built expecting it as a dynamic framework.

**Fix:** add the `pre_install` block from [Installation → iOS](02-installation.md#2-ios--podfile).

### `compiling for iOS 14.0, but module 'ZIPFoundation' has a minimum deployment target of iOS 15.1`

**Cause:** the `post_install` hook that pins `ZIPFoundation`'s deployment target to 14.0 is missing.

**Fix:** add the `post_install` block from [Installation → iOS](02-installation.md#2-ios--podfile), specifically the `if target.name == 'ZIPFoundation'` branch.

### Device install rejects with `parent bundle has the same identifier as sub-bundle`

**Cause:** `PRODUCT_BUNDLE_IDENTIFIER=…` was passed as a global `xcodebuild` override. The override propagates to the Pods sub-frameworks (e.g. `ZIPFoundation`), and `devicectl` refuses to install.

**Fix:** set the bundle ID in the app target's `project.pbxproj` (Xcode: **Targets → YourApp → General → Identity → Bundle Identifier**), never as a global `xcodebuild` argument.

### Hermes script phase fails with `: command not found`

**Cause:** Xcode's script-phase shell does not source your interactive `PATH`; `command -v node` returns empty.

**Fix:** put an absolute node path in the gitignored `ios/.xcode.env.local`:

```sh
export NODE_BINARY=/opt/homebrew/bin/node
```

Find your path with `which node`. Do not edit the versioned `ios/.xcode.env`.

### `show()` or `openScreen()` does nothing, or closes immediately with `hostModalDismiss` (iOS)

**Cause:** the call was made from inside a React Native `Modal` (a bottom sheet, a picker) that was closing at the same time. The SDK UI is presented on top of the frontmost view controller — the modal's — and is torn down together with it.

**Fix:** call from a settled screen; wait for the modal's `onDismiss` before calling. Android's `Modal` is a dialog window and is not affected. See [Code Integration → Present the SDK](04-code-integration.md#present-the-sdk).

### `updateToken()` or `clearSession()` rejects with `NO_ACTIVE_SESSION`

**Cause:** the engine is cold — no `show()`, `openScreen()`, `warmUpEngine()` or headless call has started it in this app session — so there is no session to update or clear.

**Fix:** for `updateToken()`, pass the newest token pair in your next configuration instead. For `clearSession()`, pass the configuration — the wrapper warms the engine first, then clears. See [Token Management](06-token-management.md#clearing-the-session).

### Calling `show()` while the SDK is already presenting

`show()` rejects with `ALREADY_PRESENTING` when a `show()` is still pending or the SDK UI is on screen. Check `Rolla.isPresenting()` before calling, or `dismiss()` first. Use `openScreen()` when the intent is to navigate an already-open SDK.

### Maps stay blank (Android)

**Cause:** `mapbox_access_token` is missing from `res/values/strings.xml`. The SDK runs without it, but every map stays blank.

**Fix:** add the token — see [Permissions & Entitlements → Mapbox Token](03-permissions.md#mapbox-token) and [Android Troubleshooting → Maps not showing](../android/09-troubleshooting.md#maps-not-showing).

### Bumping the package doesn't pick up the new native version

**Cause:** Gradle caches transitive Mapbox metadata per coordinate. When the wrapper's native pin changes, the cache can serve stale dependency info and fail with confusing resolution errors.

**Fix:**

```sh
cd android && ./gradlew --refresh-dependencies
```

This cannot be fixed on Rolla's side — the refresh must happen on your machine. See [Android Troubleshooting → Stale transitive dependencies](../android/09-troubleshooting.md#stale-transitive-dependencies-after-bumping-the-sdk-version).

### Token-related issues

- **`onTokenExpired` keeps firing** — your handler must call `Rolla.updateToken()` with a *fresh* pair; a pair older than the one the SDK holds is ignored by design. Obtain new tokens via `/api/login`, directly or through your backend.
- **The SDK cannot refresh on its own** — pass all three of `token`, `refreshToken` and `tokenExpiresIn`, and never spend the refresh token in your own backend calls. See [Token Management → Your App's Responsibilities](06-token-management.md#your-apps-responsibilities).

### Getting debug logs for support tickets

The SDK logs through the platform logging systems, exactly as in a native app:

- **iOS:** unified logging under subsystem `app.rolla.rollaV2` — filter Console.app by it, or `log stream --device --predicate 'subsystem == "app.rolla.rollaV2"' --level info`.
- **Android:** `adb logcat -s RollaEngineManager:* RollaSdkPlugin:* Flutter:*`.

Attach the filtered log to your ticket. More symptoms and remedies: [iOS Troubleshooting → debug logs](../ios/11-troubleshooting.md#getting-debug-logs-for-support-tickets) and [Android Troubleshooting → debug logs](../android/09-troubleshooting.md#getting-debug-logs-for-support-tickets).

## Support

Open a support ticket with:

- `@rolla-health/react-native-sdk` version (from your `package.json`)
- React Native version (`npx react-native --version`)
- Native SDK version reported by `Rolla.getNativeSdkVersion()`
- Verbatim error text and the rejection `code`, not a paraphrase
- Platform, OS version, device model
- The filtered debug log from above

- **Email:** [support@rolla.app](mailto:support@rolla.app)
- **Slack:** If your organization has a dedicated partner Slack channel with Rolla, use it for faster responses on integration questions and live debugging.

---

**Previous:** [API Reference](08-api-reference.md) | **Home:** [README](README.md)
