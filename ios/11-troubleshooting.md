# Troubleshooting

This section provides solutions to common issues encountered when integrating and using the Rolla SDK.

## Common Issues

### SDK fails to start

- Ensure all Info.plist permissions are configured
- Verify `bluetooth-central` is listed under `UIBackgroundModes` in `Info.plist`
- Check that the token is valid and not expired
- Verify HealthKit capability is enabled (if using Apple Health)

### Apple Health not showing

- Verify `NSHealthShareUsageDescription` is in Info.plist
- Verify HealthKit capability is added in Xcode (Signing & Capabilities)
- HealthKit is not available on iPads without the Health app

### Build errors

- Clean build folder: Product > Clean Build Folder (Shift-Command-K)
- Delete Pods folder and Podfile.lock
- Run `pod install` again
- Ensure you're opening `.xcworkspace`, not `.xcodeproj`

### Deployment target errors after an Xcode update

Each major Xcode release raises the oldest iOS version it can build for. Any pod in your project that still declares an older deployment target then fails the build, for example:

```
The iOS Simulator deployment target 'IPHONEOS_DEPLOYMENT_TARGET' is set to 12.0, but the range of supported deployment target versions is 15.0 to 27.1.x.
```

RollaSDK's dependencies can trigger this too: `ZIPFoundation`, for example, can still declare an older deployment target such as iOS 12.0. Raise every pod below the RollaSDK minimum iOS version (see [Prerequisites](01-prerequisites.md)) to that minimum, at the end of your Podfile:

```ruby
# RollaSDK's minimum iOS version, from Prerequisites.
rolla_minimum_ios = '15.0'

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      if config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'].to_f < rolla_minimum_ios.to_f
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = rolla_minimum_ios
      end
    end
  end
end
```

Raise them *to* the minimum, not to your app's own deployment target. RollaSDK's prebuilt `NordicDFU.xcframework` targets the minimum and imports `ZIPFoundation`, so setting `ZIPFoundation` higher fails the build with an error such as:

```
compiling for iOS 15.0, but module 'ZIPFoundation' has a minimum deployment target of iOS 16.0
```

### Calling `show()` while the SDK is already presenting

If you call `show(from:)` while the SDK UI is already on screen, the SDK does **not** crash or queue a second presentation. Instead, it immediately fires a `.alreadyPresenting` error through your delegate:

```swift
func rollaDidFailWithError(_ rolla: Rolla, error: RollaError) {
    if case .alreadyPresenting = error {
        // SDK is already showing — no action needed
    }
}
```

To avoid this, check `rolla.isPresenting` before calling `show(from:)`, or guard your UI so the user cannot trigger it twice.

### Token-related issues

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| SDK opens then immediately closes or shows an error | Expired access token | Initialize with a current token pair — see [Token Management](07-token-management.md) |
| Works at first, then `Access denied (HTTP 401)` on SDK screens after ~30 minutes; re-entering the SDK temporarily fixes it — until it recurs | The SDK's refresh token was consumed outside the SDK (or never passed), and `rollaDidRequestTokenRefresh` goes unanswered. Re-entry only masks the problem: your app passes a fresh access token at initialization, which works for another 30 minutes | [Responsibilities 2–4](07-token-management.md#your-apps-responsibilities) |
| `rollaDidRequestTokenRefresh` fires repeatedly in bursts | Same cause — every request that fails after an unsuccessful internal refresh escalates to your app | Answer it with `updateToken()`; verify the refresh token you pass at initialization is the latest one |
| 401 errors start right after your own backend refreshes its session | Your backend consumed the refresh token the SDK was holding (single-use rotation) | [Responsibility 5](07-token-management.md#your-apps-responsibilities) |
| `rollaDidRequestTokenRefresh` fires immediately at launch | The access token was already expired and the refresh token invalid at initialization | Initialize with the newest persisted pair (a `tokenExpiresIn` that overshoots the token's own expiry is corrected automatically) |
| SDK works in `"rnd"` but fails in `"production"` | Token was issued for the wrong environment | Verify your backend issues tokens against the correct Rolla environment |
| "Unauthorized" or 401-style errors | Wrong partner ID or mismatched credentials | Double-check the `partnerId` passed to `RollaConfiguration` |

### Flutter engine crash recovery

If the Flutter engine crashes (rare, but possible under extreme memory pressure), the SDK UI will close and `rollaDidClose` will fire. To recover:

1. Call `Rolla.destroyEngine()` to fully tear down the crashed engine.
2. Create a new `Rolla` instance with a fresh configuration.
3. Call `show(from:)` to restart.

```swift
func rollaDidClose(_ rolla: Rolla, reason: RollaCloseReason) {
    Rolla.destroyEngine()
    // Re-create and show when ready
}
```

See [Engine Lifecycle](08-engine-lifecycle.md) for more on `destroyEngine()`.

### Getting debug logs for support tickets

The SDK logs internally using Apple's unified logging system with the subsystem `app.rolla.rollaV2`. To capture these logs:

1. Open **Console.app** on your Mac with the device connected.
2. Filter by subsystem: `app.rolla.rollaV2`.
3. Reproduce the issue.
4. Export the filtered log and attach it to your support ticket.

You can also stream logs from the command line:

```bash
log stream --device --predicate 'subsystem == "app.rolla.rollaV2"' --level info
```

### CocoaPods issues

**`pod install` fails or resolves the wrong version:**

```bash
# Clear the CocoaPods cache and re-install
pod cache clean --all
rm -rf Pods Podfile.lock
pod install
```

**Spec repo out of date:**

```bash
pod repo update
pod install
```

**Use `pod install` vs `pod update`:**
- `pod install` — Use when adding the SDK for the first time or when checking out a project. Respects `Podfile.lock`.
- `pod update RollaSDK` — Use when you want to pull a newer SDK version. Updates the lock file.

**"Unable to find a specification" error:**
- Verify the Rolla SDK source URL in your `Podfile` is correct.
- Ensure you have network access to the Rolla SDK repository.

## Support

For issues or questions, contact Rolla support or refer to the SDK documentation.

---

**Previous:** [API Reference](10-api-reference.md) | **Home:** [README](README.md)
